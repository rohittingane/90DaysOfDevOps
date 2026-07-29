# Day 30 - Docker Images & Container Lifecycle

## Objective

Today I learned how Docker images and containers work internally.

Topics covered:

- Relationship between Docker Images and Containers
- Image layers and caching
- Complete container lifecycle
- Working with running containers
- Docker cleanup

---

# Core Concepts

## Relationship Between Images and Containers

### What?

A Docker **image** is a read-only template that contains the application code, dependencies, libraries, and configuration needed to run something. A **container** is a running (or stopped) instance created from that image.

### Why?

Understanding this relationship matters because:

- One image can be used to create many containers.
- Containers add a thin writable layer on top of the read-only image.
- Changes made inside a container do not affect the original image.
- If a container is deleted, the image still remains and can be used to create a new container.

### How it works

```
Docker Image (read-only)
        |
        | docker run / docker create
        v
Container (image + writable layer)
```

Analogy: an image is like a **class**, and a container is like an **object/instance** created from that class.

---

## Image Layers and Caching

### What?

Docker images are made up of multiple stacked, read-only **layers**. Each instruction in a Dockerfile (`FROM`, `RUN`, `COPY`, `ENV`, etc.) creates a new layer on top of the previous one.

### Why?

Layers exist so Docker can:

- Reuse layers that haven't changed instead of rebuilding them.
- Download only the layers that are missing when pulling an image.
- Save disk space by sharing common layers across multiple images.
- Speed up builds using the **build cache** — if a layer's instruction and its inputs haven't changed, Docker reuses the cached layer instead of re-executing it.

### How?

```bash
docker image history nginx
```

This shows every layer used to build the image, along with its size. Layers that only change metadata (like `CMD`, `ENV`, `EXPOSE`) show `0B` because they don't add filesystem content.

Caching rule of thumb: if you change an early instruction in a Dockerfile, every layer after it gets invalidated and rebuilt — so stable instructions (like installing dependencies) should come before frequently-changing instructions (like copying source code).

---

## Container Lifecycle

### What?

A container moves through several distinct states from creation to removal.

### Why?

Knowing the lifecycle helps in debugging, controlling resource usage, and managing containers properly instead of just running/stopping them blindly.

### How? (states and commands)

```
docker create   -> Created
docker start    -> Running
docker pause    -> Paused
docker unpause  -> Running
docker stop     -> Exited (graceful)
docker restart  -> Running
docker kill     -> Exited (forced)
docker rm       -> Removed
```

Checked after every step using:

```bash
docker ps -a
```

This lifecycle mastery makes it possible to create a container without starting it, freeze/unfreeze it, gracefully vs forcefully stop it, and clean it up when no longer needed.

---

# Task 1: Docker Images

## Pull Docker Images

### What?

Downloaded nginx, ubuntu, and alpine images from Docker Hub.

### Command

```bash
docker pull nginx
docker pull ubuntu
docker pull alpine
```

### Output

```
nginx:latest
ubuntu:latest
alpine:latest

Status: Downloaded newer image
docker.io/library/nginx:latest
docker.io/library/ubuntu:latest
docker.io/library/alpine:latest
```

### Screenshot

![Pull Docker Images](Screenshots/day30-01-pull-docker-images.png)

---

## List Docker Images

### Command

```bash
docker images
```

### Output

```
IMAGE           SIZE

alpine:latest   8.42MB
ubuntu:latest   100MB
nginx:latest    161MB
```

### Observation

- Alpine image is very small because it is a minimal Linux distribution.
- Ubuntu image is larger because it contains more packages and utilities.
- Nginx image contains the Nginx web server and required dependencies.

### Screenshot

![List Docker Images](Screenshots/day30-02-list-docker-images.png)

---

## Inspect Docker Image

### Command

```bash
docker inspect nginx
```

### Information observed

Docker image inspect provides:

- Image ID
- Created date
- Architecture
- Environment variables
- Exposed ports
- Entry point
- Image layers
- Size information

### Screenshot

![Inspect Docker Image](Screenshots/day30-03-inspect-nginx-image.png)

---

## Remove Docker Image

### What?

Removed an unused Ubuntu image.

Initially image removal failed because a container was using that image.

Error:

```
conflict: unable to remove repository reference
container is using its referenced image
```

Removed container first:

```bash
docker rm container_id
```

Then removed image:

```bash
docker rmi ubuntu
```

### Screenshot

![Remove Docker Image](Screenshots/day30-04-remove-ubuntu-image.png)

---

# Task 2: Docker Image Layers

### Command

```bash
docker image history nginx
```

### Observation

Docker image history shows every layer used to build an image.

Some layers show size:

```
RUN command        82.7MB
COPY files         KB
```

Some layers show:

```
0B
```

