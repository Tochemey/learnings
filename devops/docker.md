# Docker Commands

Quick reference for day-to-day Docker usage. See [DevOps README](./README.md) for more resources.

**Table of contents**

1. [Images](#images)
2. [Containers (run, list, stop)](#containers-run-list-stop)
3. [Logs, exec & debugging](#logs-exec--debugging)
4. [Ports & volumes](#ports--volumes)
5. [Build & tag](#build--tag)
6. [Cleanup & prune](#cleanup--prune)
7. [Docker Compose](#docker-compose)
8. [Networks](#networks)
9. [Registry & export](#registry--export)

---

## Images

List images:

```bash
docker images
# or: docker image ls
```

Pull an image:

```bash
docker pull image_name:tag
```

Remove an image:

```bash
docker rmi image_name:tag
docker rmi -f image_name:tag   # force if in use
```

Tag an image:

```bash
docker tag image_name:tag image_name:new_tag
```

Remove unused images (dangling):

```bash
docker image prune
docker image prune -a   # all unused, not just dangling
```

---

## Containers (run, list, stop)

List running containers:

```bash
docker ps
```

List all containers (including stopped):

```bash
docker ps -a
```

Run a container (interactive + TTY):

```bash
docker run -it image_name
```

Run in background (detached):

```bash
docker run -d --name my_container image_name
```

Stop a container:

```bash
docker stop container_name
```

Start an existing container:

```bash
docker start container_name
```

Restart:

```bash
docker restart container_name
```

Remove a container (must be stopped unless `-f`):

```bash
docker rm container_name
docker rm -f container_name   # force remove running container
```

Remove all stopped containers:

```bash
docker container prune
```

---

## Logs, exec & debugging

Stream container logs:

```bash
docker logs container_name
docker logs -f container_name          # follow
docker logs --tail 100 container_name  # last 100 lines
docker logs -f --since 5m container_name
```

Execute a command in a running container:

```bash
docker exec container_name ls /
docker exec -it container_name /bin/sh   # interactive shell (Alpine)
docker exec -it container_name /bin/bash # interactive shell (Debian/Ubuntu)
```

Inspect container (JSON):

```bash
docker inspect container_name
docker inspect --format '{{.State.Status}}' container_name
```

Live resource usage:

```bash
docker stats                    # all running containers
docker stats container_name     # single container
```

Process list inside container:

```bash
docker top container_name
```

Show filesystem changes (modified/added/deleted):

```bash
docker diff container_name
```

Copy file between host and container:

```bash
docker cp container_name:/path/in/container ./local_path
docker cp ./local_file container_name:/path/in/container
```

---

## Ports & volumes

Run with port mapping:

```bash
docker run -p host_port:container_port image_name
docker run -p 8080:80 -p 443:443 image_name
```

Run with volume mount:

```bash
docker run -v host_path:container_path image_name
docker run -v /data:/app/data image_name
docker run -v my_volume_name:/app/data image_name   # named volume
```

List port mappings for a container:

```bash
docker port container_name
```

---

## Build & tag

Build an image:

```bash
docker build -t image_name:tag .
docker build -t image_name:tag -f Dockerfile.alt .
```

Build without cache (full rebuild):

```bash
docker build --no-cache -t image_name:tag .
```

Build with build args:

```bash
docker build --build-arg NODE_ENV=production -t image_name:tag .
```

---

## Cleanup & prune

Remove all stopped containers:

```bash
docker container prune
```

Remove unused images (dangling):

```bash
docker image prune
docker image prune -a   # all unused images
```

Remove all unused networks:

```bash
docker network prune
```

Remove everything unused (containers, networks, images, optional build cache):

```bash
docker system prune
docker system prune -a        # include unused images
docker system prune -a --volumes   # include unused volumes (careful!)
```

Delete containers matching a pattern:

```bash
docker ps -a | grep "pattern" | awk '{print $1}' | xargs docker rm -f
```

Delete images matching a pattern:

```bash
docker images -a | grep "pattern" | awk '{print $3}' | xargs docker rmi -f
```

---

## Docker Compose

Start services (build if needed):

```bash
docker compose up -d
docker compose up -d --build
```

Stop and remove containers:

```bash
docker compose down
docker compose down -v   # remove volumes too
```

View logs:

```bash
docker compose logs
docker compose logs -f service_name
```

Run a one-off command in a service:

```bash
docker compose run --rm service_name command
docker compose exec service_name /bin/sh
```

List running compose services:

```bash
docker compose ps
```

---

## Networks

List networks:

```bash
docker network ls
```

Create a network:

```bash
docker network create my_network
```

Run a container on a specific network:

```bash
docker run --network my_network image_name
```

Inspect a network (see connected containers):

```bash
docker network inspect my_network
```

Remove unused networks:

```bash
docker network prune
```

---

## Registry & export

Log in to a registry:

```bash
docker login
docker login registry.example.com
```

Push an image:

```bash
docker push image_name:tag
```

Save image to a tar file (for air-gapped or backup):

```bash
docker save -o image.tar image_name:tag
```

Load image from a tar file:

```bash
docker load -i image.tar
```

Export a container filesystem to tar (no image metadata):

```bash
docker export container_name -o container.tar
```

---

## See also

- [Kubernetes](./kubernetes.md) — useful `kubectl` commands
- [Tips](./tips.md) — production survival guide
