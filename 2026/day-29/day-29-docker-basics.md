# Day 29 – Introduction to Docker

## Objective

Today's goal was to understand Docker fundamentals, learn why containers are used, install Docker, and run my first Docker containers.

I practiced running containers, managing container lifecycle, port mapping, logs, and executing commands inside containers.

---

## Task 1: What is Docker?

### What is Docker?

Docker is an open-source containerization platform that allows developers to package applications with their dependencies, libraries, and configurations into lightweight containers.

Docker helps applications run consistently across different environments like development, testing, and production.

---

### What is a Container?

A container is a lightweight, standalone, executable package that contains:

- Application code
- Runtime
- Libraries
- Dependencies
- Configuration files

Containers share the host operating system kernel, which makes them faster and more lightweight compared to Virtual Machines.

### Why do we need Containers?

**Without containers:**
- Applications may work on one system but fail on another.
- Dependency conflicts can occur.
- Environment setup becomes difficult.

**Containers provide:**
- Portability
- Faster deployment
- Lightweight environments
- Consistent application execution
- Better resource utilization

---

### Containers vs Virtual Machines

| Containers | Virtual Machines |
|------------|------------------|
| Share host OS kernel | Have their own OS |
| Lightweight | Heavyweight |
| Start in seconds | Take more time to boot |
| Use less memory | Require more resources |
| Faster deployment | Slower deployment |
| Best for microservices | Best for complete OS isolation |

---

### Docker Architecture

Docker architecture consists of:

**1. Docker Client**

Docker Client is the command-line interface used to communicate with Docker Daemon.

Example:
```bash
docker run nginx
```

**2. Docker Daemon**

Docker Daemon (`dockerd`) is the background service responsible for:
- Building images
- Creating containers
- Managing networks
- Managing storage
- Communicating with Docker Registry

**3. Docker Images**

Docker Image is a read-only template used to create containers.

Examples: `nginx`, `ubuntu`, `mysql`, `redis`

**4. Docker Containers**

Containers are running instances of Docker Images.

```
Ubuntu Image
      |
      ↓
docker run ubuntu
      |
      ↓
Ubuntu Container
```

**5. Docker Registry**

Docker Registry stores Docker Images.

Default registry: **Docker Hub**

Example:
```bash
docker pull nginx
```

#### Docker Architecture Diagram

```
              Docker Hub
          (Image Registry)
                 |
          Pull / Push Images
                 |
                 ↓
+--------------------------------+
|        Docker Daemon            |
|            dockerd               |
+--------------------------------+
        |          |          |
        ↓          ↓          ↓
   Container   Container   Container
     nginx       ubuntu      redis
```

---

## Task 2: Install Docker

### Install Docker

Docker was installed on an Ubuntu EC2 instance.

```bash
sudo apt update
sudo apt install docker.io -y
```

### Verify Docker Installation

```bash
docker --version
```

**Output:**
```
Docker version 28.x.x
```

![Docker installation verification](day29-01-docker-installation-verification.png)

### Check Docker Service Status

```bash
sudo systemctl status docker
```

**Output:**
```
Active: active (running)
```

Docker service was running successfully.

**Enable Docker service:**
```bash
sudo systemctl enable docker
```

**Verify:**
```bash
sudo systemctl is-enabled docker
```

**Output:**
```
enabled
```

![Docker service status](day29-02-docker-service-status.png)

### Run Hello World Container

```bash
docker run hello-world
```

**Output:**
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

**What happened:**
1. Docker searched for the `hello-world` image locally.
2. Image was not available.
3. Docker pulled the image from Docker Hub.
4. Docker created a container.
5. Container executed successfully.

![Hello world container](day29-03-hello-world-container.png)

---

## Task 3: Run Real Containers

### Run Nginx Container

```bash
docker run -d -p 8080:80 --name my-nginx nginx
```

**Explanation:**
- `-d` → Detached mode
- `-p` → Port mapping
- `--name` → Custom container name

### Verify Running Container

```bash
docker ps
```

**Output:**
```
CONTAINER ID   IMAGE   STATUS         PORTS
266b8ee69dc2   nginx   Up 24 minutes  0.0.0.0:8080->80/tcp
```

**Container details:**
```
Host Port 8080
       |
       ↓
Container Port 80
       |
       ↓
Nginx Server
```

### Access Nginx From Browser

**URL:**
```
http://13.60.99.26:8080
```

