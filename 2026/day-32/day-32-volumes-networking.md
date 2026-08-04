# Day 32 – Docker Volumes & Networking

## Goal
Solve two real Docker problems:
1. **Data Persistence** – containers lose data when removed.
2. **Container Communication** – containers can't easily talk to each other by name.

---

## Task 1: The Problem (Data Loss)

### Concept
A container's writable layer holds any data created inside it (databases, files, etc.). When the container is **removed**, that writable layer is destroyed — and so is the data. This is Docker's default, expected behavior, since containers are designed to be *ephemeral* (temporary).

### Steps & Commands

**1. Run a MySQL container**
```bash
docker run -d -e MYSQL_ROOT_PASSWORD=ROOT mysql:latest
```
This starts a MySQL container in the background (`-d`), with root password set via an environment variable.

**2. Enter the container and create data**
```bash
docker exec -it <container_id> bash
mysql -u root -pROOT
```
Inside the MySQL shell:
```sql
CREATE DATABASE devops;
USE devops;
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50),
    role VARCHAR(50)
);
INSERT INTO employees (name, role)
VALUES
('Rohit', 'DevOps Engineer'),
('Amit', 'Developer'),
('Priya', 'QA Engineer');

SELECT * FROM employees;
```

**Output:**
```
+----+-------+-----------------+
| id | name  | role            |
+----+-------+-----------------+
|  1 | Rohit | DevOps Engineer |
|  2 | Amit  | Developer       |
|  3 | Priya | QA Engineer     |
+----+-------+-----------------+
```
Data successfully created ✅

**3. Stop and remove the container**
```bash
docker stop <container_id>
docker rm <container_id>
```

**4. Run a brand-new container and check if data survived**
```bash
docker run -d -e MYSQL_ROOT_PASSWORD=ROOT mysql:latest
docker exec -it <new_container_id> bash
mysql -u root -p
SHOW DATABASES;
```

**Output:**
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

### Result
❌ The `devops` database is **gone**. The `employees` table and all its rows disappeared completely.

### Why this happened
MySQL stores its data inside the container at `/var/lib/mysql`. This path lives inside the container's writable layer. When `docker rm` deletes the container, it deletes this layer entirely — along with every byte of data that was written to it. There was no external storage backing it up, so nothing survived.

### Screenshots
- `task1-data-created.png` – database, table, and rows created
- `task1-container-removed.png` – `docker stop` + `docker rm` + `docker ps -a` confirming removal
- `task1-new-container-data-lost.png` – new container's `SHOW DATABASES` missing `devops`

---

## Task 2: Named Volumes

### Concept
A **named volume** is storage that Docker manages *outside* the container's writable layer, on the host machine (typically under `/var/lib/docker/volumes/<name>/_data`). Because it exists independently of any single container, the volume — and its data — survives even if the container using it is deleted.

### Steps & Commands

**1. Create a named volume**
```bash
docker volume create mysql-data
```
Output: `mysql-data`

**2. Verify it exists**
```bash
docker volume ls
```
```
DRIVER    VOLUME NAME
local     mysql-data
```

**3. Run a MySQL container, attaching the volume**
```bash
docker run -d -e MYSQL_ROOT_PASSWORD=ROOT \
  -v mysql-data:/var/lib/mysql \
  --name mysql-with-volume mysql:latest
```

**Command explained:**
| Part | Meaning |
|---|---|
| `-v mysql-data:/var/lib/mysql` | Maps host-managed volume `mysql-data` to MySQL's internal data folder `/var/lib/mysql` |
| `--name mysql-with-volume` | Gives the container an easy-to-reference name |

Whatever MySQL writes to `/var/lib/mysql` now physically lands inside the `mysql-data` volume instead of the container's own writable layer.

> **Troubleshooting note:** During practice, the container kept exiting with error `No space left on device`. `df -h` confirmed the root disk was 100% full. Fixed by running:
> ```bash
> docker system df        # check reclaimable space
> docker volume prune     # remove unused/orphaned volumes
> docker container prune  # remove stopped containers
> docker image prune -a   # remove unused images
> ```
> This freed enough space (99% → 84% used) for MySQL to initialize successfully.

