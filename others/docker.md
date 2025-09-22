
# Docker Core Concepts

## Table of Contents
- [Docker Fundamentals](#docker-fundamentals)
- [Key Components](#key-components)
  - [Dockerfile](#dockerfile)
  - [Image](#image)
  - [Container](#container)
  - [Registry](#registry)
  - [Docker Engine](#docker-engine)
- [Basic Commands](#basic-commands)
- [Container Management](#container-management)
- [Docker Compose](#docker-compose)
- [Networking](#networking)
  - [Bridge Network](#bridge-network)
  - [Host Network](#host-network)
  - [Overlay Network](#overlay-network)
  - [Macvlan Network](#macvlan-network)
- [Storage](#storage)
- [Best Practices](#best-practices)


## Docker Fundamentals


Docker is a platform for developing, shipping, and running applications in containers. Containers are lightweight, standalone, and executable packages that include everything needed to run a piece of software: code, runtime, system tools, libraries, and settings. This ensures that applications run consistently across different computing environments.

**Key Points:**
- **Lightweight:** Containers share the host OS kernel, making them more efficient than virtual machines.
- **Portability:** Run the same container image on any system with Docker installed (laptop, server, cloud, etc.).
- **Isolation:** Each container runs in its own isolated environment, reducing conflicts between applications.
- **Consistency:** Avoids the "it works on my machine" problem by packaging dependencies and configuration with the app.
- **Rapid Deployment:** Containers can be started and stopped quickly, enabling fast scaling and updates.

**Example Use Case:**
Suppose you have a Python web app that requires specific versions of Python and libraries. By containerizing it, you ensure that every developer and server runs the app in the same environment, eliminating version conflicts.

## Key Components

### Dockerfile

A `Dockerfile` is a text file containing step-by-step instructions for building a Docker image. Each instruction creates a new layer in the image.

**Common Dockerfile Instructions:**
- `FROM`: Specifies the base image.
- `WORKDIR`: Sets the working directory inside the container.
- `COPY`/`ADD`: Copies files/directories into the image.
- `RUN`: Executes commands during build (e.g., install packages).
- `CMD`/`ENTRYPOINT`: Sets the default command to run when the container starts.

**Example:**
```Dockerfile
# Use an official Python runtime as a parent image
FROM python:3.10-slim

# Set the working directory
WORKDIR /app

# Copy the current directory contents into the container
COPY . /app

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose port 5000 for the app
EXPOSE 5000

# Run the application
CMD ["python", "app.py"]
```

**Multi-stage Build Example:**
```Dockerfile
FROM node:18 AS builder
WORKDIR /app
COPY . .
RUN npm install && npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```
This approach reduces final image size by copying only the build artifacts.

### Image

A Docker image is a read-only template that contains the application code, libraries, dependencies, and other files needed to run an application. Images are built from Dockerfiles and can be versioned and shared.

**Example:**
You can pull a pre-built image from Docker Hub:
```sh
docker pull redis:7-alpine
```
Or build your own image:
```sh
docker build -t myapp:1.0 .
```

### Container

A container is a runnable instance of an image. It is isolated from the host and other containers, but can interact with them through defined channels (e.g., networks, volumes).

**Example:**
```sh
docker run -d --name webapp -p 8080:80 nginx:alpine
```
This starts a detached Nginx container named `webapp` and maps port 8080 on the host to port 80 in the container.

### Registry

A registry is a repository for storing, distributing, and sharing Docker images. The most popular public registry is Docker Hub, but you can also use private registries (e.g., Amazon ECR, Google Container Registry, Harbor).

**Example:**
```sh
docker push myusername/myapp:latest
docker pull myusername/myapp:latest
```

### Docker Engine

The Docker Engine is the core client-server technology that builds and runs containers. It consists of:
- **Docker Daemon (`dockerd`)**: Runs on the host, manages images, containers, networks, and storage.
- **Docker CLI (`docker`)**: Command-line tool to interact with the daemon.

## Basic Commands


### Build an image
```sh
docker build -t myapp:latest .
# -t: Tag the image with a name
# .: Build context (current directory)
```

### Run a container
```sh
docker run -d -p 5000:5000 --name mycontainer myapp:latest
# -d: Detached mode
# -p: Map host port to container port
# --name: Assign a name to the container
```

### Pull an image from a registry
```sh
docker pull nginx:alpine
```

### Push an image to a registry
```sh
docker tag myapp:latest myrepo/myapp:latest
docker push myrepo/myapp:latest
```

### List running containers
```sh
docker ps
```

### List all containers (including stopped)
```sh
docker ps -a
```

### List available images
```sh
docker images
```

## Container Management


### Stop a running container
```sh
docker stop <container_id or name>
```

### Start a stopped container
```sh
docker start <container_id or name>
```

### Restart a container
```sh
docker restart <container_id or name>
```

### Remove a container
```sh
docker rm <container_id or name>
# Add -f to force remove a running container
```

### Remove an image
```sh
docker rmi <image_id or name>
```

### View container logs
```sh
docker logs <container_id or name>
```

### Execute a command in a running container
```sh
docker exec -it <container_id or name> /bin/sh
# -it: Interactive terminal
```

## Docker Compose


Docker Compose is a tool for defining and running multi-container Docker applications. It uses a YAML file (`docker-compose.yml`) to configure application services, networks, and volumes, making it easy to manage complex setups.

**Key Features:**
- Define multiple services (e.g., web, db, cache) in one file
- Configure networks and volumes
- Start, stop, and rebuild all services with a single command

**Example `docker-compose.yml`:**
```yaml
version: '3.8'
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend
  app:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
    environment:
      - ENV=production
    volumes:
      - appdata:/app/data
    networks:
      - frontend
      - backend
  db:
    image: postgres:13
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - dbdata:/var/lib/postgresql/data
    networks:
      - backend
volumes:
  appdata:
  dbdata:
networks:
  frontend:
  backend:
```

**Commands:**
- Start all services:
  ```sh
  docker-compose up -d
  ```
- Stop all services:
  ```sh
  docker-compose down
  ```
- View logs for all services:
  ```sh
  docker-compose logs
  ```
- Rebuild services after code changes:
  ```sh
  docker-compose up --build
  ```

## Networking



### Bridge Network
**What:** The default network driver. Creates a private, isolated network on the Docker host. Containers on the same bridge network can communicate with each other, but are isolated from containers on other bridge networks and the host by default.

**When to use:**
- For most standalone applications running on a single host.
- When you want containers to communicate with each other but not expose all ports to the host.

**How to use:**
```sh
# Create a custom bridge network
docker network create --driver bridge my_bridge
# Run containers on the custom network
docker run -d --network my_bridge --name web nginx
docker run -d --network my_bridge --name app busybox sleep 3600
# Test connectivity
docker exec app ping -c 2 web
```
**Tip:** Use custom bridge networks (not the default "bridge") to enable automatic DNS-based service discovery between containers.

---

### Host Network
**What:** Removes network isolation between the container and the Docker host. The container shares the host's networking namespace and IP address.

**When to use:**
- When you need maximum network performance (no NAT or port mapping overhead).
- When the container must listen on the same network interfaces as the host (e.g., monitoring agents, VPNs).
- Only available on Linux.

**How to use:**
```sh
docker run --network host nginx
# Nginx will listen on the host's ports directly
```
**Caution:** All ports exposed by the container are available on the host. Use with care to avoid port conflicts and security risks.

---

### Overlay Network
**What:** Enables communication between containers running on different Docker hosts (nodes). Used primarily with Docker Swarm or Kubernetes for multi-host networking.

**When to use:**
- For distributed applications or microservices spanning multiple hosts.
- When you need service discovery and load balancing across a cluster.

**How to use:**
```sh
# Initialize Docker Swarm (if not already)
docker swarm init
# Create an overlay network
docker network create -d overlay my_overlay
# Deploy services to the overlay network
docker service create --name web --network my_overlay nginx
```
**Note:** Overlay networks require a cluster manager (Swarm, Kubernetes) and are not available in standalone Docker Engine mode.

---

### Macvlan Network
**What:** Assigns a unique MAC address to each container, making it appear as a physical device on the network. Containers can be given IP addresses on the local LAN, just like physical machines.

**When to use:**
- When containers need to appear as full-fledged devices on the local network (e.g., for legacy applications, network appliances, or when integrating with systems that require unique MAC/IP addresses).
- When you want containers to be accessible from the local network without port mapping.

**How to use:**
```sh
# Create a macvlan network (replace eth0 with your host's network interface)
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 my_macvlan
# Run a container on the macvlan network
docker run --network my_macvlan --name legacy-app busybox sleep 3600
```
**Caution:** Macvlan networks can be tricky to set up and may require additional host network configuration. Containers on macvlan networks cannot communicate with the host by default.

---

**Summary Table:**

| Driver   | Use Case                                 | Host Access | Multi-Host | Example Command |
|----------|------------------------------------------|-------------|------------|-----------------|
| Bridge   | Default, single-host, isolated networks  | Port-mapped | No         | `docker network create --driver bridge mynet` |
| Host     | Max performance, host-level access       | Full        | No         | `docker run --network host ...` |
| Overlay  | Multi-host, clustering, Swarm/K8s        | Port-mapped | Yes        | `docker network create -d overlay mynet` |
| Macvlan  | LAN integration, legacy, unique MAC/IP   | Direct      | No         | `docker network create -d macvlan ...` |

Choose the network driver that best fits your application's architecture and security requirements.

## Storage


Docker supports several storage options for persisting and sharing data:

- **Volumes:** Managed by Docker, best for persistent data. Volumes are stored outside the container's filesystem, making them ideal for databases and user data.
  - *Create and use a volume:*
    ```sh
    docker volume create mydata
    docker run -v mydata:/data busybox
    ```
  - *Inspect a volume:*
    ```sh
    docker volume inspect mydata
    ```
- **Bind mounts:** Map a specific file or directory from the host into the container. Useful for development (live code reload).
  - *Example:*
    ```sh
    docker run -v /Users/spu/app:/app busybox
    ```
- **tmpfs:** Store data in the host's memory only (non-persistent, fast, erased on container stop).
  - *Example:*
    ```sh
    docker run --tmpfs /app/tmp:rw,size=100m busybox
    ```

## Best Practices


- **Use official base images:** Start from trusted images (e.g., `python:3.10-slim`, `nginx:alpine`) to reduce vulnerabilities.
- **Minimize image layers:** Combine commands where possible to keep images small and builds fast.
  - *Example:*
    ```Dockerfile
    RUN apt-get update && apt-get install -y \
        package1 \
        package2 \
      && rm -rf /var/lib/apt/lists/*
    ```
- **Ephemeral containers:** Design containers to be stateless; store persistent data in volumes.
- **Use `.dockerignore`:** Exclude files/folders not needed in the image to speed up builds and reduce image size.
  - *Example `.dockerignore`:*
    ```
    __pycache__/
    *.pyc
    .git
    node_modules
    Dockerfile*
    *.log
    ```
- **Security practices:**
  - Avoid running as root inside containers. Use a non-root user:
    ```Dockerfile
    RUN useradd -m appuser
    USER appuser
    ```
  - Keep images up to date and scan for vulnerabilities.
  - Limit container capabilities and network access.
- **Principle of least privilege:** Only grant containers the permissions and resources they need.
- **Tag images properly:** Use semantic versioning and avoid using `latest` in production.
- **Automate builds and tests:** Use CI/CD pipelines to build, test, and deploy images automatically.

By following these concepts and best practices, you can effectively use Docker to build, ship, and run applications in a consistent, secure, and efficient manner.
