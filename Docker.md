# 📌 Important Commands

To check which process/user owns a specific port, run:
```bash
sudo lsof -i :<port_number>
```

---

# 🐳 Docker Overview

Docker is a tool that allows us to run services in an isolated container format. These containers can be easily shared with anyone.

### 📥 Installation & Service Management

1. **Install Docker** via CLI:
   ```bash
   sudo apt update && sudo apt install -y docker.io
   ```

2. **Manage the Docker Service**:
   ```bash
   sudo systemctl status docker
   sudo systemctl start docker
   sudo systemctl stop docker
   sudo systemctl reload docker
   ```

### 👤 Running Docker Without `sudo`

By default, Docker requires root privileges, so you have to prepend `sudo` to every command. To avoid using `sudo` every time, you can add your current user to the `docker` group:

1. Add the current user to the docker group:
   ```bash
   sudo usermod -aG docker $USER
   ```
2. Refresh group membership without logging out:
   ```bash
   newgrp docker
   ```

---

# 📦 Docker Images

A **Docker Image** is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, system libraries, and settings.

> [!TIP]
> Think of a Docker Image as a **read-only blueprint** or a "screenshot" of a pre-configured environment. When you run an image, Docker creates a live, isolated **Container** from it. The container is the actual running instance, and the image is the static template it is made from.