**4. Add data**
```bash
docker exec -it mysql-with-volume mysql -uroot -pROOT
```
```sql
CREATE DATABASE impdata;
SHOW DATABASES;
USE impdata;
CREATE TABLE employees (id INT, name VARCHAR(50), role VARCHAR(50));
```

**5. Stop and remove the container**
```bash
docker stop mysql-with-volume
docker rm mysql-with-volume
```

**6. Run a brand-new container with the SAME volume**
```bash
docker run -d -e MYSQL_ROOT_PASSWORD=ROOT \
  -v mysql-data:/var/lib/mysql \
  --name mysql-volume-test2 mysql:latest

docker exec -it mysql-volume-test2 mysql -uroot -pROOT
```
```sql
SHOW DATABASES;
```

**Output:**
```
+--------------------+
| Database           |
+--------------------+
| impdata             |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

### Result
✅ `impdata` database **survived** — a completely different, brand-new container could see data created by the earlier (now-deleted) container, purely because both were attached to the same named volume.

### Verification commands
```bash
docker volume ls
docker volume inspect mysql-data
```
`docker volume inspect` shows key details:
```json
"Mountpoint": "/var/lib/docker/volumes/mysql-data/_data",
"Name": "mysql-data",
"Driver": "local"
```
This `Mountpoint` is the real, physical location on the host machine where the data actually lives.

### Screenshots
- `task2-volume-created.png`
- `task2-fresh-container-with-volume.png`
- `task2-database-created.png`
- `task2-container-removed.png`
- `task2-data-survived-new-container.png`
- `task2-volume-inspect.png`

---

## Task 3: Bind Mounts

### Concept
A **bind mount** links a specific folder on the host machine directly to a folder inside the container. Unlike a named volume (which Docker manages internally at a hidden location), a bind mount uses a path *you* choose and can inspect/edit directly.

### Steps & Commands

**1. Create a folder + HTML file on the host**
```bash
mkdir ~/nginx-site
echo "<h1>Hello from Bind Mount - Rohit's Nginx</h1>" > ~/nginx-site/index.html
cat ~/nginx-site/index.html
```

**2. Run an Nginx container with the folder bind-mounted**
```bash
docker run -d --name my-nginx -p 8080:80 \
  -v ~/nginx-site:/usr/share/nginx/html \
  nginx:latest