**Result:** `Welcome to nginx!`

Nginx container was successfully accessed from the browser.

![Nginx browser output](day29-07-nginx-browser-output.png)
![Nginx browser output 1](day29-07.1-nginx-browser-output.png)
![Nginx browser output 2](day29-07.2-nginx-browser-output.png)

### Run Ubuntu Interactive Container

```bash
docker run -it ubuntu bash
```

**Output:**
```
root@461d240a4ac5:/#
```

**Explored the container:**

**Check current directory**
```bash
pwd
```
Output: `/`

**List files**
```bash
ls
```
Output: `bin boot dev etc home lib usr var`

**Check OS information**
```bash
cat /etc/os-release
```
Output:
```
PRETTY_NAME="Ubuntu 26.04 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
```

**Check user**
```bash
whoami
```
Output: `root`

**Check hostname**
```bash
hostname
```
Output: `461d240a4ac5`

![Ubuntu container interactive](day29-08-ubuntu-container-interactive.png)

**List running containers**
```bash
docker ps
```
Output:
```
CONTAINER ID   IMAGE   STATUS
266b8ee69dc2   nginx   Up 24 minutes
```

**List all containers**
```bash
docker ps -a
```
Output:
```
CONTAINER ID   IMAGE
461d240a4ac5   ubuntu
266b8ee69dc2   nginx
4a64a631f338   hello-world
```

**Stop container**
```bash
docker stop my-nginx
```
Output: `my-nginx`

**Remove container**
```bash
docker rm my-nginx
```
Output: `my-nginx`

![Docker container management](day29-09-docker-container-management.png)

---

## Task 4: Docker Exploration

### Detached Mode

```bash
docker run -d nginx
```

**Output:**
```
9c972f643b55127b2b6796c121e486b0fbeda78dede1d8ccd23059fc42aedef2
```

**Verify:**
```bash
docker ps
```
Output:
```
CONTAINER ID   IMAGE   STATUS
9c972f643b55   nginx   Up 11 seconds
```

### Custom Container Name

```bash
docker run -d --name webserver nginx
```

**Output:**
```
e5f918bdb69ce2986f60222818a3080d1b1efeae6060fae62f52c970b9e2c443
```

**Verify:**
```bash
docker ps
```
Output (NAMES column):
```
webserver
mystifying_chandrasekhar
```

![Docker detached mode and custom name](day29-10-docker-detached-custom-name-port-mapping.png)

### Port Mapping

```bash
docker run -d -p 8081:80 --name nginx-port-test nginx
```

**Output:**
```
e44682cd590804385f9e39753f2cfb51d1f9847c20d4cacf13d1b841ba273590
```

**Verify:**
```bash
docker ps
```
Output (PORTS column):
```
0.0.0.0:8081->80/tcp
```

![Docker port mapping](day29-11-docker-port-mapping.png)

### Check Container Logs

```bash
docker logs nginx-port-test
```

**Output:**
```
Configuration complete; ready for start up
nginx/1.29.8
start worker processes
```

Logs confirmed that Nginx started successfully.

![Docker container logs](day29-12-docker-container-logs.png)

### Execute Command Inside Running Container

```bash
docker exec -it nginx-port-test bash
```

**Inside container:**
```bash
ls
hostname
```

**Exit:**
```bash
exit
```

![Docker exec into container](day29-13-docker-exec-container.png)

---

## Docker Commands Learned

| Command | Purpose |
|---------|---------|
| `docker run` | Create and run container |
| `docker ps` | List running containers |
| `docker ps -a` | List all containers |
| `docker stop` | Stop container |
| `docker rm` | Remove container |
| `docker logs` | View container logs |
| `docker exec` | Execute command inside container |
| `docker images` | List Docker images |

---


## Key Learnings

- Learned Docker fundamentals.
- Understood containers and their benefits.
- Compared containers with Virtual Machines.
- Learned Docker architecture.
- Installed and verified Docker.
- Ran hello-world, Nginx, and Ubuntu containers.
- Practiced container lifecycle management.
- Learned detached mode, port mapping, logs, and `docker exec`.

---

## Conclusion

Today I learned the basics of Docker and containerization.

I successfully installed Docker, pulled images from Docker Hub, created containers, managed container lifecycle, exposed applications using port mapping, and explored running containers.

Docker is an important foundation for DevOps, CI/CD pipelines, Kubernetes, and modern application deployment.
