# Install Docker Engine (Linux) without Docker Desktop

This guide installs `Docker Engine` on Ubuntu/Debian-style systems **without** Docker Desktop and validates the install with `hello-world`. It also includes an **offline image transfer** pattern and a working example using the `mcr.microsoft.com/mssql/server` image.

---

## 1) Remove conflicting packages

On Ubuntu, remove packages that may conflict with Docker’s official packages:

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y "$pkg"
done
```

---

## 2) Add Docker’s APT repository and install

```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=$(dpkg --print-architecture)] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt-get update -y
sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

Sanity-check contexts:

```shell
sudo docker context ls
```

---

## 3) Allow running Docker without `sudo` (optional but common)

Add your user to the `docker` group, then refresh group membership.

```shell
sudo usermod -aG docker <your-username>
newgrp docker
```

Notes:
- Use `newgrp docker` to refresh group membership for the current shell.
- Log out and back in if group changes do not apply.

---

## 4) Validate with `hello-world`

### Online flow

```shell
docker run hello-world
```

### Offline flow (air-gapped / restricted networks)

If the target machine cannot pull images directly, use a machine that can pull, then export/import:

**On a machine that can pull:**

```shell
docker pull hello-world:latest

# Save for transfer
docker save -o hello-world.tar hello-world:latest
```

**On the target machine:**

```shell
docker load -i hello-world.tar

docker run hello-world:latest
```

If you host tar files via a web server (as in the original notes), make sure the download directory is readable:

```shell
sudo mv hello-world.tar /var/www/html/download/
sudo chown -R www-data:www-data /var/www/html/download/
```

---

## 5) SQL Server 2022 container (offline-friendly)

### Prerequisites

- `Docker Engine` installed.
- At least **2 GB RAM** and **2 GB disk** available.
- A strong `SA` password (SQL Server enforces a password policy).

### Pull and transfer the image

**On a machine that can pull:**

```shell
sudo docker pull mcr.microsoft.com/mssql/server:2022-latest

docker images

docker save -o mcr.microsoft.com-mssql-server.tar mcr.microsoft.com/mssql/server:2022-latest
```

**On the target machine:**

```shell
docker load -i mcr.microsoft.com-mssql-server.tar
```

### Run SQL Server

Important:
- You must set `ACCEPT_EULA=Y`.
- Weak passwords prevent the container from starting.

```shell
# Example: map host port 14330 -> container port 1433
docker run -d \
  --name sql_container_2 \
  -e "ACCEPT_EULA=Y" \
  -e "MSSQL_SA_PASSWORD=<strong-password>" \
  -p 14330:1433 \
  mcr.microsoft.com/mssql/server:2022-latest
```

Optional: enable SQL Agent inside the running container:

```shell
docker exec -it sql_container_2 /opt/mssql/bin/mssql-conf set sqlagent.enabled true
```

### Useful troubleshooting commands

```shell
# Image metadata
docker image history --no-trunc mcr.microsoft.com/mssql/server:2022-latest

# Container logs (includes SQL Server error output)
docker logs --details -t -f sql_container_2 --since 60m

# Container lifecycle events
docker events --filter 'type=container' --since 60m
```

---

## 6) Install `sqlcmd` (Microsoft tools) on Ubuntu 22.04

```shell
curl https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
curl https://packages.microsoft.com/config/ubuntu/22.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list

sudo apt-get update -y
sudo apt-get install -y mssql-tools18 unixodbc-dev
```

(Optional) add tools to `PATH`:

```shell
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' | sudo tee -a /etc/environment
source /etc/environment
```
