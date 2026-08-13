# Day 36 – Docker Project: Dockerize a Full Application

## Overview

The goal of this task was to take a real application and Dockerize it end-to-end — write a Dockerfile, set up Docker Compose with a database, ship the image to Docker Hub, and verify it works when pulled fresh with no local build.

---

## Task 1: Picking the App

**App chosen:** A Flask (Python) application connected to a PostgreSQL database.

**Why this combination:**
- Flask is a small, easy-to-understand Python web framework — good for a first Docker project.
- Postgres has a widely used, well-documented official Docker image, so it's a common real-world combination (Flask/Django + Postgres is very standard in the industry).
- It lets me demonstrate a real multi-container setup: one container for the application (built from my own Dockerfile) and one container for the database (from a ready-made official image).

**What the app does:**
It's a simple visit counter. Every time the `/` route is hit:
1. Flask inserts a new row into a `visits` table in Postgres.
2. Flask asks Postgres for the total row count.
3. Flask returns that count as JSON.

This proves the app container and the database container are actually talking to each other over Docker's network — not just running side by side doing nothing.

There's also a `/health` route that just returns `{"status": "ok"}`, useful for healthchecks.

---

## Task 2: Writing the Dockerfile

### Final Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN useradd --create-home --shell /bin/bash appuser

COPY app/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app/ .
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 5000

