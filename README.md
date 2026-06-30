# 🐳 The Complete Docker & Docker Compose Cheat Sheet

A comprehensive, production-ready developer manual for managing image layers, network bridging, data persistence, multi-container orchestration, and containerization workflows.

---

## 📑 Table of Contents
1. [Core Docker Concepts](#-core-docker-concepts)
2. [Image Layers Architecture](#-image-layers-architecture)
3. [Essential Docker CLI Commands](#-essential-docker-cli-commands)
4. [Docker Networking in Practice (MongoDB & Mongo Express Setup)](#-docker-networking-in-practice)
5. [Multi-Container Orchestration with Docker Compose](#-multi-container-orchestration-with-docker-compose)
6. [Containerizing an Application (Dockerization)](#-containerizing-an-application-dockerization)
7. [Data Persistence with Docker Volumes](#-data-persistence-with-docker-volumes)

---

## 💡 Core Docker Concepts

Before executing commands, it is essential to understand the primary building blocks of the Docker ecosystem:

* **Docker Image:** A read-only, immutable blueprint template containing the application code, runtime libraries, environment variables, and configuration files required to run an application.
* **Docker Container:** A runnable, isolated instance of an image. If an image is a class definition, a container is an instantiated object living in memory.
* **Docker Registry:** A centralized hosted service (like Docker Hub) used for storing, sharing, and version-tracking Docker images.
* **Docker Compose:** A tool for defining and executing multi-container Docker applications using a single, unified declarative configuration file (`yaml`).

---

## 🏗️ Image Layers Architecture

Docker images are constructed as a stack of read-only, reusable layers. Each command in a `Dockerfile` adds a distinct layer. When a container is instantiated, Docker mounts a thin, temporary **Writable Container Layer** directly on top of the immutable image stack.



```text
    ┌───────────────────────────────────┐
    │     Writable Container Layer      │  <-- Created when container boots (Ephemeral)
    ├───────────────────────────────────┤
    │              Layer 2              │  <-- e.g., Installed packages / dependencies
    ├───────────────────────────────────┤
    │              Layer 1              │  <-- e.g., Added application source code
    ├───────────────────────────────────┤
    │            Base Layer             │  <-- e.g., Base Operating System (Ubuntu, Alpine)
    └───────────────────────────────────┘

```
## 💻 Essential Docker CLI Commands
### Basic Engine Operations
| Command | Description |
|---|---|
| docker images | Lists all downloaded and locally built images. |
| docker ps | Lists all currently active, running containers. |
| docker ps -a | Lists all containers regardless of state (Running, Stopped, Exited). |
### Lifecycle Management
 * **Fetch an Image from Registry:**
   ```bash
   docker pull IMAGE_NAME:version
   
   ```
 * **Run a Container (Foreground):**
   ```bash
   docker run IMAGE_NAME
   
   ```
 * **Run an Interactive Ubuntu Shell:**
   ```bash
   docker run -it ubuntu
   
   ```
 * **Run in Background (Detached Mode) with Custom Name:**
   ```bash
   docker run --name CONT_NAME -d IMAGE_NAME
   
   ```
 * **State Control:**
   ```bash
   docker stop CONT_NAME_OR_ID
   docker start CONT_NAME_OR_ID
   
   ```
 * **Resource Invalidation (Deletion):**
   ```bash
   docker rm CONT_NAME_OR_ID    # Removes a stopped container
   docker rmi IMAGE_NAME        # Removes a local image
   
   ```
### Port Binding
To expose your containerized workloads outside of the isolated container environment, bridge the Host Operating System port to the internal Container port:
```bash
docker run -p <host_port>:<container_port> IMAGE_NAME

```
*Example:* Routing incoming traffic on local port 8080 to an internal MySQL server listening on 3306:
```bash
docker run -p 8080:3306 mysql

```
### Runtime Troubleshooting & Diagnostics
 * **Inspect Container Output Logs:**
   ```bash
   docker logs CONT_ID
   
   ```
 * **Execute a Live Session inside a Running Container:**
   ```bash
   # For standard Linux bash shells:
   docker exec -it CONT_ID /bin/bash
   
   # For lightweight Alpine distributions:
   docker exec -it CONT_ID /bin/sh
   
   ```
## 🌐 Docker Networking in Practice
By default, containers can talk to each other only if they are attached to the same network. This practical guide builds an isolated network environment featuring a database engine (**MongoDB**) linked directly to a web-based administration frontend (**Mongo Express**).
### 1. Inspect Existing Networks
```bash
docker network ls

```
*Output reference:*
```text
NETWORK ID     NAME      DRIVER    SCOPE
847e35f5e977   bridge    bridge    local
f17249175f12   host      host      local
f334e5ab3c5c   none      null      local

```
### 2. Provision an Isolated Network Bridge
```bash
docker network create mongo-network

```
### 3. Spin up the Database Engine (MongoDB Container)
Run the backend database inside the new network while feeding administrative configuration via environment variables (-e flags):
```bash
docker run -d \
  -p 27017:27017 \
  --name mongo \
  --network mongo-network \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=qwerty \
  mongo

```
### 4. Spin up the Dashboard Frontend (Mongo Express Container)
Attach this container to the same network. Instead of hardcoding changing IP targets, reference the database container directly by its registered network identifier name (mongo):
```bash
docker run -d \
  -p 8081:8081 \
  --name mongo-express \
  --network mongo-network \
  -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
  -e ME_CONFIG_MONGODB_ADMINPASSWORD=qwerty \
  -e ME_CONFIG_MONGODB_URL="mongodb://admin:qwerty@mongo:27017" \
  mongo-express

```
### 5. Verify Inter-Container Cluster Status
```bash
docker ps

```
*Output reference:*
```text
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS                      NAMES
2f2da4d6b120   mongo-express   "/sbin/tini - /doc.."   5 seconds ago   Up 4 seconds   0.0.0.0:8081->8081/tcp     mongo-express
2336e408d926   mongo           "docker-entrypoint.s.."  4 minutes ago   Up 4 minutes   0.0.0.0:27017->27017/tcp   mongo

```
> **🚀 Success Notification:** You can now open your host browser and safely manage databases by visiting: http://localhost:8081
> 
## 🛠️ Multi-Container Orchestration with Docker Compose
Running multiple docker run commands manually becomes tedious. Docker Compose streamlines this by consolidating multi-container systems into a single declarative file (mongodb.yaml).
```yaml
version: "3.8"

services:
  mongo:
    image: mongo
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty

  mongo-express:
    image: mongo-express
    ports:
      - 8081:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: admin
      ME_CONFIG_MONGODB_ADMINPASSWORD: qwerty
      ME_CONFIG_MONGODB_URL: mongodb://admin:qwerty@mongo:27017/

```
### Stack Lifecycle Orchestration Commands
 * **Boot the Complete Service Cluster (Detached):**
   ```bash
   docker compose -f mongodb.yaml up -d
   
   ```
 * **Tear Down Stack Infrastructure and Clear Networks:**
   ```bash
   docker compose -f mongodb.yaml down
   
   ```
## 📦 Containerizing an Application (Dockerization)
Follow these steps to package custom code (like a Node.js web app) into a deployable Docker image template.
### 1. Write the Application Dockerfile
Create a file named Dockerfile in your root project repository:
```dockerfile
# syntax=docker/dockerfile:1
FROM node:24-alpine

WORKDIR /app

# Safely aggregate application codebase files
COPY . .

# Run production-grade configurations
RUN npm install --omit=dev

EXPOSE 3000

CMD ["node", "src/index.js"]

```
### 2. Build the Executable Target Binary Image
```bash
docker build -t testapp:1.0 .

```
### 3. Spin Up the Newly Composed Image Runtime Locally
```bash
docker run testapp:1.0

```
### 4. Interactive Debugging and Workspace Verification
```bash
docker run -it testapp:1.0 /bin/sh

```
## 💾 Data Persistence with Docker Volumes
Containers are naturally stateless. When an instance gets terminated, its runtime state modifications are permanently lost. Volumes isolate and decouple host application mutations safely outside the container lifetime.
[Image diagram showing the difference between Bind Mounts mapping straight to host files and Named Volumes controlled by Docker Engine]
### Types of Storage Mounts
 1. **Named Volumes:** Managed completely by Docker internal filesystems. High security/portable extraction patterns.
 2. **Anonymous Volumes:** Automatically assigned an obfuscated hash tracking identifier without a friendly tag.
 3. **Bind Mounts:** Explicit file or directory mappings made directly between folders on your physical machine and directories inside your container.
```bash
# Bind Mount Syntax Pattern
docker run -it -v /absolute/host_path/data:/container_target_dir/data ubuntu

# Named Volume Mapping Execution
docker run -v volume_identifier:/container_target_dir/data ubuntu

```
### Volumes Directory Map References
 * **Windows:** C:\ProgramData\docker\volumes
 * **Mac / Linux Platforms:** /var/lib/docker/volumes
### Declarative Volumes inside Docker Compose Engine (mongodb.yaml)
```yaml
version: "3.8"
services:
  mongo:
    image: mongo
    ports:
      - 27017:27017
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: qwerty
    volumes:
      - /Users/your_user/data/db:/data/db # Maps host directory to internal database path

```
### Storage Maintenance Utilities
```bash
docker volume ls              # Lists active storage disks
docker volume create VOL_NAME # Manually builds an storage mount disk
docker volume rm VOL_NAME     # Clears specific volume disk structures
docker volume prune           # Purges unattached anonymous volumes safely

```
*Happy Dockering! 🚀*
```
***

```

