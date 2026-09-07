# Day 48 – GitHub Actions Project: End-to-End CI/CD Pipeline

This document explains **everything** I built for this project — the app, the Dockerfile, every workflow file (line by line, in simple words), every error I hit, and how I fixed each one. Anyone reading this — even a beginner — should be able to understand what was done and rebuild it themselves.

---

## 1. Project Overview

**Goal:** Build a complete CI/CD pipeline using GitHub Actions that automatically:
1. Tests the code when someone opens a Pull Request (PR)
2. Builds a Docker image and pushes it to Docker Hub when code is merged to `main`
3. Deploys the image to a production server (AWS EC2)
4. Runs a health check every 12 hours to make sure the app is still alive

**App used:** A simple Python **Flask** app with one route: `/health`, which returns `{"status": "ok"}`.

**Why this matters:** In a real company, a developer should never have to manually copy files to a server and restart it. This pipeline does that work automatically, safely, and with a record of what happened (logs + screenshots).

---

## 2. Pipeline Architecture (Simple Flow Diagram)

```
Developer opens a Pull Request
        │
        ▼
  pr-pipeline.yml runs
        │
        ▼
  Calls reusable-build-test.yml
   (checkout → install deps → run tests)
        │
        ▼
  PR comment: "PR checks passed for branch: <branch>"
  (No Docker build/push happens here — PRs are test-only)

──────────────────────────────────────────────

Developer merges PR into main
        │
        ▼
  main-pipeline.yml runs
        │
        ├─► Job 1: calls reusable-build-test.yml (test again)
        │
        ├─► Job 2: calls reusable-docker.yml
        │         (docker login → build image → push to Docker Hub
        │          tags: latest AND sha-<short-commit-hash>)
        │
        └─► Job 3: deploy job
                  (prints "Deploying image: <url> to production"
                   uses the "production" GitHub Environment,
                   can require manual approval)

──────────────────────────────────────────────

Every 12 hours (cron schedule) OR manual run
        │
        ▼
  health-check.yml runs
        │
        ▼
  pulls latest image → runs container → waits 5s →
  curls /health → prints PASS/FAIL → writes a report
  to the Actions Summary page → stops & removes container
```

---

## 3. The App Files (given by the "developer")

