# Docker Cheat Sheet

This document consolidates **command snippets and examples** from the original notes.
For full step-by-step guides, see the instruction sets in [`Instruction Sets/`](../Instruction%20Sets/).

---

## Important paths and directories

### Docker host directories (Linux)

- Docker runtime sockets and transient state often live under `/var/run/`.
- Docker persistent state typically lives under `/var/lib/docker/`.

```text
/var/run/docker/
/var/lib/docker/
```

### Common Docker JSON files (Linux)

```text
/var/lib/docker/containers/<container-id>/config.v2.json
/var/lib/docker/containers/<container-id>/hostconfig.json
/var/lib/docker/image/overlay2/repositories.json
```

---

## Images

### List images

```shell
# All images
docker images

# Include intermediate layers/images
docker images -a

# Filter by repository
docker images nginx

# Tag-specific
docker images nginx:tagname

# Include digests
docker images --digests [REPOSITORY][:TAG]
```

### Remove images

```shell
# Remove by image ID or name
docker rmi [IMAGE_ID_OR_NAME]
```

### Image history / layers

```shell
# List layers and commands used to build the image
docker history [IMAGE_NAME]

# Detailed history for an image (useful for metadata and labels)
docker image history --no-trunc mcr.microsoft.com/mssql/server:2022-latest
```

---

## Containers

### List containers

```shell
# Running containers
docker ps

# All containers (running + stopped)
docker ps -a
# Equivalent:
docker container ls -a

# IDs only
docker ps -aq

# Latest created container
docker ps -l

# N last created containers
docker ps -n [NUMBER]

# Include size information
docker ps -s
```

### Start/stop/remove

```shell
# Stop a container
docker stop sql_container

# Remove a container (stop first, or use -f to force)
docker rm sql_container

# Start/stop (two separate commands)
docker start container_name
docker stop container_name
```

### `docker run` essentials

Notes:
- `docker run` **creates a new container**. Re-running it creates additional containers unless you remove them.
- Use `--name` to set a predictable container name.

```shell
# Create a new container (default command)
docker run <image>

# Interactive shell in a new container
docker run -it <image> bash

# Run a command in a new container and exit
docker run -it ubuntu lslogins

# Name a container
docker run -it --name my_container_name my_image /bin/bash

# Run privileged (container can access more of the host; use sparingly)
docker run --privileged -d your_image

# Bind mount example
# Host path -> container path
docker run -v /path/to/host/directory:/path/in/container -d your_image
```

### `docker exec` (run a command in a running container)

```shell
# General form
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

# Example: open a shell as root
docker exec -u root -it sql_container_2 bash
```

Common options:
- `-d`: detached
- `-i`: keep STDIN open
- `-t`: allocate a pseudo-TTY
- `--user`: run as a specific user/UID
- `--env`: set environment variable(s)
- `--workdir`: set working directory

---

## Inspecting and troubleshooting

### Inspect container JSON

```shell
# Full JSON
docker inspect portainer

# View environment variables (example: SQL Server password)
# Note: the exact output format can vary; adjust parsing to your shell.
docker inspect sql_container | grep MSSQL_SA_PASSWORD
```

### Logs and events

```shell
# Follow logs (timestamps + details)
docker logs --details -t -f sql_container --since 60m --until 10m

# Container lifecycle events
docker events --filter 'type=container' --since 60m --until 20m
```

---

## Volumes

```shell
# Create a named volume
docker volume create portainer_data

# Default volume location on Linux
# (Docker manages these paths; do not edit by hand.)
# /var/lib/docker/volumes/

# Create a bind-like volume (advanced; use a normal bind mount when possible)
docker volume create --name my_custom_volume \
  --opt type=none \
  --opt device=/path/to/custom/location \
  --opt o=bind

# Inspect a container to view attached volumes
docker inspect -f '{{ json .Mounts }}' portainer_be

# Remove all unused volumes
docker volume prune

# Find containers attached to a volume
docker ps -a --filter volume=VOLUME_NAME_OR_MOUNT_POINT
```

