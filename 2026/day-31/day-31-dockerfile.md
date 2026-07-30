# 🐳 Day 31 – Dockerfile: Build Your Own Images

## 📌 Objective

Today I learned how to create my own Docker images using Dockerfile.

Before this, I was using ready-made images from Docker Hub.
With Dockerfile, I learned how DevOps engineers package applications with required dependencies and create repeatable environments.

This is an important skill because in real projects:

```
Developer Application Code
          |
          |
     Dockerfile
          |
          |
    Docker Build
          |
          |
    Docker Image
          |
          |
   Docker Container
          |
          |
 Running Application
```

---

# 1. What is a Dockerfile?

A Dockerfile is a text file that contains instructions used by Docker to create an image.

Instead of manually installing software and configuring servers every time, we define everything inside Dockerfile and create a reusable image.

Example:

```Dockerfile
FROM ubuntu

RUN apt update

CMD ["echo","Hello Docker"]
```

Screenshot:

![Dockerfile Content](./Screenshots/day31-01-dockerfile-content.png)

---

# 2. Creating My First Docker Image

## Scenario

A developer provides an application.

As a DevOps engineer, my responsibility is:

* Create Dockerfile
* Install required dependencies
* Build Docker image
* Run and test container

## Step 1: Create Project Folder

```bash
mkdir my-first-image

cd my-first-image
```

---

## Step 2: Create Dockerfile

Dockerfile:

```Dockerfile
FROM ubuntu

RUN apt update && apt install -y curl

CMD ["echo","Hello from my custom image!"]
```

### Explanation

| Instruction | Purpose                               |
| ----------- | ------------------------------------- |
| FROM        | Selects the base image                |
| RUN         | Executes commands during image build  |
| CMD         | Default command when container starts |

---

## Step 3: Build Docker Image

Command:

```bash
docker build -t my-ubuntu:v1 .
```

What happened internally:

```
Dockerfile
    |
    |
Docker Build
    |
    |
Image Layers Created
    |
    |
my-ubuntu:v1 Image
```

Screenshot (build started):

![Docker Build Start](./Screenshots/day31-02-build-start.png)

Screenshot (build success):

![Docker Build Success](./Screenshots/day31-03-build-success.png)

---

## Step 4: Verify Image

Command:

```bash
docker images
```

Output:

![Docker Image List](./Screenshots/day31-04-custom-image-list.png)

---

## Step 5: Run Container

Command:

```bash
docker run my-ubuntu:v1
```

Output:

```
Hello from my custom image!
```

Screenshot:

![Container Output](./Screenshots/day31-05-run-custom-image.png)

### Important Concept

When we run:

```
docker run my-ubuntu:v1
```

Docker:

1. Creates a container from image
2. Executes CMD
3. Container exits after command completion

---

# 3. Understanding Dockerfile Instructions

Created another Dockerfile to understand common instructions.

Dockerfile:

```Dockerfile
FROM ubuntu

RUN apt update

WORKDIR /app

COPY index.html .

EXPOSE 80

CMD ["cat","index.html"]
```

Screenshot (Dockerfile content):

![Task 2 Dockerfile Content](./Screenshots/day31-06-task2-dockerfile-content.png)

Screenshot (build success):

![Task 2 Build Success](./Screenshots/day31-07-task2-build-success.png)

Screenshot (output):

![Task 2 Output](./Screenshots/day31-08-task2.webp)

Screenshot (container status):

![Task 2 Container Output Status](./Screenshots/day31-09-task2-container-output-status.png)

---

## Instruction Explanation

### FROM

Defines the base image.

Example:

```Dockerfile
FROM ubuntu
```

---

### RUN

Runs commands during image creation.

Example:

```Dockerfile
RUN apt update
```

---

### WORKDIR

Sets the working directory inside container.

Example:

```Dockerfile
WORKDIR /app
```

---

### COPY

Copies files from host machine to image.

Example:

```Dockerfile
COPY index.html .
```

---

### EXPOSE

Documents which port the application uses.

Example:

```Dockerfile
EXPOSE 80
```

---

### CMD

Defines the default command.

Example:

```Dockerfile
CMD ["cat","index.html"]
```

---

# 4. CMD vs ENTRYPOINT

## CMD

CMD provides a default command.

Example:

```Dockerfile
FROM ubuntu

CMD ["echo","Hello"]
```

Screenshot (Dockerfile):

![CMD Dockerfile](./Screenshots/day31-11-cmd-dockerfile.png)

Screenshot (build success):

![CMD Build Success](./Screenshots/day31-12-cmd-build-success.png)

Run:

```bash
docker run cmd-demo:v1
```

Output:

```
Hello
```

If we provide another command:

```bash
docker run cmd-demo:v1 ls
```

Docker replaces CMD.

Screenshot (default vs override):

![CMD Default vs Override](./Screenshots/day31-13-cmd-default-vs-override.png)

---

## ENTRYPOINT

ENTRYPOINT defines a fixed executable.

Example:

```Dockerfile
FROM ubuntu

ENTRYPOINT ["echo"]
```

Screenshot (Dockerfile):