```

**Command explained:**
| Part | Meaning |
|---|---|
| `-p 8080:80` | Maps host port `8080` → container port `80` (where Nginx serves traffic) |
| `-v ~/nginx-site:/usr/share/nginx/html` | Bind-mounts host folder `~/nginx-site` to Nginx's default web root inside the container |

**3. Access the page in the browser**
```
http://<server-public-ip>:8080
```
Output: page displayed **"Hello from Bind Mount - Rohit's Nginx"**

**4. Edit the file on the host and refresh**
```bash
echo "<h1>Updated Live! Bind Mount is Working - Rohit</h1>" > ~/nginx-site/index.html
```
Refreshed the browser (`F5`) — **no container restart needed** — and the updated text appeared immediately.

### Result
✅ Changes made directly to the host file were reflected instantly inside the running container, because the container's web folder and the host folder are literally the same underlying storage location.

### Named Volume vs Bind Mount — Key Difference

| | Named Volume | Bind Mount |
|---|---|---|
| Managed by | Docker (hidden location under `/var/lib/docker/volumes/`) | You (any folder you choose on the host) |
| Path visibility | Abstracted — Docker handles it | Fully visible/editable at a known host path |
| Best use case | Production data (databases) that Docker should manage safely | Development — live-editing code/config without rebuilding the image |
| Portability | Easier to back up/move via Docker commands | Tied to the exact host machine's file structure |

### Screenshots
- `task3-folder-file-created.png`
- `task3-nginx-container-running.png`
- `task3-browser-page-initial.png`
- `task3-browser-page-fixed.png`
- `task3-live-edit-updated.png`

---

## Task 4: Docker Networking Basics

### Concept
Docker automatically creates a **default `bridge` network**, and every container joins it unless told otherwise. Containers on this network **can reach each other by IP address, but not by container name** — the default bridge has no built-in DNS resolution service.

### Steps & Commands

**1. List all Docker networks**
```bash
docker network ls
```
```
NETWORK ID     NAME      DRIVER    SCOPE
477c5d44a8a3   bridge    bridge    local
31fde7ed0b63   host      host      local
cd633f317c7a   none      null      local
```

**2. Inspect the default `bridge` network**
```bash
docker network inspect bridge
```
Key details found:
```json
"Subnet": "172.17.0.0/16",
"Gateway": "172.17.0.1"
```
- **Subnet** (`172.17.0.0/16`) — the full range of IP addresses available to containers on this network (e.g. `172.17.0.1` to `172.17.255.254`).
- **Gateway** (`172.17.0.1`) — the network's main entry/exit point connecting containers to the host machine and outside world.

Existing containers (`mysql-volume-test2`, `my-nginx`) were listed with their assigned IPs (`172.17.0.2`, `172.17.0.3`).

**3. Try pinging by name (expected to fail)**
```bash
docker exec my-nginx ping -c 3 mysql-volume-test2
```
> Note: `ping` isn't installed in the slim `nginx` image by default, so first needed:
> ```bash
> docker exec my-nginx apt update
> docker exec my-nginx apt install -y iputils-ping
> ```

**Output:**
```
ping: mysql-volume-test2: Name or service not known
```
❌ Failed — confirms default bridge has no name-based DNS resolution.

**4. Ping by IP instead**
```bash
docker exec my-nginx ping -c 3 172.17.0.2
```

**Output:**
```
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.041 ms
--- 172.17.0.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```
✅ Success — IP-based communication works fine.

### Result Summary

| Method | Result |
|---|---|
| Ping by name | ❌ Failed – "Name or service not known" |
| Ping by IP | ✅ Success – 0% packet loss |

### Screenshots
- `task4-network-list.png`
- `task4-bridge-inspect.png`
- `task4-ping-name-vs-ip.png`

---

## Task 5: Custom Networks

### Concept
Creating a **custom bridge network** gives Docker an internal DNS resolver for that network. Containers attached to it can reach each other **by container name**, and Docker automatically keeps the name-to-IP mapping updated even if a container's IP changes on restart.

### Steps & Commands

**1. Create a custom bridge network**
```bash
docker network create my-app-net
```

**2. Verify**
```bash
docker network ls
```
```
NETWORK ID     NAME         DRIVER    SCOPE
...
9f650099cfe8   my-app-net   bridge    local
```

**3. Run two containers on it**
```bash
docker run -d --name app1 --network my-app-net nginx:latest
docker run -d --name app2 --network my-app-net nginx:latest
```

**4. Ping by name**
```bash
docker exec app1 apt update
docker exec app1 apt install -y iputils-ping
docker exec app1 ping -c 3 app2
```

**Output:**
```
PING app2 (172.18.0.3) 56(84) bytes of data.
64 bytes from app2.my-app-net (172.18.0.3): icmp_seq=1 ttl=64 time=0.054 ms
64 bytes from app2.my-app-net (172.18.0.3): icmp_seq=2 ttl=64 time=0.109 ms
64 bytes from app2.my-app-net (172.18.0.3): icmp_seq=3 ttl=64 time=0.050 ms
--- app2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```
✅ **Success!** `app1` resolved `app2`'s name to its IP automatically and pinged it successfully.

### Why does custom networking allow name resolution but the default bridge doesn't?

Docker only runs its **embedded DNS server** (127.0.0.11 inside each container) for **user-defined (custom) networks**. When a container joins a custom network, Docker registers its name against its current IP in that network's internal DNS table, and every other container on the same network can query it by name.

The default `bridge` network is a legacy network kept for backward compatibility, and it was never given this embedded DNS feature — it only supports raw IP-based communication (and optionally `--link`, which is deprecated).

This is exactly why real multi-container setups (and tools like `docker-compose`, which auto-creates a custom network per project) always rely on custom networks — so services can reference each other by name in configuration/code without hardcoding IPs that can change.

### Screenshots
- `task5-network-created-and-listed.png`
- `task5-two-containers-running.png`
- `task5-ping-by-name-success.png`

---

## Task 6: Put It Together

### Concept
Combine everything learned: a custom network (for name-based communication) + a database container backed by a volume (for data persistence) + an app container that reaches the database purely by name — a mini version of how real production multi-container apps are wired together.

### Steps & Commands

**1. Create a custom network**
```bash
docker network create shop-net
```

**2. Create a volume for the database**
```bash
docker volume create shop-db-data
```

**3. Run the database container on the network, with the volume attached**
```bash
docker run -d --name shop-db \
  -e MYSQL_ROOT_PASSWORD=ROOT \
  --network shop-net \
  -v shop-db-data:/var/lib/mysql \
  mysql:latest