---

## Restart policies

Use `--restart` with `docker run`.

```text
no
on-failure[:max-retries]
always
unless-stopped
```

Example:

```shell
docker run -d --restart=unless-stopped --name my_service my_image
```

---

## Offline image transfer (pull/save/load)

This is the **command reference**; for a guided procedure see:
- [`offline_docker_image_installation_instructions.md`](../Instruction%20Sets/offline_docker_image_installation_instructions.md)

```shell
# On a machine that can pull from registries:
docker pull hello-world:latest

# Save to a tar file for transfer
docker save -o hello-world.tar hello-world:latest

# On the target machine:
docker load -i hello-world.tar
```

---

## Docker Hub and registry queries

### Search Docker Hub

```shell
docker search --filter is-official=true --filter stars=3 nginx
```

### List tags via registry HTTP API (requires `jq`)

```shell
# Install jq (example shown with snap)
snap install jq

# Example: Docker Hub library/nginx tags
TOKEN=$(curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:library/nginx:pull" | jq -r '.token')
curl -s -H "Authorization: Bearer $TOKEN" https://index.docker.io/v2/library/nginx/tags/list | jq '.tags'
```

---

## Local Docker registry (quick commands)

For a full setup (including insecure registry + auth), see:
- [`docker_registry_setup_instructions.md`](../Instruction%20Sets/docker_registry_setup_instructions.md)

```shell
# Run the registry
docker run -d -p 5000:5000 --restart always --name registry registry:latest

# Tag an image for your registry
docker tag my-image localhost:5000/my-image

# Push to the registry
docker push localhost:5000/my-image

# Persist registry data
docker run -d -p 5000:5000 --restart always --name registry \
  -v /path/on/host:/var/lib/registry \
  registry:2
```

---

## Cleanup by image-name pattern

These examples use `grep`, `awk`, and `xargs` (typical on Linux). Adjust for your environment.

```shell
# Find container IDs whose image name contains 'pattern'
docker ps -a --format '{{.ID}} {{.Image}}' | grep 'pattern' | awk '{print $1}'

# Stop and remove those containers
docker ps -a --format '{{.ID}} {{.Image}}' | grep 'pattern' | awk '{print $1}' | xargs -r docker stop
docker ps -a --format '{{.ID}} {{.Image}}' | grep 'pattern' | awk '{print $1}' | xargs -r docker rm

# Remove images matching the pattern
docker images --filter=reference='*pattern*' -q | xargs -r docker rmi -f
```

Behavior note:
- `docker rmi -f` **does not stop running containers**. Running containers keep running, but may show the image **ID** instead of the tag/name.

---

## Docker Compose / build

```shell
# Build an image from the current directory's Dockerfile
docker build -t pg_fad_advworks .

# Start a compose stack using a specific compose file
docker-compose -f /path/to/your-compose-file.yml up -d
```

---

## SQL Server container notes

### Common data file locations (Linux, inside the container filesystem)

```text
/var/opt/mssql/data/
```

If you need a guided SQL Server container setup, see:
- [`docker_engine_install_without_desktop_instructions.md`](../Instruction%20Sets/docker_engine_install_without_desktop_instructions.md)

---

## Windows helpers

### WSL port forwarding (Portainer example)

```powershell
# Forward port 9000 on Windows to port 9000 inside WSL
# Replace <WSL_INTERNAL_IP> with the WSL distro IP (e.g., from `wsl hostname -I`)
netsh interface portproxy add v4tov4 listenport=9000 listenaddress=0.0.0.0 connectport=9000 connectaddress=<WSL_INTERNAL_IP>
```

### Simple disk size listing (batch)

```bat
@echo off
for /f "tokens=1,2" %%a in ('wmic logicaldisk get DeviceID^, Size') do (
  if %%a NEQ DeviceID (
    set /a size_gb=%%b/1024/1024/1024
    echo %%a: %size_gb% GB
  )
)
```