- **Immutability**: Images are immutable (cannot be changed once built).
- **Registries**: Images are stored in registries like [Docker Hub](https://hub.docker.com/).

### Basic Commands
* **Pull an image**:
  ```bash
  docker pull <image_name>
  # Example:
  docker pull nginx
  ```
* **Run a container from an image**:
  ```bash
  docker run <image_name>
  # Example (run detached):
  docker run -d nginx
  ```
* **Login to Docker Hub**:
  Create an account on [hub.docker.com](https://hub.docker.com/) and sign in via terminal:
  ```bash
  docker login
  ```

---

## 🛠️ Dockerfile Structure & Build

A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image.

### General Structure:
```dockerfile
# 1. Base image (e.g., node, ubuntu, python)
FROM node:18

# 2. Working directory inside the container
WORKDIR /app

# 3. Copy application files (source, destination)
COPY . .

# 4. Install dependencies (runs during build phase)
RUN npm install
# RUN npm run test

# 5. Document the port the container listens on
EXPOSE 3000

# 6. Command to execute when the container starts (runtime)
CMD ["node", "server.js"]
```

### Building an Image
To build a Docker image from a `Dockerfile`, run:
```bash
docker build -t <image_name>:<tag> .
```

* **`docker build`**: Instructs Docker to build an image.
* **`-t`**: Stands for "tag" (assigns a name and optionally a version tag to the image).
* **`.`**: Specifies the build context (tells Docker to look in the current directory for the `Dockerfile`).

---

## 🔌 Port Mapping & Detached Mode

Since a Docker container is isolated, we need to map its internal port to our host machine's port to access it.

### Exposing Ports
```bash
docker run -p <host_port>:<container_port> <image_name>
```

### Running in the Background (Detached Mode)
To run a container in the background so it doesn't block your terminal, use the `-d` (detach) flag:
```bash
docker run -d -p <host_port>:<container_port> <image_name>
```

### Running Multiple Containers
You can run multiple containers at the same time, but they must be exposed on different host ports:
```bash
docker run -d -p 8080:80 nginx
docker run -d -p 8081:80 nginx
```

---

## 💻 Essential Docker Commands

| Command | Description |
| :--- | :--- |
| `docker ps` | List all running containers |
| `docker ps -a` | List all containers (running and stopped) |
| `docker image ls` / `docker images` | List all downloaded/built images |
| `docker stop <container_id>` | Stop a running container |
| `docker start <container_id>` | Start a stopped container |
| `docker rm <container_id>` | Remove a container |
| `docker rmi <image_name_or_id>` | Remove a Docker image |
| `docker tag <old_name> <new_name>` | Rename / retag an image |
| `docker image inspect <image_name>` | Show detailed metadata about an image |

### Advanced Running Flags
* **Auto-remove container on stop (`--rm`)**:
  ```bash
  docker run -d --rm -p <host_port>:<container_port> <image_name>
  ```
* **Run with a custom container name (`--name`)**:
  ```bash
  docker run -d --name <custom_name> -p <host_port>:<container_port> <image_name>
  ```

---

## 🌍 Predefined Images & Interactive Mode

### Predefined Images
You can access pre-built official images of operating systems or runtimes (like Node.js, Python, Ubuntu) on Docker Hub.
* If you run a container using an image not yet present on your machine, Docker automatically pulls it from Docker Hub first:
  ```bash
  docker run python:3.9
  ```

### Interactive Mode
If you need to interact with the container (e.g., run a shell or type input), run it with `-it` (interactive + TTY flags):
```bash
docker run -it ubuntu /bin/bash
```

---

# 💾 Docker Volumes (Data Persistence)

A **Docker Volume** is a mechanism used to store data outside the container's writable layer, allowing data to persist even if the container is stopped, deleted, or recreated.

> [!NOTE]
> Docker stores volumes on your host machine (usually in `/var/lib/docker/volumes/`) and maps them into the container's filesystem. This allows multiple containers to share the same persistent data.

### Volume Management Commands

* **Create a volume**:
  ```bash
  docker volume create <volume_name>
  ```
* **List all volumes**:
  ```bash
  docker volume ls
  ```
* **Inspect a volume** (view host mount path and details):
  ```bash
  docker volume inspect <volume_name>
  ```
* **Mount a volume to a container (`-v`)**:
  ```bash
  docker run -it --rm -v <volume_name>:<container_path> <image_name>
  # Example:
  docker run -it --rm -v myvol:/app mynodeapp
  ```
* **Remove a specific volume**:
  ```bash
  docker volume rm <volume_name>
  ```
* **Remove all unused volumes** (free up space):
  ```bash
  docker volume prune
  ```

---

## 🔗 Bind Mounts

A **Bind Mount** mounts a file or directory from your host machine directly into the container. Any changes to the host file/directory are immediately reflected in the container (and vice versa). This is highly useful for local development.

### How to use Bind Mounts:
```bash
docker run -it -v <absolute_host_path>:<container_path> <image_name>
```

#### Example:
If you have a file `server.txt` containing critical info and want it to update in real-time inside the container:
```bash
docker run -it -v /home/ankit/practice/server.txt:/app/server.txt mynodeapp
```

---

## 🚫 `.dockerignore` File

Similar to `.gitignore`, a `.dockerignore` file specifies files or directories (like `node_modules` or local logs) that should be excluded when copying files into the Docker image. This helps keep image sizes small and build times fast.

**Example `.dockerignore`:**
```text
node_modules
npm-debug.log
.git
Dockerfile
.dockerignore
```

---

# 🌐 Multi-Container Networking

By default, Docker containers are isolated. To allow containers to communicate with each other, we can connect them to a shared **Docker Network**. When connected to the same network, containers can communicate using their **container names** instead of IP addresses.

### Creating and Using a Network

1. **Create a custom network**:
   ```bash
   docker network create <network_name>
   ```
2. **Run container 1 on the network**:
   ```bash
   docker run -d --name container1 --network <network_name> <image_name>
   ```
3. **Run container 2 on the network**:
   ```bash
   docker run -d --name container2 --network <network_name> <image_name>
   ```

Now, `container1` can resolve and communicate with `container2` simply by using the hostname `container2`.

---

## 🔑 Environment Variables

To pass environment variables to a container's runtime process, use the `-e` or `--env` flag:

```bash
docker run -d -e <variable_name>=<value> <image_name>
```

#### Example:
```bash
docker run -e USER="ankit" -e PASS="235" myapp
```

---

# 🎼 Docker Compose

**Docker Compose** is a tool for defining and running multi-container Docker applications. It uses a configuration file (`docker-compose.yaml`) to manage services, networks, and volumes for your entire architecture.

### General Structure (`docker-compose.yaml`):

```yaml
version: '3.8'

services:
  web-app:
    image: node:18
    ports:
      - "3000:3000"
    networks:
      - app-network
    environment:
      - NODE_ENV=production

  database:
    image: postgres:15
    environment:
      - POSTGRES_USER=ankit
      - POSTGRES_PASSWORD=securepass
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  db-data:
```

### Docker Compose Commands

* **Build the services** (if custom Dockerfiles are specified):
  ```bash
  docker compose build
  ```
* **Start all containers** in the background (detached):
  ```bash
  docker compose up -d
  ```
* **Stop and remove all containers, networks, and volumes** defined in the file:
  ```bash
  docker compose down
  ```

---

# 🔍 Docker Scout

**Docker Scout** is a vulnerability scanning tool for Docker images that analyzes dependencies and highlights security concerns.

> [!NOTE]
> Docker Scout comes built-in with Docker Desktop, but is not included in the standard `docker.io` package.

### Installation & Setup

1. **Install via script**:
   ```bash
   curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s --
   ```
2. **Verify installation**:
   ```bash
   docker scout version
   ```
3. **Login to Docker Hub** (required for scanning):
   ```bash
   docker login -u <username>
   ```

### Scanning Images

* **Quick vulnerability overview**:
  ```bash
  docker scout quickview <image_name>
  ```
* **Detailed list of CVEs** (Common Vulnerabilities and Exposures):
  ```bash
  docker scout cves <image_name>
  ```

---

# ✨ Docker Init (Personal Favorite)

Writing a high-quality `Dockerfile` and `docker-compose.yaml` from scratch can be tedious. You can automate this process using:

```bash
docker init
```

Running `docker init` in your terminal launches an interactive walkthrough. It will ask you about your project's language, framework, port, and standard setup, then automatically generate:
- `Dockerfile`
- `docker-compose.yaml`
- `.dockerignore`
