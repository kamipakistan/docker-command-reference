## Introduction

This document provides a definitive, single-source reference for the most important Docker CLI commands. It is tailored for software engineers, platform developers, and operations professionals who need to manage containerized workloads efficiently—whether onboarding to Docker, preparing for certification, or simply requiring a reliable everyday companion.

Each command is presented with a concise, plain-English explanation that clarifies not only what the command does, but also *when* and *why* you would use it. The reference is deliberately organized into six focused sections, mapping directly to the day-to-day concerns of container lifecycle management:

- **Images** – building, listing, and removing the immutable templates that define your application environments
- **Containers** – creating, running, inspecting, and gracefully shutting down running instances
- **Troubleshooting** – debugging runtime issues by accessing logs, executing interactive shells, and inspecting container metadata
- **Docker Hub** – interacting with the public registry: authentication, searching, pulling, and publishing images
- **Volumes** – designing stateful applications by mounting persistent, reusable storage independent of container lifecycles
- **Networks** – connecting containers securely using user-defined bridge networks and service discovery via container names

All examples are copy-paste ready, with flags and arguments clearly annotated. You can incorporate them directly into your workflow without translating from abstract documentation, which makes this reference ideal for fast-paced development, CI/CD integration, and on-call troubleshooting.

---

## 📑 Table of Contents
1. [Images](#-images)
2. [Containers](#-containers)
3. [Troubleshooting](#-troubleshooting)
4. [Docker Hub](#-docker-hub)
5. [Volumes](#-volumes)
6. [Networks](#-networks)

---

## 🖼️ Images
*Images are read-only templates used to create containers. Think of them as the "blueprint" of your application.*

### 📋 List all local images
```bash
docker images
```
**Explanation:** Displays a list of all the Docker images you have downloaded or built on your local machine. It shows the repository name, tag, image ID, when it was created, and its size.

### 🗑️ Delete an image
```bash
docker rmi <image_name>
```
**Explanation:** Removes a specific image from your local storage. *(Note: You cannot delete an image if it is currently being used by a running container. Stop and remove the container first!)*

### 🧹 Remove unused images
```bash
docker image prune
```
**Explanation:** Acts like a cleanup crew. It deletes all "dangling" images (images that have no name/tag and are not associated with any container) to free up disk space.

### 🛠️ Build an image from a Dockerfile
```bash
docker build -t <image_name>:<version> .
```
**Explanation:** Reads the instructions in your `Dockerfile` and creates a new image. 
* The `-t` flag tags your image with a name and an optional version (like `my-app:1.0`).
* The `.` at the end tells Docker to look for the Dockerfile in the current directory.

```bash
docker build -t <image_name>:<version> . --no-cache
```
**Explanation:** Builds the image completely from scratch, ignoring any previously cached layers. This is highly useful when you want to ensure you are pulling the freshest updates of packages during the build process.

---

## 📦 Containers
*Containers are the actual running instances of your images. If the image is the blueprint, the container is the living house.*

### 📋 List all local containers
```bash
docker ps -a
```
**Explanation:** Shows *every* container on your system—whether it is currently running or has been stopped. 

```bash
docker ps
```
**Explanation:** Shows only the containers that are actively running right now.

### 🚀 Create & run a new container
```bash
docker run <image_name>
```
**Explanation:** Spins up a new container from the specified image. If the image isn't already on your computer, Docker will automatically go to Docker Hub, download it, and then run it.

### 🌙 Run container in the background
```bash
docker run -d <image_name>
```
**Explanation:** The `-d` flag stands for "detached". It runs the container in the background, freeing up your terminal window so you can run other commands while the container operates silently.

### 🏷️ Run container with a custom name
```bash
docker run --name <container_name> <image_name>
```
**Explanation:** By default, Docker gives containers random, witty names (like `sleepy_einstein`). This command lets you give it a specific, memorable name so it's easier to manage later.

### 🌐 Port Binding
```bash
docker run -p <host_port>:<container_port> <image_name>
```
**Explanation:** Creates a bridge between your computer (the host) and the container. 
*Example:* `-p 8080:80` means "any traffic hitting my computer's port 8080 should be forwarded directly to the container's port 80."

### ⚙️ Set environment variables
```bash
docker run -e <var_name>=<var_value> <image_name>
```
**Explanation:** The `-e` flag passes dynamic configuration into the container without changing the code. 
*Example:* `-e MYSQL_ROOT_PASSWORD=mysecretpassword` sets the database password at runtime.

### ⏯️ Start or Stop an existing container
```bash
docker start <container_name_or_id>
docker stop <container_name_or_id>
```
**Explanation:** Safely boots up or shuts down a container that has already been created. `stop` sends a graceful shutdown signal, giving the app time to save data before closing.

### 🔍 Inspect a running container
```bash
docker inspect <container_name_or_id>
```
**Explanation:** Unlocks the hood of your container. It outputs a massive, detailed JSON file containing its IP address, mounted volumes, environment variables, and network settings.

### 🗑️ Delete a container
```bash
docker rm <container_name_or_id>
```
**Explanation:** Permanently deletes a stopped container from your system to free up resources. *(Remember: The container must be stopped before you can remove it).*

---

## 🛠️ Troubleshooting
*When things go wrong, these commands help you look under the hood to find out why.*

### 📜 Fetch logs of a container
```bash
docker logs <container_name_or_id>
```
**Explanation:** Prints out the standard output and error streams of a container. If your app crashed on startup, this is the first place you should look to read the error messages.

### 💻 Open a shell inside a running container
```bash
docker exec -it <container_name> /bin/bash
```
**Explanation:** Drops you directly inside the container's terminal as if you had SSH'd into a remote server. 
* `-i` keeps the connection open.
* `-t` gives you a proper terminal interface.
* `/bin/bash` opens the Bash shell.

```bash
docker exec -it <container_name> sh
```
**Explanation:** Does the exact same thing as above, but uses `sh` (Shell) instead of Bash. This is necessary for lightweight images (like Alpine Linux) that don't have Bash installed by default.

---

## ☁️ Docker Hub
*Docker Hub is the cloud-based registry where you can find and share Docker images with the world.*

### ⬇️ Pull an image
```bash
docker pull <image_name>
```
**Explanation:** Downloads an image from Docker Hub to your local machine without running it. Great for pre-downloading large images so they are ready to use instantly later.

### ⬆️ Publish an image
```bash
docker push <username>/<image_name>
```
**Explanation:** Uploads your locally built image to your Docker Hub account so others can download and use it. *(You must tag your image with your username first).*

### 🔐 Login / Logout
```bash
docker login -u <username>
# OR simply:
docker login
```
**Explanation:** Authenticates your terminal with Docker Hub. It will securely prompt you for your password. Use `docker logout` when you are done to remove your credentials.

### 🔎 Search for an image
```bash
docker search <image_name>
```
**Explanation:** Queries Docker Hub directly from your terminal and returns a list of relevant images, along with a short description and how popular they are (number of stars).

---

## 💾 Volumes
*Containers are ephemeral (temporary)—if you delete a container, its data dies with it. Volumes are folders on your host machine mapped to the container so data survives even after the container is deleted.*

### 📋 List all Volumes
```bash
docker volume ls
```
**Explanation:** Shows all the storage volumes currently existing on your system.

### ➕ Create a Named Volume
```bash
docker volume create <volume_name>
```
**Explanation:** Creates a specific, named storage area managed entirely by Docker. You don't need to know where it lives on your actual hard drive; Docker handles the file path.

### 🗑️ Delete a Named Volume
```bash
docker volume rm <volume_name>
```
**Explanation:** Permanently deletes a specific volume. *(Warning: This will delete any data stored inside that volume!)*

### 🔗 Mount a Named Volume
```bash
docker run --volume <volume_name>:<mount_path> <image_name>

# OR using the newer --mount syntax:
docker run --mount type=volume,src=<volume_name>,dest=<mount_path> <image_name>
```
**Explanation:** Links a named volume to a specific folder inside the container. Any data the app writes to `<mount_path>` inside the container is safely saved in the `<volume_name>` on your host.

### 🕵️ Mount an Anonymous Volume
```bash
docker run --volume <mount_path> <image_name>
```
**Explanation:** Similar to a named volume, but Docker assigns it a random, long string as a name. It's quick to use, but hard to reuse later because you don't know its name.

### 📂 Create a Bind Mount
```bash
docker run --volume <host_path>:<container_path> <image_name>

# OR using the --mount syntax:
docker run --mount type=bind,src=<host_path>,dest=<container_path> <image_name>
```
**Explanation:** Maps a very specific folder on *your actual computer* (like `/Users/john/Desktop/code`) directly into the container. This is heavily used by developers because any changes you make to the files on your computer instantly reflect inside the container.

### 🧹 Remove unused local volumes
```bash
docker volume prune
```
**Explanation:** Deletes all volumes that are not currently attached to any container. *(Highly effective at cleaning up anonymous volumes left behind by deleted containers).*

---

## 🌐 Network
*By default, containers are isolated. Networks allow containers to talk to each other securely using their container names instead of complex IP addresses.*

### 📋 List all networks
```bash
docker network ls
```
**Explanation:** Displays all the networks Docker has created. You will usually see three defaults: `bridge`, `host`, and `none`.

### ➕ Create a network
```bash
docker network create <network_name>
```
**Explanation:** Creates a custom, isolated network environment. If you put multiple containers on this network, they can instantly communicate with each other.

### 🗑️ Remove a network
```bash
docker network rm <network_name>
```
**Explanation:** Deletes a specific custom network. *(Note: You cannot delete a network if there are still containers actively connected to it).*

### 🧹 Remove all unused networks
```bash
docker network prune
```
**Explanation:** The ultimate network cleaner. It scans your system and deletes any custom networks that aren't currently being used by any containers.

---
*Happy Dockering! 🚀*
```
