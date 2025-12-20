# Modify container variables (environment, hostname, ports)

This guide explains what you can change at container creation time, and what to do when a container already exists.

Key idea:
- A container’s environment variables, published ports, and hostname are defined at `docker run` time. If you need to change them later, you typically recreate the container.

---

## A) Set variables when creating a container

```shell
# Set hostname
docker run --hostname=your_new_hostname your_image

# Set environment variables
docker run -e "VAR_NAME=value" your_image

# Publish ports (host:container)
docker run -p host_port:container_port your_image
```

---

## B) When the container already exists

### Option 1 (common): recreate the container

1. Record the current settings:

```shell
docker inspect <container>
```

2. Stop and remove the container:

```shell
docker stop <container>
docker rm <container>
```

3. Run a new container with the desired flags (`-e`, `-p`, `--hostname`, volumes, etc.).

---

### Option 2: commit → re-run (use with caution)

This preserves filesystem changes, but it can hide configuration drift. Prefer `Dockerfile` + rebuild when possible.

1. Create a new image from the container:

```shell
docker commit \
  -m "Your commit message" \
  -a "Author Name" \
  container_id_or_name \
  repository/new_image_name:tag
```

2. Stop and remove the existing container:

```shell
docker stop container_id_or_name
docker rm container_id_or_name
```

3. Run a new container from the committed image with the new settings.