### 3.1 `app.py` — the Flask app

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/health")
def health():
    return jsonify({"status": "ok"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

**Line by line, in simple words:**
- `from flask import Flask, jsonify` → we borrow two tools from the Flask library: `Flask` (to create the app) and `jsonify` (to turn a Python dictionary into proper JSON that a browser/curl can read).
- `app = Flask(__name__)` → this creates the actual app object. Every Flask app needs this line.
- `@app.route("/health")` → this is a "decorator." It tells Flask: "whenever someone visits the web address `/health`, run the function right below this line."
- `def health():` → the function that runs when `/health` is visited.
- `return jsonify({"status": "ok"})` → sends back the JSON `{"status": "ok"}` as the response. This is what our health checks look for.
- `if __name__ == "__main__":` → this means "only run the code below if this file is run directly" (not if it's imported by another file).
- `app.run(host="0.0.0.0", port=5000)` → starts the web server. `host="0.0.0.0"` means "listen on every network interface" (needed so Docker can expose it), and `port=5000` is the port number the app listens on.

**Important lesson learned:** the app only answers on `/health`, NOT on `/` (root). Visiting `http://localhost:5000` gives a 404 error — that is expected. You must visit `http://localhost:5000/health`.

### 3.2 `requirements.txt`

```
flask
```

This just tells Python/pip which packages to install before the app can run.

### 3.3 `Dockerfile`

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

**Line by line:**
- `FROM python:3.11-slim` → start from an existing, small (slim) image that already has Python 3.11 installed. We build on top of it instead of starting from nothing.
- `WORKDIR /app` → sets `/app` as the "current folder" inside the container. Every command below happens inside this folder.
- `COPY requirements.txt .` → copies just the requirements file into the container first (before the rest of the code). This is a trick to speed up future builds — if the code changes but requirements don't, Docker can reuse the old "install" step.
- `RUN pip install --no-cache-dir -r requirements.txt` → installs Flask (and anything else listed) inside the container. `--no-cache-dir` keeps the image smaller by not saving pip's temporary cache files.
- `COPY . .` → now copy everything else (app.py, etc.) into the container.
- `EXPOSE 5000` → documentation for humans and Docker tooling that this container listens on port 5000. It does NOT actually open the port by itself — that happens later with `-p` when running the container.
- `CMD ["python", "app.py"]` → the command that runs automatically when the container starts.

### 3.4 `README.md` (developer's original, before badges)

A short description of the app and how to run it locally with `docker build` and `docker run`.

---

## 4. Task 2 — `reusable-build-test.yml`

```yaml
name: Reusable Build & Test

on:
  workflow_call:
    inputs:
      python_version:
        required: false
        type: string
        default: "3.11"
      run_tests:
        required: false
        type: boolean
        default: true
    outputs:
      test_result:
        description: "Result of the test run"
        value: ${{ jobs.build-test.outputs.result }}

jobs:
  build-test:
    runs-on: ubuntu-latest
    outputs:
      result: ${{ steps.set-result.outputs.result }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: ${{ inputs.python_version }}

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Run tests
        if: ${{ inputs.run_tests == true }}
        run: |
          python app.py &
          sleep 3
          curl --fail http://localhost:5000/health

      - name: Set result output
        id: set-result
        if: always()
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            echo "result=passed" >> "$GITHUB_OUTPUT"
          else
            echo "result=failed" >> "$GITHUB_OUTPUT"
          fi
```

**Explained simply:**
- `on: workflow_call:` → this is what makes the file "reusable." It means this workflow does NOT run by itself. It only runs when another workflow calls it, like calling a function in programming.
- `inputs:` → the values the calling workflow can pass in. `python_version` (what Python version to use) and `run_tests` (true/false — should we actually run tests, or skip them).
- `outputs:` → this workflow can hand back a result (`test_result`) to whoever called it, so the caller knows if tests passed or failed.
- `jobs: build-test:` → the actual job. `runs-on: ubuntu-latest` means it runs on a fresh Ubuntu virtual machine provided free by GitHub.
- `actions/checkout@v4` → downloads (checks out) our repo's code onto that virtual machine, so later steps have access to `app.py`, `Dockerfile`, etc.
- `actions/setup-python@v5` → installs the requested Python version onto the machine.
- `pip install -r requirements.txt` → installs Flask.
- `Run tests` step → starts the Flask app in the background (`&`), waits 3 seconds so it has time to boot, then uses `curl --fail` to hit `/health`. `--fail` means "if the server responds with an error code, treat this step as failed" — this is our "test."
- `if: ${{ inputs.run_tests == true }}` → this step only runs if the caller asked for tests to run. This lets us skip tests in some situations if needed.
- `set-result` step → after everything, this checks `job.status` (success or failure) and writes `passed` or `failed` into `$GITHUB_OUTPUT`, which becomes the workflow's official output value that other workflows can read.

---

## 5. Task 3 — `reusable-docker.yml`

```yaml
name: Reusable Docker Build & Push

on:
  workflow_call:
    inputs:
      image_name:
        required: true
        type: string
      tag:
        required: true
        type: string
    secrets:
      docker_username:
        required: true
      docker_token:
        required: true
    outputs:
      image_url:
        description: "Full path of the pushed image"
        value: ${{ jobs.docker.outputs.url }}

jobs:
  docker:
    runs-on: ubuntu-latest
    outputs:
      url: ${{ steps.set-url.outputs.url }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.docker_username }}
          password: ${{ secrets.docker_token }}

      - name: Build and push image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}

      - name: Set image URL output
        id: set-url
        run: |
          echo "url=${{ secrets.docker_username }}/${{ inputs.image_name }}:${{ inputs.tag }}" >> "$GITHUB_OUTPUT"
```

**Explained simply:**
- `secrets: docker_username / docker_token:` → this workflow requires two secret values to be passed in. Secrets are stored safely in GitHub (Settings → Secrets and variables → Actions) and never shown in logs.
- `docker/login-action@v3` → an official, ready-made GitHub Action that logs into Docker Hub using our username and token, so we're allowed to push images.
- `docker/build-push-action@v6` → another official Action that builds the Docker image from our `Dockerfile` (`context: .` means "use the current folder") and pushes it straight to Docker Hub in one step (`push: true`).
- `tags:` → we combine the username, image name, and tag into the full Docker Hub path, e.g. `rohit123/myapp:latest`.
- `set-url` step → saves that full image path as an output (`image_url`) so `main-pipeline.yml` can print it later in the deploy step.

**Screenshots:**

![Docker Hub access token setup](./Screenshots/docker-access-token-setup.png)
*Creating the Docker Hub access token used for `docker_token`.*

![GitHub Actions secrets setup](./Screenshots/github-actions-secrets-setup.png)
*Adding `DOCKER_USERNAME` and `DOCKER_TOKEN` as repo secrets in GitHub Settings.*

---

## 6. Task 4 — `pr-pipeline.yml`

```yaml
name: PR Pipeline

on:
  pull_request:
    branches: [main]
    types: [opened, synchronize]

jobs:
  build-test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      run_tests: true

  pr-comment:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - name: Print PR check summary
        run: |
          echo "PR checks passed for branch: ${{ github.head_ref }}"
```

**Explained simply:**
- `on: pull_request: branches: [main], types: [opened, synchronize]` → this workflow only triggers when a PR is opened targeting `main`, or when new commits are pushed to an already-open PR (`synchronize`).
- `build-test: uses: ./.github/workflows/reusable-build-test.yml` → this is how we "call" our reusable workflow from Task 2, using its file path instead of copy-pasting all its steps again.
- `with: run_tests: true` → we explicitly tell it to run tests for every PR.
- `pr-comment: needs: build-test` → this second job only starts after `build-test` finishes successfully. `needs:` is how we control the order of jobs.
- `github.head_ref` → a built-in variable that holds the name of the branch that opened the PR.
- **Important:** notice this file never calls `reusable-docker.yml`. That's on purpose — we do NOT want to push a Docker image just because someone opened a PR.

**Screenshots:**

![Pull request page before creation](./Screenshots/pull-request-page-before-creation.png)
*About to open a PR into `main` to trigger `pr-pipeline.yml`.*

![Pull request form filled](./Screenshots/pull-request-form-filled.png)
*PR title/description filled in, ready to create.*

![PR pipeline running in progress](./Screenshots/pr-pipeline-running-in-progress.png)
*`pr-pipeline.yml` executing on GitHub Actions.*

![PR pipeline job flow diagram](./Screenshots/pr-pipeline-job-flow-diagram.png)
*GitHub's visual graph showing `build-test` → `pr-comment`.*

![PR comment final output log](./Screenshots/pr-comment-final-output-log.png)
*Log output of the `pr-comment` job printing "PR checks passed for branch: ...".*

![PR pipeline all checks passed](./Screenshots/pr-pipeline-all-checks-passed.png)
*All PR checks green — confirms no Docker build/push happened, tests only.*

![Test PR closed and branch deleted](./Screenshots/test-pr-closed-branch-deleted.png)
*Cleaning up — closing the test PR and deleting the temporary branch after confirming it worked.*

---

## 7. Task 5 — `main-pipeline.yml`

```yaml
name: Main Pipeline

on:
  push:
    branches: [main]

jobs:
  test:
    uses: ./.github/workflows/reusable-build-test.yml
    with:
      run_tests: true

  docker:
    needs: test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: myapp
      tag: latest
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  docker-sha:
    needs: test
    uses: ./.github/workflows/reusable-docker.yml
    with:
      image_name: myapp
      tag: sha-${{ github.sha }}
    secrets:
      docker_username: ${{ secrets.DOCKER_USERNAME }}
      docker_token: ${{ secrets.DOCKER_TOKEN }}

  deploy:
    needs: [docker, docker-sha]
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Announce deployment
        run: |
          echo "Deploying image: ${{ needs.docker.outputs.image_url }} to production"
```

**Explained simply:**
- `on: push: branches: [main]` → this only runs when code is actually pushed/merged into `main` — not on every PR.
- `test:` job → runs the same reusable test workflow again, as a safety net (tests should pass on `main` too, not just in the PR).
- `docker:` job → `needs: test` means it waits for tests to pass first. It calls `reusable-docker.yml` and passes it `image_name: myapp` and `tag: latest`, plus the two secrets it needs.
- `docker-sha:` job → does almost the same thing, but tags the image with the short commit SHA instead of `latest` (e.g. `sha-abc1234`). This way we always have a specific, traceable version of the image, not just a moving `latest` tag.
- `${{ github.sha }}` → the full commit hash of this push. (The hint in the task suggested shortening it with `cut -c1-7`; here the reusable workflow's `tag` input receives the value as-is — shortening can be added as an extra step if a shorter tag is preferred.)
- `deploy:` job → `needs: [docker, docker-sha]` waits for both image pushes to finish. `environment: production` ties this job to a GitHub "Environment" named `production`. If that environment has "Required reviewers" turned on (Settings → Environments), the job will **pause and wait for a human to click Approve** before it continues — this is what gives us manual approval before deploying.
- `needs.docker.outputs.image_url` → this is how we read the output that `reusable-docker.yml` produced earlier and print it in the deploy step.

**Screenshots:**

![Main pipeline first run failed](./Screenshots/main-pipeline-first-run-failed.png)
*First run — failed at the Docker step (auth error, see Troubleshooting Issue 1).*

![Actions tab showing main pipeline failure](./Screenshots/actions-tab-main-pipeline-failure.png)
*Failure summary across jobs in the Actions tab.*

![Main pipeline re-run in progress](./Screenshots/main-pipeline-rerun-in-progress.png)
*Re-run in progress after fixing the Docker Hub secrets.*

![Main pipeline final success](./Screenshots/main-pipeline-final-success.png)
*Final successful run — `test`, `docker`, `docker-sha`, and `deploy` all green.*

![Actions tab workflow run success](./Screenshots/actions-tab-workflow-run-success.png)
*Overall workflow run list showing the successful run.*

![GitHub Actions pipeline success](./Screenshots/github_actions_pipeline_success.png)
*Full pipeline graph, all jobs passed.*

![Docker Hub repositories](./Screenshots/docker_hub_repositories.png)
*Docker Hub account showing the `myapp` repository after the first successful push.*

![Docker Hub myapp tags](./Screenshots/docker_hub_myapp_tags.png)
*Tags list for `myapp` — `latest` and `sha-<commit>` both visible.*

![Docker Hub myapp tags detail](./Screenshots/docker_hub_myapp_tags_detail.png)
*Detail view of a specific tag, confirming the push timestamp and digest.*

![Docker pull and run container](./Screenshots/docker_pull_and_run_container.png)
*On the EC2 server: pulling the image and running the container.*

![App health check OK](./Screenshots/app_health_check_ok.png)
*`curl http://localhost:5000/health` returning `{"status": "ok"}` on the deployed server (see Troubleshooting Issue 4 for the earlier 404 confusion).*

---

## 8. Task 6 — `health-check.yml`

```yaml
name: Scheduled Health Check

on:
  schedule:
    - cron: "0 */12 * * *"
  workflow_dispatch:

jobs:
  health-check:
    runs-on: ubuntu-latest
    steps:
      - name: Pull latest image
        run: docker pull ${{ secrets.DOCKER_USERNAME }}/myapp:latest

      - name: Run container
        run: docker run -d --name myapp -p 5000:5000 ${{ secrets.DOCKER_USERNAME }}/myapp:latest

      - name: Wait for app to start
        run: sleep 5

      - name: Check health endpoint
        id: check
        run: |
          if curl --fail http://localhost:5000/health; then
            echo "status=PASSED" >> "$GITHUB_OUTPUT"
          else
            echo "status=FAILED" >> "$GITHUB_OUTPUT"
          fi

      - name: Write summary
        run: |
          echo "## Health Check Report" >> $GITHUB_STEP_SUMMARY
          echo "- Image: myapp:latest" >> $GITHUB_STEP_SUMMARY
          echo "- Status: ${{ steps.check.outputs.status }}" >> $GITHUB_STEP_SUMMARY
          echo "- Time: $(date)" >> $GITHUB_STEP_SUMMARY

      - name: Stop and remove container
        if: always()
        run: |
          docker stop myapp || true
          docker rm myapp || true
```

**Explained simply:**
- `schedule: cron: "0 */12 * * *"` → cron is a way of writing "run at this time." This specific pattern means "at minute 0, every 12th hour" — so it runs at (roughly) midnight and noon every day.
- `workflow_dispatch:` → adds a manual "Run workflow" button in the GitHub Actions tab, so we can test it any time instead of waiting 12 hours.
- `docker pull ...:latest` → downloads the newest pushed image from Docker Hub.
- `docker run -d --name myapp -p 5000:5000 ...` → starts the container in detached mode (`-d`, meaning it runs in the background instead of blocking the terminal), names it `myapp`, and maps port 5000 on the runner to port 5000 inside the container (`-p 5000:5000`).
- `sleep 5` → waits 5 seconds so Flask has time to fully start before we check it.
- `check` step → uses curl on `/health` (not `/`, since that's the only route that exists). Based on success/failure it writes `PASSED` or `FAILED` as an output.
- `$GITHUB_STEP_SUMMARY` → a special file GitHub gives every job. Anything written into it (in Markdown) shows up as a nicely formatted "Summary" on the Actions run page — no need to dig through raw logs.
- `if: always()` on the last step → makes sure the container is stopped and removed even if an earlier step failed, so we don't leave leftover containers running forever on the same runner.

**Screenshots:**

![GitHub Actions health check workflow](./Screenshots/github_actions_health_check_workflow.png)
*`health-check.yml` triggered manually via `workflow_dispatch` and completing successfully.*

![health-check.yml workflow](./Screenshots/health-check.yml%20workflow.png)
*Job summary page showing the `$GITHUB_STEP_SUMMARY` report (Image, Status: PASSED, Time).*

---

## 9. Troubleshooting Log — Real errors I hit, and how I fixed them

### Issue 1: Docker Hub authentication failure

**Error message:**
```
Error response from daemon: Get "https://registry-1.docker.io/v2/": unauthorized: incorrect username or password
```

**What this means in simple words:** GitHub Actions tried to log in to Docker Hub using the `DOCKER_USERNAME` and `DOCKER_TOKEN` secrets, but Docker Hub rejected them.

**Cause:** The token stored in GitHub Secrets was wrong or had expired. Docker Hub access tokens can expire or be revoked, and typing the username/token wrong when first creating the secret is a very common mistake.

**Fix:**
1. Logged in to Docker Hub → Account Settings → Security → created a brand-new Access Token.
2. Went to the GitHub repo → Settings → Secrets and variables → Actions.
3. Updated `DOCKER_TOKEN` with the new value (and double-checked `DOCKER_USERNAME` was correct).
4. Re-ran the failed workflow from the Actions tab.

**Result:** All jobs turned green on the re-run — `test`, `docker`, `docker-sha`, and `deploy` all passed.

![First run failed](./Screenshots/main-pipeline-first-run-failed.png)
![Failure summary](./Screenshots/actions-tab-main-pipeline-failure.png)
![New Docker Hub token created](./Screenshots/docker-access-token-setup.png)
![Secret updated in GitHub](./Screenshots/github-actions-secrets-setup.png)
![Re-run in progress](./Screenshots/main-pipeline-rerun-in-progress.png)
![Final success](./Screenshots/main-pipeline-final-success.png)

**Lesson:** Never store a password directly — always use a Docker Hub **access token**, not your real account password, since tokens can be revoked individually without changing your main password.

---

### Issue 2: VS Code showing "Unable to find reusable workflow" errors

**What I saw:** VS Code's YAML/GitHub Actions extension underlined lines like `uses: ./.github/workflows/reusable-build-test.yml` with a red squiggly line and the message "Unable to find reusable workflow," even though the file clearly existed in the folder.

**Cause:** This is a **false positive**. VS Code's extension checks these paths *locally on your laptop*, and depending on which branch you currently have checked out, or simply due to the extension's own limitations with relative paths, it sometimes cannot "see" the reusable workflow file even though it is genuinely there and works fine once pushed to GitHub.

**How I confirmed it was a false positive:** I pushed the workflow to GitHub and ran it — GitHub Actions found the reusable workflow file with no problem and executed it successfully. The error only existed inside the VS Code editor, not in real life.

**Fix / takeaway:** Ignore this specific warning as long as:
- the file path after `uses:` exactly matches the real file location, and
- the workflow actually runs successfully on GitHub.

No code change was needed — it was purely an editor display issue.

---

### Issue 3: Wrong branch — changes not appearing on `main`

**What happened:** After building `main-pipeline.yml`, pushing to GitHub didn't seem to update `main` — because the terminal prompt showed I was still on a different branch, `test-pr-pipeline`, and the new file had never been committed there.

**Fix (step by step):**
```bash
git add .
git commit -m "Add main-pipeline.yml workflow"
git checkout main
git merge test-pr-pipeline
git push origin main
```

**In simple words:**
- `git add .` → mark all changed files as "ready to be saved."
- `git commit -m "..."` → actually save them, with a short message describing what changed.
- `git checkout main` → switch to the `main` branch.
- `git merge test-pr-pipeline` → bring all the new commits from `test-pr-pipeline` into `main`.
- `git push origin main` → upload the updated `main` branch to GitHub.

**Lesson:** Always check *which branch you're on* (shown in brackets in the terminal prompt) before assuming a file has been saved to the branch you expect.

---

### Issue 4: `curl http://localhost:5000` returned 404 Not Found

**What happened:** After deploying to the AWS EC2 server and running the container, `curl http://localhost:5000` returned a 404 error, which looked like the deployment had failed.

**Cause:** The app was working perfectly — but it has no route defined for `/` (the root address). The only route that exists is `/health`. A 404 on `/` is completely expected behavior, not a bug.

**Fix:** Ran the correct command:
```bash
curl http://localhost:5000/health
```
This returned `{"status": "ok"}` — confirming the deployment actually worked.

![Pull and run container on EC2](./Screenshots/docker_pull_and_run_container.png)
![Health check OK](./Screenshots/app_health_check_ok.png)

**Lesson:** Always double-check *which route* an app actually defines before assuming a 404 means something is broken.

---

## 10. All Screenshots (full list)

All screenshots live in `2026/day-48/Screenshots/` next to this file, so every image link above uses the relative path `./Screenshots/<file-name>.png`. Full list, in the order they were captured:

| File name | Used in |
|---|---|
| `pull-request-page-before-creation.png` | Task 4 — before opening the test PR |
| `pull-request-form-filled.png` | Task 4 — PR form filled in |
| `pr-pipeline-running-in-progress.png` | Task 4 — `pr-pipeline.yml` running |
| `pr-pipeline-job-flow-diagram.png` | Task 4 — job graph (`build-test` → `pr-comment`) |
| `pr-comment-final-output-log.png` | Task 4 — `pr-comment` job log output |
| `pr-pipeline-all-checks-passed.png` | Task 4 — all PR checks green |
| `test-pr-closed-branch-deleted.png` | Task 4 — cleanup after testing |
| `docker-access-token-setup.png` | Task 3 / Issue 1 — new Docker Hub access token |
| `github-actions-secrets-setup.png` | Task 3 / Issue 1 — GitHub repo secrets |
| `main-pipeline-first-run-failed.png` | Task 5 / Issue 1 — first failed run (Docker auth error) |
| `actions-tab-main-pipeline-failure.png` | Task 5 / Issue 1 — failure summary |
| `main-pipeline-rerun-in-progress.png` | Task 5 / Issue 1 — re-run after fix |
| `main-pipeline-final-success.png` | Task 5 / Issue 1 — final successful run |
| `actions-tab-workflow-run-success.png` | Task 5 — workflow run list, success |
| `github_actions_pipeline_success.png` | Task 5 — full pipeline graph, all green |
| `docker_hub_repositories.png` | Task 5 — Docker Hub repo after first push |
| `docker_hub_myapp_tags.png` | Task 5 — `latest` and `sha-<commit>` tags |
| `docker_hub_myapp_tags_detail.png` | Task 5 — tag detail view |
| `docker_pull_and_run_container.png` | Task 5 / Issue 4 — pulling & running on EC2 |
| `app_health_check_ok.png` | Task 5 / Issue 4 — `/health` returning `{"status": "ok"}` |
| `github_actions_health_check_workflow.png` | Task 6 — manual run of `health-check.yml` |
| `health-check.yml workflow.png` | Task 6 — `$GITHUB_STEP_SUMMARY` report |

---

## 11. Docker Hub Image

Pushed image:
```
https://hub.docker.com/repository/docker/rtingane2611/myapp/general
```

---

## 12. Submission Checklist

- [x] `github-actions-capstone` repo created
- [x] Flask app + Dockerfile + basic test
- [x] `reusable-build-test.yml`
- [x] `reusable-docker.yml`
- [x] `pr-pipeline.yml` (tested with a real PR)
- [x] `main-pipeline.yml` (tested with a real merge to `main`, deployed to EC2)
- [x] `health-check.yml` (tested manually via `workflow_dispatch`)
- [x] Status badges added to `README.md`
- [x] This file (`day-48-actions-project.md`) added to `2026/day-48/`