```

> **Troubleshooting note:** The container initially exited with code `137` (killed) due to the host disk filling up again (`99% used`). Cleaned up with:
> ```bash
> docker container prune
> docker image prune -a
> ```
> This freed space (99% → 84%), and the container ran successfully afterward.

**4. Run an app container on the same network**
```bash
docker run -d --name shop-app --network shop-net nginx:latest
```

**5. Verify the app container can reach the database by name**
```bash
docker exec shop-app apt update
docker exec shop-app apt install -y iputils-ping
docker exec shop-app ping -c 3 shop-db
```

**Output:**
```
PING shop-db (172.19.0.2) 56(84) bytes of data.
64 bytes from shop-db.shop-net (172.19.0.2): icmp_seq=1 ttl=64 time=0.078 ms
64 bytes from shop-db.shop-net (172.19.0.2): icmp_seq=2 ttl=64 time=0.046 ms
64 bytes from shop-db.shop-net (172.19.0.2): icmp_seq=3 ttl=64 time=0.046 ms
--- shop-db ping statistics ---
3 packets transmitted, 3 received, 0% packet loss
```

### Result
✅ `shop-app` successfully reached `shop-db` **purely by container name** — no IP hardcoded anywhere. And because `shop-db` uses the `shop-db-data` volume, its data would survive even if the container were removed and recreated (as proven in Task 2).

### Screenshots
- `task6-network-and-volume-created.png`
- `task6-database-container-running.png`
- `task6-app-container-running.png`
- `task6-both-containers-running.png`
- `task6-ping-success-final.png`

---

## Overall Key Takeaways

1. **Containers are ephemeral by default** — any data written inside a container's own filesystem is lost the moment the container is removed.
2. **Named volumes** solve this by storing data outside the container's writable layer, on the host, managed by Docker — so data survives container removal and recreation.
3. **Bind mounts** link a specific host folder directly into a container — ideal for development, where live edits on the host should reflect instantly inside the container.
4. **The default bridge network** only supports container-to-container communication via IP address — it has no built-in DNS.
5. **Custom bridge networks** enable Docker's embedded DNS resolver, allowing containers to reach each other by name — regardless of IP changes on restart.
6. Combining a **custom network + a volume-backed database + an app container** mirrors how real production multi-container applications (e.g., via `docker-compose`) are architected — reliable data storage and name-based service discovery, without hardcoded IPs.

## Troubleshooting Learnings (real issues hit during practice)
- **Disk full errors** (`No space left on device`, exit code `137`) occurred twice due to a small root disk (6.8G) filling up with unused images/volumes/containers. Fixed each time using `docker system df` to check reclaimable space, then `docker container prune`, `docker volume prune`, and `docker image prune -a`.
- **`ping: executable file not found`** — the `nginx` and `mysql` base images are slim and don't include `ping` by default; had to install `iputils-ping` via `apt update && apt install -y iputils-ping` inside each container before testing connectivity.
- **`mysql -u root -p root`** vs **`mysql -u root -pROOT`** — a space after `-p` makes MySQL treat the next word as a database name, not part of the password flag; the password must be attached directly to `-p` with no space.

---