![Entrypoint Dockerfile](./Screenshots/day31-15-entrypoint-dockerfile.png)

Screenshot (build success):

![Entrypoint Build Success](./Screenshots/day31-16-entrypoint-build-success.png)

Run:

```bash
docker run entry-demo:v1 Rohit
```

Output:

```
Rohit
```

Here Docker appends the argument.

Screenshot (output):

![Entrypoint Output](./Screenshots/day31-17-entrypoint-output.png)

---

## Difference

| CMD               | ENTRYPOINT             |
| ----------------- | ---------------------- |
| Default command   | Fixed command          |
| Can be overridden | Arguments are appended |
| Flexible          | More strict            |

---

## Real-Time Usage

CMD is useful when:

* Container can run different commands
* Default behavior is enough

ENTRYPOINT is useful when:

* Container should always run a specific application
* Example: Monitoring agent, API service

---

# 5. Dockerizing a Static Website using Nginx

## Real-Time Scenario

Developer provides:

```
index.html
```

DevOps engineer creates a Docker image and deploys it.

Flow:

```
HTML File
    |
    |
Dockerfile
    |
    |
Nginx Image
    |
    |
Container
    |
    |
Website Available
```

---

## Create index.html

Example:

```html
<h1>Hello from Docker!</h1>

<p>Day 31 - Dockerfile Practice</p>
```

---

## Create Nginx Dockerfile

```Dockerfile
FROM nginx:alpine

COPY index.html /usr/share/nginx/html/

EXPOSE 80

CMD ["nginx","-g","daemon off;"]
```

Explanation:

* nginx:alpine → Lightweight web server image
* COPY → Places website file inside nginx directory
* EXPOSE → Application port
* CMD → Starts nginx server

Screenshot:

![Nginx Dockerfile](./Screenshots/day31-19-nginx-dockerfile.png)

---

## Build Website Image

Command:

```bash
docker build -t my-website:v1 .
```

Screenshot:

![Website Build](./Screenshots/day31-19-website-build-success.png)

---

## Run Container

Command:

```bash
docker run -d -p 8080:80 my-website:v1
```

Verify:

```bash
docker ps
```

Screenshot:

![Nginx Container](./Screenshots/day31-20-nginx-container-running.png)

---

## Access Website

Open:

```
http://EC2_PUBLIC_IP:8080
```

Output:

```
Hello from Docker!
Day 31 - Dockerfile Practice
```

Screenshot:

![Website Output](./Screenshots/day31-21-nginx-website-browser.png)

---

# 6. .dockerignore

## Why .dockerignore?

During build, Docker sends files from the current directory as build context.

Unnecessary files increase build time.

Example:

```
node_modules
.git
*.md
.env
```

Benefits:

* Smaller build context
* Faster builds
* Avoid copying sensitive files

Screenshot:

![Dockerignore](./Screenshots/day31-22-dockerignore-build-cache.png)

---

# 7. Docker Build Cache Optimization

## Docker Layers

Every Dockerfile instruction creates a layer.

Example:

```
FROM nginx
      |
COPY files
      |
RUN commands
      |
CMD
```

Docker stores these layers and reuses unchanged layers.

---

## First Build

```bash
docker build -t cache-demo:v1 .
```

Screenshot:

![Cache First Build](./Screenshots/day31-23-cache-first-build.png)

---

## Change Application File

Modified:

```
index.html
```

Build again:

```bash
docker build -t cache-demo:v2 .
```

Docker rebuilt the changed layer.

Screenshot:

![Cache Break After Change](./Screenshots/day31-24-cache-break-after-change.png)

---

## Build Without Changes

```bash
docker build -t cache-demo:v3 .
```

Output:

```
Using cache
```

Docker reused previous layers.

Screenshot:

![Cache Reuse](./Screenshots/day31-25-cache-reuse.png)

---

## Why Layer Order Matters?

Good Dockerfile:

```Dockerfile
FROM ubuntu

RUN apt install dependencies

COPY application-code .
```

Dependencies change less frequently, so Docker can reuse cache.

---

# 8. Interview Questions

## Q1. Difference between docker build and docker run?

Answer:

`docker build` creates an image from Dockerfile.

`docker run` creates and starts a container from an image.

---

## Q2. Why do we use Dockerfile?

Answer:

Dockerfile helps create a repeatable environment by defining application dependencies and configuration.

---

## Q3. CMD vs ENTRYPOINT?

Answer:

CMD can be overridden during runtime.

ENTRYPOINT defines the main executable and accepts additional arguments.

---

## Q4. Why use .dockerignore?

Answer:

To exclude unnecessary files from Docker build context and improve build performance.

---

## Q5. Why does Docker use layers?

Answer:

Layers allow Docker to reuse unchanged parts of an image, making builds faster.

---

# ✅ Key Learnings

Today I learned:

✅ How to create custom Docker images
✅ How Dockerfile instructions work
✅ Difference between CMD and ENTRYPOINT
✅ How to deploy a website using Nginx container
✅ How .dockerignore improves builds
✅ How Docker layer caching optimizes CI/CD workflows