CMD ["python", "app.py"]
```

### Line-by-line explanation

| Line | What it does | Why |
|---|---|---|
| `FROM python:3.12-slim` | Uses the official Python 3.12 image, "slim" variant | Slim has only what's needed to run Python — much smaller than the full image (~150MB vs ~1GB), while still being more compatible than `alpine` (which can cause issues installing packages like `psycopg2`) |
| `WORKDIR /app` | Sets `/app` as the working directory inside the container | Keeps all app files in one predictable place instead of scattered in the container's root filesystem |
| `RUN useradd --create-home --shell /bin/bash appuser` | Creates a non-root Linux user called `appuser` | By default, containers run as `root`, which is a security risk — if an attacker breaks into the container, they'd have root access. Running as a limited user reduces that risk. This is created early so it exists before we assign file ownership later. |
| `COPY app/requirements.txt .` | Copies only the dependencies file first | Docker caches each instruction as a "layer". By copying `requirements.txt` before the rest of the code, the `pip install` layer only re-runs when dependencies actually change — not every time application code changes. This makes rebuilds much faster. |
| `RUN pip install --no-cache-dir -r requirements.txt` | Installs Flask and psycopg2 (the Postgres driver for Python) | `--no-cache-dir` stops pip from keeping its own download cache inside the image, since the image is never "reused" for a future install — this keeps the image smaller |
| `COPY app/ .` | Copies the rest of the application code | Done *after* installing dependencies, since code changes far more often than dependencies — this keeps the layer caching benefit |
| `RUN chown -R appuser:appuser /app` | Gives ownership of `/app` to the `appuser` user | The files were copied in as `root`; without this, `appuser` wouldn't have permission to read/write them |
| `USER appuser` | Switches the active user from `root` to `appuser` | From this point on (including when the app actually runs), the container operates as a non-root user |
| `EXPOSE 5000` | Documents that the app listens on port 5000 | This doesn't actually open the port — it's metadata. The real port mapping happens later in `docker-compose.yml` |
| `CMD ["python", "app.py"]` | The command that runs when the container starts | This is the container's main process — as long as it's alive, the container stays alive |

### `.dockerignore`

```
__pycache__
*.pyc
.git
.gitignore
.env
*.md
.dockerignore
```

**Why each entry:**
- `__pycache__`, `*.pyc` — Python's auto-generated cache files; not needed at runtime and can cause stale-bytecode issues if copied in
- `.git` — the entire commit history; can be huge and is never needed inside a running container
- `.env` — contains secrets (DB password); should never end up baked into an image
- `*.md` — documentation files; humans read these, the app doesn't need them

### Build and test results

- Final image size: **140MB** (well under the "keep it small" target, thanks to the `slim` base image)
- Verified with `docker build`, then `docker run -p 5000:5000` — the container started cleanly and (as expected, with no database yet) failed gracefully with `Exception: Could not connect to the database`, proving the retry logic worked correctly.

**Screenshots:**
- `Screenshots/01-project-folder-structure.png` — final project folder layout
- `Screenshots/02-dockerignore-and-dockerfile.png` — `.dockerignore` and `Dockerfile` contents
- `Screenshots/03-docker-build-process.png` — `docker build` in progress
- `Screenshots/04-build-success-image-and-container-run.png` — successful build, image size, and standalone container run

---

## Task 3: Docker Compose

### Why Compose is needed

Running `docker run` twice (once for the app, once for the database) works, but it means manually creating a network, remembering the exact flags every time, and getting the startup order right. Docker Compose puts all of that into one YAML file, so the entire stack starts with a single command: `docker compose up`.

### Final `docker-compose.yml`

```yaml
services:
  app:
    build: .
    container_name: testapp2
    ports:
      - "5000:5000"
    environment:
      DB_HOST: db
      DB_NAME: ${POSTGRES_DB}
      DB_USER: ${POSTGRES_USER}
      DB_PASSWORD: ${POSTGRES_PASSWORD}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - app_network

  db:
    image: postgres:16-alpine
    container_name: postgres_db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d ${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - app_network

volumes:
  db_data:

networks:
  app_network:
    driver: bridge
```

### Key concepts explained

- **`build: .`** — builds the `app` image from the Dockerfile in the current folder, instead of pulling a ready-made one.
- **`ports: - "5000:5000"`** — maps host port 5000 to container port 5000, so the app is reachable from outside the container.
- **`environment:`** — the app's code (`app.py`) reads `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` via `os.environ.get(...)`. These are injected here instead of hardcoded. `DB_HOST` is set to `db` directly because Compose automatically makes each service reachable by its service name — so `db` resolves to the Postgres container's IP on the shared network.
- **`${POSTGRES_DB}` etc.** — these values are pulled from the `.env` file, not hardcoded in the compose file. This keeps secrets out of version control and lets the same file work across environments.
- **`depends_on: db: condition: service_healthy`** — makes the `app` container wait until `db` passes its healthcheck (not just "started", but actually *ready to accept connections*) before starting. Without this, the app could start before Postgres is ready and crash trying to connect too early.
- **`image: postgres:16-alpine`** — uses the official Postgres image directly, no custom Dockerfile needed for the database.
- **`volumes: - db_data:/var/lib/postgresql/data`** — maps a named Docker volume to the folder where Postgres stores its actual data files. This means the data survives even if the container is deleted and recreated — only deleting the volume itself (`docker compose down -v`) removes it.
- **`healthcheck:`** — runs `pg_isready` inside the `db` container every 5 seconds, up to 5 retries, to determine when Postgres is truly ready.
- **`networks: app_network: driver: bridge`** — creates a private, isolated network so `app` and `db` can talk to each other by name, without other unrelated containers on the host being able to reach them directly.

### Verified results

- `docker compose up --build` started both containers; `db` became healthy first, then `app` started successfully — no database connection error this time.
- Tested with repeated `curl http://localhost:5000/` — `total_visits` incremented (1 → 2 → 3...) confirming Flask and Postgres were reading/writing the same data correctly.
- Tested `/health` endpoint — returned `{"status": "ok"}`.
- Opened the app in a browser via the EC2 public IP (`http://<ec2-ip>:5000/`) after opening port 5000 in the AWS Security Group — confirmed external access works.
- Restarted the stack with `docker compose down` (without `-v`) then `docker compose up -d` — the visit counter picked up where it left off instead of resetting to 1, proving the volume correctly persists data across container recreation.

**Screenshots:**
- `Screenshots/docker-compose-full-file.png` — full `docker-compose.yml` contents
- `Screenshots/docker-compose-up-build-process.png` — `docker compose up --build` running, pulling Postgres and building the app image
- `Screenshots/ssh-login-and-app-curl-test.png` — curl tests showing `total_visits` increasing
- `Screenshots/app-working-in-browser-public-ip.png` — app accessed from a browser via the EC2 public IP

---

## Task 4: Shipping to Docker Hub

### Commands used

```bash
docker login
docker tag flask-postgres-app rtingane2611/flask-postgres-app:latest
docker push rtingane2611/flask-postgres-app:latest
```

**What each command does:**
- `docker login` — authenticates the CLI with a Docker Hub account (already logged in via saved credentials in this case).
- `docker tag <old-name> <new-name>` — doesn't create a new image, just gives the existing image an additional name. Docker Hub requires the format `username/repository:tag`, so the local image `flask-postgres-app` was tagged as `rtingane2611/flask-postgres-app:latest`.
- `docker push <image>` — uploads all image layers to Docker Hub under that repository name. Layers that already exist on Docker Hub (e.g. base image layers shared with the official `python` image) are skipped/mounted instead of re-uploaded, which is why some layers showed "Mounted from library/python" instead of "Pushed".

### Docker Hub Link

**https://hub.docker.com/r/rtingane2611/flask-postgres-app**

Anyone can pull this image directly:
```bash
docker pull rtingane2611/flask-postgres-app:latest
```

**Screenshots:**
- `Screenshots/docker-tag-and-push-success.png` — tagging and push process, ending in a successful digest
- `Screenshots/dockerhub-repo-listing.png` — the repository visible on Docker Hub, marked Public

---

## Task 5: Fresh Pull Test

### Goal

Prove that the shipped image is genuinely self-contained — that someone with only `docker-compose.yml` and no access to the source code or Dockerfile could pull the image from Docker Hub and run the full stack successfully.

### Steps taken

1. **Changed `docker-compose.yml`** — replaced `build: .` under the `app` service with:
   ```yaml
   image: rtingane2611/flask-postgres-app:latest
   ```
   This tells Compose to pull the image from Docker Hub instead of building it locally.

2. **Stopped and removed all running containers:**
   ```bash
   docker compose down
   ```
   (Note: `-v` was intentionally *not* used here, to keep the database volume intact and test persistence as well.)

3. **Deleted every local image**, to guarantee a truly clean slate:
   ```bash
   docker rmi -f $(docker images -aq)
   ```
   Verified with `docker images` — the list came back completely empty.

4. **Brought the stack back up, without building anything:**
   ```bash
   docker compose up -d
   ```
   Compose pulled both `postgres:16-alpine` and `rtingane2611/flask-postgres-app:latest` directly from their registries — no `build` step ran at all.

5. **Verified the result:**
   ```bash
   docker compose ps
   curl http://localhost:5000/
   ```
   Output:
   ```json
   {"message":"Flask + Postgres Dockerized app running!","total_visits":11}
   ```

### Why this result matters

- `total_visits` was **11**, not 1 — meaning the database volume from before the cleanup was still intact, and the freshly pulled containers picked up exactly where the old ones left off.
- Both containers came up healthy and connected correctly with zero local source code or Dockerfile involved in the `app` container's creation — only the Docker Hub image and the compose file were needed.

**Screenshots:**
- `Screenshots/fresh-test-cleanup-images-deleted.png` — all local images being deleted, confirmed empty
- `Screenshots/fresh-pull-test-success-total-visits-11.png` — full fresh pull, healthy containers, and `total_visits: 11` proving both a clean pull and data persistence

---

## Challenges Faced

1. **Non-English text in code and error messages.** The app code originally had Marathi comments and error strings, which looked out of place in a professional codebase. Fixed by rewriting `app.py` fully in English and rebuilding the image.
2. **YAML indentation errors in `docker-compose.yml`.** Mixing Python syntax (`os.environ.get(...)`) directly into an `environment:` block caused invalid YAML. Fixed by understanding that Compose only accepts simple `key: value` pairs, with values pulled in from `.env` using `${VARIABLE}` syntax.
3. **Understanding `depends_on` vs `healthcheck`.** Initially, `depends_on` alone only controlled start *order*, not readiness — the app could still try connecting before Postgres was ready. Adding `condition: service_healthy` combined with a `pg_isready` healthcheck solved this properly.
4. **Confusing `db` service environment variables with `app` service environment variables.** Postgres's official image expects `POSTGRES_DB` / `POSTGRES_USER` / `POSTGRES_PASSWORD`, while the Flask app's own code expects `DB_NAME` / `DB_USER` / `DB_PASSWORD`. Solved by keeping both blocks separate but pointing to the same underlying `.env` values, so both containers agree on the same credentials.
5. **`docker rmi -f $(docker images -aq)` partially failing** with "cannot be forced — image has dependent child images" errors. This happened because of parent-child layer relationships between images sharing a base. Re-running `docker images` confirmed all images were in fact removed despite the errors on some intermediate layers.

---

## Final Image Details

| Detail | Value |
|---|---|
| Base image | `python:3.12-slim` |
| Final image size | **140MB** |
| Non-root user | Yes (`appuser`) |
| Docker Hub repository | `rtingane2611/flask-postgres-app` |
| Docker Hub link | https://hub.docker.com/r/rtingane2611/flask-postgres-app |

---

## Submission Checklist

- [x] `day-36-docker-project.md` (this file)
- [x] `Dockerfile`
- [x] `docker-compose.yml`
- [x] `.dockerignore`
- [x] `.env.example`
- [x] `README.md`
- [x] Application code (`app/app.py`, `app/requirements.txt`)
- [x] Image pushed to Docker Hub
- [x] Verified fresh pull with no local build