because they only store metadata changes like:

- CMD
- ENV
- ENTRYPOINT
- EXPOSE

### What are Docker Layers?

Docker images are built using multiple read-only layers.

Each Dockerfile instruction creates a new layer.

```
Base Image
    |
    Layer 1
    |
    Layer 2
    |
    Layer 3
    |
Final Image
```

### Why Docker uses Layers?

- Faster image building
- Reuses unchanged layers
- Saves disk space
- Improves caching

### Screenshot

![Image History Nginx](Screenshots/day30-05-image-history-nginx.png)

---

# Task 3: Container Lifecycle

Practiced complete lifecycle using an nginx container.

## Create Container Without Starting

```bash
docker create --name lifecycle-test nginx
```

Status: `Created`

![Create Container](Screenshots/day30-06-container-create.png)

While experimenting, tried creating a container with a name that was already in use, which produced an error:

```
docker: Error response from daemon: Conflict. The container name "/lifecycle-test" is already in use
```

![Container Create Error](Screenshots/day30-06-container-create-error.png)

## Start Container

```bash
docker start lifecycle-test
```

Status: `Up and Running`

![Start Container](Screenshots/day30-07-container-start.png)

## Pause Container

```bash
docker pause lifecycle-test
```

Status: `Up (Paused)`

![Pause Container](Screenshots/day30-08-container-pause.png)

## Unpause Container

```bash
docker unpause lifecycle-test
```

Status: `Running`

![Unpause Container](Screenshots/day30-09-container-unpause.png)

## Stop Container

```bash
docker stop lifecycle-test
```

Status: `Exited`

![Stop Container](Screenshots/day30-10-container-stop.png)

## Restart Container

```bash
docker restart lifecycle-test
```

Status: `Running`

![Restart Container](Screenshots/day30-11-container-restart.png)

## Kill Container

```bash
docker kill lifecycle-test
```

Status: `Exited (137)`

![Kill Container](Screenshots/day30-12-container-kill.png)

## Remove Container

```bash
docker rm lifecycle-test
```

Container removed successfully.

![Remove Container](Screenshots/day30-13-container-remove.png)

---

# Task 4: Working With Running Containers

## Run Nginx Container

```bash
docker run -d --name nginx-running -p 8080:80 nginx
```

Started nginx container in detached mode.

![Run Nginx Detached](Screenshots/day30-14-nginx-detached-container.png)

## View Container Logs

```bash
docker logs nginx-running
```

Logs show:

- Nginx startup
- Configuration process
- Worker processes

![View Logs](Screenshots/day30-15-container-logs.png)

## Follow Real-Time Logs

```bash
docker logs -f nginx-running
```

Used follow mode to monitor live logs (no separate screenshot for this step — same output pattern as `docker logs`, just streamed live).

## Execute Command Inside Container

```bash
docker exec -it nginx-running bash
```

Checked container filesystem:

```bash
ls
ls /etc/nginx
```

![Exec Into Container](Screenshots/day30-17-exec-into-container.png)

## Run Single Command Inside Container

```bash
docker exec nginx-running ls /etc/nginx
```

Output:

```
conf.d
fastcgi_params
mime.types
nginx.conf
```

![Run Single Command](Screenshots/day30-18-exec-single-command.png)

## Inspect Container

```bash
docker inspect nginx-running
```

Information found:

- Container ID
- IP Address
- Port Mapping
- Mounts
- Network configuration

![Inspect Container](Screenshots/day30-19-container-inspect.png)

---

# Task 5: Docker Cleanup

## Stop All Running Containers

```bash
docker stop $(docker ps -q)
```

Stopped all running containers.

![Stop All Containers](Screenshots/day30-23-stop-all-running-containers.png)

## Remove Stopped Containers

```bash
docker container prune
```

Removed unused stopped containers.

![Remove Stopped Containers](Screenshots/day30-24-remove-stopped-containers.png)

## Remove Unused Images

```bash
docker image prune -a
```

Output:

```
Total reclaimed space: 169.7MB
```

![Remove Unused Images](Screenshots/day30-25-remove-unused-images.png)

## Check Docker Disk Usage

```bash
docker system df
```

Output:

```
Images          0B
Containers      0B
Local Volumes   0B
Build Cache     0B
```

Docker environment cleaned successfully.

![Docker Disk Usage](Screenshots/day30-26-docker-disk-usage.png)

---

# Key Learnings

- Docker images are templates used to create containers.
- Containers are running instances of images.
- Images are built using multiple layers.
- Docker reuses layers using caching.
- Containers have different lifecycle states.
- Proper cleanup helps save disk space.

# Conclusion

Day 30 helped me understand Docker images, layers, container lifecycle, container debugging, and cleanup practices through hands-on commands.
