# Docker & Docker Compose v2 Guide for Linux

A comprehensive, production-oriented reference for **Docker Engine** and **Docker Compose v2** on Linux. This guide covers core containerization concepts, step-by-step installation instructions, everyday CLI commands, Dockerfile best practices, Compose orchestration, and troubleshooting.

---

## 📑 Table of Contents

1. [Understanding Docker & Containerization](#-1-understanding-docker--containerization)
2. [Installation Guide for Linux](#-2-installation-guide-for-linux)
   - [Ubuntu & Debian Installation](#step-by-step-ubuntu--debian)
   - [RHEL, CentOS & Fedora Installation](#step-by-step-rhel-centos-rocky-linux--fedora)
   - [Linux Post-Installation Steps](#post-installation-steps-non-root-user--systemd)
   - [Verification](#verify-installation)
3. [Core Docker Architecture](#️-3-core-docker-architecture)
4. [Essential Docker CLI Commands](#️-4-essential-docker-cli-commands)
   - [System & Disk Management](#system--disk-management)
   - [Image Management](#image-management)
   - [Container Lifecycle & Management](#container-lifecycle--management)
   - [Inspection, Logs & Debugging](#inspection-logs--debugging)
   - [Volumes & Storage](#volumes--storage-persistence)
   - [Networks](#networking)
5. [Building Images with Dockerfile](#️-5-building-images-with-dockerfile)
   - [Key Directives](#key-dockerfile-directives)
   - [CMD vs ENTRYPOINT](#cmd-vs-entrypoint)
   - [Multi-Stage Build Example](#multi-stage-build-production-example)
   - [.dockerignore File](#the-dockerignore-file)
6. [Docker Compose v2](#-6-docker-compose-v2)
   - [Compose V1 vs Compose V2](#compose-v1-docker-compose-vs-compose-v2-docker-compose)
   - [compose.yaml Structure](#anatomy-of-a-composeyaml-file)
   - [Full Multi-Container Example](#full-multi-container-stack-example)
   - [Compose CLI Commands](#docker-compose-v2-command-reference)
   - [Environment Variables (.env)](#environment-variables-and-env-file)
7. [Production Best Practices & Security](#-7-production-best-practices--security)
8. [Troubleshooting Common Issues](#-8-troubleshooting-common-issues)

---

## 🐳 1. Understanding Docker & Containerization

### What is a Container?

A **container** is a lightweight, standalone, executable package that includes everything needed to run a piece of software: code, runtime, system tools, system libraries, and settings.

Containers isolate applications from their environment and ensure that software works uniformly regardless of differences between development, staging, and production environments.

### Containers vs. Virtual Machines (VMs)

| Feature | Virtual Machines (VMs) | Docker Containers |
| :--- | :--- | :--- |
| **Architecture** | Hypervisor runs full guest operating systems on top of hardware. | Containers share the host Linux kernel via namespaces and cgroups. |
| **Size** | Gigabytes (includes full OS kernel, utilities, drivers). | Megabytes to few hundred megabytes (only runtime & dependencies). |
| **Startup Time** | Minutes (boots a complete operating system). | Milliseconds to seconds (starts a standard Linux process). |
| **Performance** | Significant virtualization overhead. | Near bare-metal CPU, memory, and I/O performance. |
| **Portability** | Heavy VM images, hypervisor-dependent. | Highly portable OCI-compliant container images. |

---

## 💻 2. Installation Guide for Linux

Docker provides two primary distribution channels on Linux:

1. **Docker Engine (`docker-ce`)**: The official, production-grade engine directly maintained by Docker Inc.
2. **Distribution packages (`docker.io`)**: Maintained by distro package managers (often outdated; **not recommended**).

[!IMPORTANT]
Always install from the **official Docker repository** to get the latest security patches, Buildx, and **Docker Compose v2** (as a native CLI plugin).

### Step-by-Step: Ubuntu & Debian

#### 1. Uninstall any old or conflicting packages

```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do
  sudo apt-get remove -y $pkg
done
```

#### 2. Set up the Docker `apt` repository

```bash
# Update package index and install required utilities
sudo apt update
sudo apt install -y ca-certificates curl gnupg

# Create directory for GPG keyrings with correct permissions
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's official GPG key
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "${DEBIAN_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Note for Ubuntu users:
# Replace "https://download.docker.com/linux/debian" with "https://download.docker.com/linux/ubuntu"
```

#### 3. Install Docker Engine and Docker Compose v2

```bash
sudo apt update
sudo apt install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

---

### Step-by-Step: RHEL, CentOS, Rocky Linux & Fedora

#### 1. Uninstall older versions

```bash
sudo dnf remove -y docker \
  docker-client \
  docker-client-latest \
  docker-common \
  docker-latest \
  docker-latest-logrotate \
  docker-logrotate \
  docker-engine \
  podman \
  runc
```

#### 2. Set up repository

```bash
# Install dnf plugins
sudo dnf -y install dnf-plugins-core

# Add Docker CE repository (CentOS / Rocky Linux / RHEL)
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# (For Fedora, use: https://download.docker.com/linux/fedora/docker-ce.repo)
```

#### 3. Install Docker packages

```bash
sudo dnf install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-buildx-plugin \
  docker-compose-plugin
```

---

### Post-Installation Steps: Non-Root User & Systemd

By default, the Docker daemon binds to a Unix socket owned by `root`. To run Docker commands without prepending `sudo`:

#### 1. Create the `docker` group (if it doesn't already exist) and add your user

```bash
sudo groupadd docker 2>/dev/null || true
sudo usermod -aG docker $USER
```

#### 2. Apply group changes

```bash
# Activate changes in current terminal session:
newgrp docker

# Or log out and log back in to your Linux session.
```

#### 3. Enable and start Docker services on boot

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
sudo systemctl start docker.service
```

[!NOTE]
**WSL 2 Users**: In Windows Subsystem for Linux (WSL 2), ensure systemd is enabled in `/etc/wsl.conf`:

```ini
[boot]
systemd=true
```

Then restart WSL from Windows PowerShell via `wsl --shutdown`.

---

### Verify Installation

Run the verification commands:

```bash
# 1. Check Docker Engine version
docker --version
# Output example: Docker version 27.x.x, build ...

# 2. Check Docker Compose v2 version
docker compose version
# Output example: Docker Compose version v2.29.x

# 3. Run the official test container
docker run --rm hello-world
```

If the test container displays:
`"Hello from Docker! This message shows that your installation appears to be working correctly."`, your installation is complete and healthy.

---

## 🏛️ 3. Core Docker Architecture

```text
+-------------------------------------------------------------+
|                        DOCKER CLIENT                        |
|   docker build | docker pull | docker run | docker compose  |
+------------------------------+------------------------------+
                               | (Unix Socket / REST API)
                               v
+-------------------------------------------------------------+
|                    DOCKER DAEMON (dockerd)                  |
|  - Manages Images, Containers, Networks, Volumes            |
|  - Interfaces with Container Runtime                        |
+------------------------------+------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                     CONTAINERD & RUNC                       |
|  - containerd: High-level container lifecycle manager       |
|  - runc: OCI low-level runtime (Linux cgroups & namespaces) |
+-------------------------------------------------------------+
                               |
                               v
+-------------------------------------------------------------+
|                      LINUX HOST KERNEL                      |
|  - Namespaces (PID, NET, IPC, MNT, UTS, USER isolation)     |
|  - Control Groups / cgroups (CPU, Memory, I/O limits)       |
|  - OverlayFS (Union filesystem layered storage)             |
+-------------------------------------------------------------+
```

### Key Concepts

* **Image**: An immutable, read-only template built from a series of ordered layers representing filesystem changes.
* **Container**: A lightweight, runnable instance of an image with a thin, read-write layer added on top.
* **Registry**: A hosted repository service for sharing images (Docker Hub, GitHub Container Registry `ghcr.io`, AWS ECR).
* **Volume**: Persistent storage directory hosted on the host machine filesystem (`/var/lib/docker/volumes/`) completely independent of the container lifecycle.
* **Network**: Software-defined virtual bridge or overlay facilitating private communication between containers and port forwarding to the host.

---

## 🛠️ 4. Essential Docker CLI Commands

### System & Disk Management

Keep track of Docker daemon health, information, and disk consumption.

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker version` | Displays client and server versions. | `docker version` |
| `docker info` | Displays comprehensive system-wide information (storage driver, active containers, runtime). | `docker info` |
| `docker system df` | Shows disk space used by images, containers, local volumes, and build cache. | `docker system df` |
| `docker system df -v` | Detailed, itemized disk space breakdown. | `docker system df -v` |
| `docker system prune` | Removes unused data (stopped containers, dangling networks, dangling images). | `docker system prune -f` |
| `docker system prune -a --volumes` | **Deep clean**: Removes ALL unused images, stopped containers, and unused volumes. | `docker system prune -a --volumes -f` |

---

### Image Management

Images are the foundational building blocks of containers.

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker pull` | Downloads an image from a registry. | `docker pull python:3.11-slim` |
| `docker images` / `docker image ls` | Lists all local images. | `docker images` |
| `docker build` | Builds an image from a Dockerfile in the current context. | `docker build -t myapp:1.0 .` |
| `docker tag` | Creates a new tag referencing an existing image. | `docker tag myapp:1.0 registry.example.com/myapp:latest` |
| `docker push` | Pushes an image to a remote registry. | `docker push username/myapp:1.0` |
| `docker rmi` / `docker image rm` | Deletes one or more local images. | `docker rmi python:3.11-slim` |
| `docker image prune` | Removes untagged/dangling images (`<none>`). | `docker image prune` |
| `docker image prune -a` | Removes all images not referenced by any container. | `docker image prune -a` |
| `docker history` | Shows the layer history and commands used to build an image. | `docker history nginx:alpine` |
| `docker save` | Saves an image to a tar archive file. | `docker save -o myapp.tar myapp:1.0` |
| `docker load` | Loads an image from a tar archive. | `docker load -i myapp.tar` |

---

### Container Lifecycle & Management

Containers run your applications in isolated user space.

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker run` | Creates and starts a container in one step. | `docker run -d -p 8080:80 --name web nginx` |
| `docker ps` | Lists currently running containers. | `docker ps` |
| `docker ps -a` | Lists all containers (both running and stopped). | `docker ps -a` |
| `docker start` | Starts one or more stopped containers. | `docker start web` |
| `docker stop` | Gracefully stops a running container (sends `SIGTERM`, then `SIGKILL` after grace period). | `docker stop web` (or `docker stop -t 30 web`) |
| `docker restart` | Restarts a container. | `docker restart web` |
| `docker pause` / `unpause` | Pauses/resumes all processes in a container using cgroups freezer. | `docker pause web` |
| `docker kill` | Forcefully stops a container immediately (sends `SIGKILL`). | `docker kill web` |
| `docker rm` | Removes one or more stopped containers. | `docker rm web` |
| `docker rm -f` | Forcefully terminates and deletes a running container. | `docker rm -f web` |
| `docker container prune` | Removes all stopped containers. | `docker container prune -f` |

#### Common `docker run` Flags Explained

```bash
docker run \
  -d \                               # Detached mode: run container in the background
  --name my-web-app \                # Assign a friendly container name
  -p 8080:80 \                       # Port binding: host_port:container_port
  -v app_data:/var/lib/data \        # Named volume mount
  -v $(pwd)/config:/app/config:ro \  # Bind mount local directory as read-only
  -e ENV_VAR="production" \          # Set environment variable
  --env-file .env \                  # Load environment variables from a file
  --restart unless-stopped \         # Restart policy (always, unless-stopped, on-failure)
  --network app-net \                # Attach container to a custom bridge network
  --memory="512m" \                  # Hard limit memory allocation
  --cpus="1.5" \                     # Limit container to 1.5 CPU cores
  --rm \                             # Automatically remove container when it exits (interactive tests)
  python:3.11-slim
```

---

### Inspection, Logs & Debugging

Diagnose issues, monitor performance, and inspect container state.

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker logs` | Fetches container stdout and stderr. | `docker logs web` |
| `docker logs -f --tail 100` | Follows logs in real time starting from the last 100 lines. | `docker logs -f --tail 100 web` |
| `docker exec -it` | Executes a command inside a running container interactively. | `docker exec -it web /bin/bash` |
| `docker inspect` | Returns low-level JSON details (IP, volumes, state, config). | `docker inspect web` |
| `docker stats` | Live streaming stats of CPU, memory, network, and disk I/O. | `docker stats` |
| `docker top` | Displays the running processes of a container. | `docker top web` |
| `docker port` | Lists port mappings for a container. | `docker port web` |
| `docker diff` | Shows files changed or added on the container's writable layer. | `docker diff web` |
| `docker cp` | Copies files/folders between container and local host filesystem. | `docker cp web:/app/log.txt ./log.txt` |

#### Useful Inspection Snippets

```bash
# Extract only the IP address of a container:
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' web

# Inspect container healthcheck status:
docker inspect -f '{{json .State.Health.Status}}' web
```

---

### Volumes & Storage (Persistence)

Containers are ephemeral: data written inside a container is lost when the container is deleted. Docker provides three mechanisms for persistent data:

1. **Volumes**: Managed entirely by Docker inside `/var/lib/docker/volumes/`. Best for production databases and stateful services.
2. **Bind Mounts**: Mounts an arbitrary directory or file from the host filesystem into the container. Ideal for live source code editing in development.
3. **tmpfs Mounts**: Stored in the host's system memory only, never written to disk. Great for sensitive secrets or scratch state.

#### Volume CLI Commands

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker volume create` | Creates a new named volume. | `docker volume create db_data` |
| `docker volume ls` | Lists all volumes. | `docker volume ls` |
| `docker volume inspect` | Inspects volume location and driver. | `docker volume inspect db_data` |
| `docker volume rm` | Deletes an unused volume. | `docker volume rm db_data` |
| `docker volume prune` | Deletes all volumes not used by at least one container. | `docker volume prune -f` |

#### Volume Usage Syntax

```bash
# Method A: Named volume using -v (host-independent)
docker run -d --name db -v db_data:/var/lib/postgresql/data postgres:16-alpine

# Method B: Bind mount using -v (development live reload)
docker run -d --name dev-app -v $(pwd):/app -p 3000:3000 node:20-alpine

# Method C: Recommended modern --mount syntax (explicit and fails early on typo)
docker run -d --name db \
  --mount type=volume,source=db_data,target=/var/lib/postgresql/data \
  postgres:16-alpine
```

---

### Networking

Docker networks enable containers to communicate with each other securely and with the outside world.

#### Core Network Drivers

* **`bridge` (default)**: Private virtual network on a single host. User-defined bridge networks provide automatic **internal DNS resolution** between containers by container name.
* **`host`**: Removes network isolation; container shares host network stack directly (maximum performance, no `-p` needed).
* **`none`**: Completely disables all networking for the container.
* **`overlay`**: Multi-host networking across Swarm or Kubernetes clusters.
* **`macvlan`**: Assigns a real MAC address to a container, making it appear as a physical device on your LAN.

#### Network CLI Commands

| Command | Description | Example |
| :--- | :--- | :--- |
| `docker network create` | Creates a user-defined network. | `docker network create my-bridge` |
| `docker network ls` | Lists all networks. | `docker network ls` |
| `docker network inspect` | Inspects network IP range and connected containers. | `docker network inspect my-bridge` |
| `docker network connect` | Connects a running container to an existing network. | `docker network connect my-bridge web` |
| `docker network disconnect` | Disconnects a container from a network. | `docker network disconnect my-bridge web` |
| `docker network rm` | Removes one or more networks. | `docker network rm my-bridge` |
| `docker network prune` | Removes all unused networks. | `docker network prune -f` |

#### Inter-Container DNS Example

Containers connected to a custom bridge can resolve each other using their container names:

```bash
# 1. Create a custom network
docker network create backend-net

# 2. Run Redis on the network
docker run -d --name redis-server --network backend-net redis:alpine

# 3. Test pinging the Redis container from another container on the same network
docker run --rm --network backend-net alpine ping -c 3 redis-server
```

---

## 🏗️ 5. Building Images with Dockerfile

A `Dockerfile` is a text document containing instructions to assemble a Docker image layer by layer.

### Key Dockerfile Directives

| Instruction | Purpose | Example |
| :--- | :--- | :--- |
| `FROM` | Sets the base image (must be the first valid instruction). | `FROM python:3.11-slim` |
| `WORKDIR` | Sets the working directory for subsequent instructions. Creates it if missing. | `WORKDIR /app` |
| `COPY` | Copies files/folders from the build context into the image filesystem. | `COPY requirements.txt .` |
| `ADD` | Like `COPY`, but extracts local tarballs and can fetch remote URLs (prefer `COPY`). | `ADD archive.tar.gz /opt/` |
| `RUN` | Executes commands during build time and commits the result as a new layer. | `RUN apt-get update && apt-get install -y curl` |
| `ENV` | Sets persistent environment variables available at build time and container runtime. | `ENV PORT=8000 PYTHONUNBUFFERED=1` |
| `ARG` | Defines build-time variables that can be passed via `--build-arg` (not in runtime). | `ARG APP_VERSION=1.0.0` |
| `EXPOSE` | Documents which ports the container listens on (informational only; does not publish). | `EXPOSE 8000` |
| `VOLUME` | Creates a mount point with the specified path for persistent storage. | `VOLUME ["/app/data"]` |
| `USER` | Switches the UID/user for subsequent instructions and container execution. | `USER nonroot` |
| `ENTRYPOINT` | Configures the fixed executable that is invoked when the container starts. | `ENTRYPOINT ["python", "main.py"]` |
| `CMD` | Provides default arguments for `ENTRYPOINT` or default executable if no `ENTRYPOINT`. | `CMD ["--host", "0.0.0.0"]` |

---

### CMD vs. ENTRYPOINT

Understanding how `ENTRYPOINT` and `CMD` interact is crucial:

* **Exec form (Recommended)**: `ENTRYPOINT ["executable", "param1"]` — Does not invoke a shell; passes signals (`SIGTERM`) directly to the process.
* **Shell form**: `ENTRYPOINT executable param1` — Runs via `/bin/sh -c`; prevents graceful shutdown signals from reaching the app unless handled.

#### Combined Pattern

```dockerfile
# Base command that should always run:
ENTRYPOINT ["python", "app.py"]

# Default argument that can easily be overridden on the command line:
CMD ["--port", "8000"]
```

* Running `docker run myimage` executes: `python app.py --port 8000`
* Running `docker run myimage --port 9000` executes: `python app.py --port 9000`

---

### Multi-Stage Build (Production Example)

Multi-stage builds allow you to use compilers, SDKs, and build dependencies in an early stage, then copy **only the final built artifacts** into a clean, minimal runtime stage. This drastically reduces image size and eliminates attack vectors.

#### Example: Production Python FastAPI Application

```dockerfile
# ==========================================
# Stage 1: Build & Dependencies
# ==========================================
FROM python:3.11-slim AS builder

WORKDIR /build

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libpq-dev \
 && rm -rf /var/lib/apt/lists/*

# Copy dependency definition
COPY requirements.txt .

# Install Python wheels into a dedicated prefix
RUN pip install --no-cache-dir --user -r requirements.txt

# ==========================================
# Stage 2: Final Runtime Image
# ==========================================
FROM python:3.11-slim AS runner

WORKDIR /app

# Install only minimal runtime system libraries
RUN apt-get update && apt-get install -y --no-install-recommends \
    libpq5 \
    curl \
 && rm -rf /var/lib/apt/lists/*

# Create a non-root user and group for security
RUN groupadd -r appgroup && useradd -r -g appgroup -u 10001 appuser

# Copy installed Python packages from builder stage
COPY --from=builder /root/.local /home/appuser/.local

# Copy application source code
COPY --chown=appuser:appgroup ./src /app/src

# Configure environment
ENV PATH="/home/appuser/.local/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# Run as non-root user
USER appuser

# Expose port and configure health check
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=5s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8000/health || exit 1

ENTRYPOINT ["uvicorn", "src.main:app"]
CMD ["--host", "0.0.0.0", "--port", "8000"]
```

---

### The `.dockerignore` File

Create a `.dockerignore` file in the same directory as your `Dockerfile` to exclude unnecessary files from being sent to the Docker build context. This speeds up build times and prevents leaking secrets.

```gitignore
# Git version control
.git
.gitignore

# Environment files and credentials (CRITICAL)
.env
.env.*
*.pem
*.key
id_rsa

# Python caches
__pycache__/
*.py[cod]
.venv/
venv/
.pytest_cache/

# Node.js
node_modules/
npm-debug.log

# IDE & OS files
.vscode/
.idea/
.DS_Store
*.swp
```

---

## 🎼 6. Docker Compose v2

**Docker Compose** is a tool for defining and running multi-container Docker applications using a single YAML configuration file.

### Compose V1 (`docker-compose`) vs. Compose V2 (`docker compose`)

* **Compose V1 (Deprecated)**: Written in Python as a separate binary `docker-compose`. Officially deprecated and no longer receives updates.
* **Compose V2 (Current Standard)**: Written in Go, fully integrated into the official Docker CLI as a native plugin:

  ```bash
  # Correct (v2):
  docker compose up -d

  # Obsolete (v1):
  docker-compose up -d
  ```

* Standard configuration file naming: `compose.yaml` (recommended) or `docker-compose.yml`.

---

### Anatomy of a `compose.yaml` File

A Compose file defines:

* **`services`**: Containers to instantiate.
* **`volumes`**: Persistent storage volumes shared across containers.
* **`networks`**: Custom virtual networks connecting services.
* **`secrets` / `configs`**: Sensitive data or configuration files mounted securely.

---

### Full Multi-Container Stack Example

Below is a complete, production-grade example featuring an **API Service**, a **PostgreSQL Database**, a **Redis Cache**, and persistent volumes with healthchecks.

File: `compose.yaml`

```yaml
services:
  # ------------------------------------------------
  # 1. PostgreSQL Database Service
  # ------------------------------------------------
  postgres:
    image: postgres:16-alpine
    container_name: production_postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${DB_NAME:-app_db}
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-securepassword123}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - internal-net
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres} -d ${DB_NAME:-app_db}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s

  # ------------------------------------------------
  # 2. Redis Cache Service
  # ------------------------------------------------
  redis:
    image: redis:7-alpine
    container_name: production_redis
    restart: unless-stopped
    command: ["redis-server", "--appendonly", "yes"]
    volumes:
      - redis_data:/data
    networks:
      - internal-net
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # ------------------------------------------------
  # 3. Backend API Service
  # ------------------------------------------------
  api:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: production_api
    restart: unless-stopped
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql://${DB_USER:-postgres}:${DB_PASSWORD:-securepassword123}@postgres:5432/${DB_NAME:-app_db}
      REDIS_URL: redis://redis:6379/0
      APP_ENV: ${APP_ENV:-production}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - internal-net
      - public-net
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 1024M
        reservations:
          memory: 256M

# --------------------------------------------------
# Named Volumes (Managed by Docker)
# --------------------------------------------------
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

# --------------------------------------------------
# Networks
# --------------------------------------------------
networks:
  internal-net:
    driver: bridge
  public-net:
    driver: bridge
```

---

### Docker Compose v2 Command Reference

| Command | Description | Typical Use |
| :--- | :--- | :--- |
| `docker compose up -d` | Builds, creates, and starts all containers in background. | Initial start or applying updates |
| `docker compose up -d --build` | Forces a rebuild of images before launching services. | Deploying new code changes |
| `docker compose down` | Stops and removes containers and internal networks. | Clean shutdown |
| `docker compose down -v` | Stops containers and **destroys named volumes** (resets DB!). | Full wipe / clean test setup |
| `docker compose ps` | Displays status of all services in the current stack. | Health check & port review |
| `docker compose logs -f` | Streams logs from all services in parallel with color prefixes. | Real-time debugging |
| `docker compose logs -f <service>` | Streams logs for a specific service. | `docker compose logs -f api` |
| `docker compose exec <service> <cmd>` | Runs an interactive command inside a running service container. | `docker compose exec api sh` |
| `docker compose run --rm <service> <cmd>` | Starts a one-off container for a service command, then deletes it. | `docker compose run --rm api python manage.py migrate` |
| `docker compose stop` | Stops running services without deleting containers. | Temporary pause |
| `docker compose start` | Starts existing stopped services. | Resume after stop |
| `docker compose restart [service]` | Restarts all or specified services. | `docker compose restart api` |
| `docker compose build` | Builds or rebuilds image services without starting them. | Pre-build checks |
| `docker compose pull` | Pulls the latest base/external images defined in compose. | Image updates |
| `docker compose config` | Validates compose syntax and displays resolved variables. | Debugging compose configuration |
| `docker compose top` | Displays the running processes within stack services. | Process monitoring |

---

### Environment Variables and `.env` File

Compose automatically reads a `.env` file located in the same directory as `compose.yaml`.

#### Example `.env` file

```ini
DB_NAME=production_db
DB_USER=app_admin
DB_PASSWORD=SuperSecretPassword987!
APP_ENV=production
```

#### Interpolation Syntax

* `${VARIABLE}` — Value of `VARIABLE`. Warns if unset.
* `${VARIABLE:-default}` — Uses `default` if `VARIABLE` is unset or empty.
* `${VARIABLE:=default}` — Sets and uses `default` if `VARIABLE` is unset or empty.
* `${VARIABLE:?error_msg}` — Fails with `error_msg` if `VARIABLE` is unset or empty.

---

## 🔒 7. Production Best Practices & Security

### 1. Never Run as Root Inside Containers

Running containers as root poses serious security risks if an application escape vulnerability occurs.

* Create a dedicated user in your `Dockerfile`:

  ```dockerfile
  RUN useradd -u 10001 -m appuser
  USER appuser
  ```

### 2. Configure Docker Daemon Log Rotation

By default, Docker stores container stdout/stderr JSON logs without rotation, which can fill up your server's root disk (`/var/lib/docker/`).

Create or edit `/etc/docker/daemon.json`:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```

Apply the configuration:

```bash
sudo systemctl reload docker
```

### 3. Pin Base Image Tags

Never use `:latest` in production Dockerfiles. Pin specific version tags or digests to ensure reproducible builds:

```dockerfile
# Good:
FROM python:3.11.9-slim-bookworm

# Bad:
FROM python:latest
```

### 4. Limit Container Resources

Prevent a runaway memory leak or high CPU usage from starving the host OS:

```bash
docker run -d --memory="1g" --cpus="2.0" myapp:1.0
```

### 5. Protect the Docker Socket

Never expose `/var/run/docker.sock` over an unencrypted network or mount it into public-facing containers. Anyone with write access to the Docker socket effectively possesses root access to the entire host system.

---

## 🩺 8. Troubleshooting Common Issues

### Issue 1: "Cannot connect to the Docker daemon at unix:///var/run/docker.sock"

* **Cause**: The Docker systemd service is not active.
* **Resolution**:

  ```bash
  sudo systemctl status docker
  sudo systemctl start docker
  ```

---

### Issue 2: "Got permission denied while trying to connect to the Docker daemon socket"

* **Cause**: The current Linux user does not belong to the `docker` group.
* **Resolution**:

  ```bash
  sudo usermod -aG docker $USER
  newgrp docker
  ```

---

### Issue 3: "Bind for 0.0.0.0:80 failed: port is already allocated"

* **Cause**: Another service or container is already listening on the requested host port.
* **Resolution**:

  ```bash
  # Identify the process occupying the port:
  sudo lsof -i :80
  # or
  sudo ss -tulpn | grep :80

  # Stop conflicting process or change the left-side port in docker run / compose:
  # e.g., -p 8080:80 instead of -p 80:80
  ```

---

### Issue 4: "No space left on device"

* **Cause**: Dangling images, stopped containers, build cache, or unpruned volumes have exhausted disk capacity.
* **Resolution**:

  ```bash
  # Check Docker disk usage:
  docker system df

  # Safe cleanup (removes stopped containers, dangling images, build cache):
  docker system prune -f

  # Complete cleanup (removes all unused images, stopped containers, and volumes):
  docker system prune -a --volumes -f
  ```

---

### Issue 5: Container Exits Immediately After Starting (`Exited (0)` or `Exited (1)`)

* **Cause**: A Docker container only stays alive as long as its foreground PID 1 process is running. If your process finishes or runs as a background daemon, the container terminates immediately.
* **Resolution**:

  ```bash
  # Check container exit logs:
  docker logs <container_id_or_name>

  # For debugging, run an interactive shell:
  docker run -it --entrypoint /bin/sh <image_name>
  ```

---

## 📚 9. Quick Cheat Sheet Summary

| Task | Command |
| :--- | :--- |
| **Run container detached with port & volume** | `docker run -d -p 8080:80 -v data:/data --name my-app my-img` |
| **Interactive shell in container** | `docker exec -it my-app /bin/bash` |
| **View live container logs** | `docker logs -f --tail 50 my-app` |
| **Monitor resource usage** | `docker stats` |
| **Build image with tag** | `docker build -t my-img:1.0 .` |
| **Prune all unused resources** | `docker system prune -a --volumes` |
| **Start Compose stack in background** | `docker compose up -d` |
| **Rebuild and start Compose stack** | `docker compose up -d --build` |
| **Stop and remove Compose stack** | `docker compose down` |
| **View Compose services status** | `docker compose ps` |
