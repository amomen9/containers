# Offline Docker image installation (pull → save → transfer → load)

Use this when the target machine cannot reach Docker registries (air-gapped networks, restricted connectivity, etc.).

---

## 1) Pull on a machine with internet access

```shell
# Example: Portainer CE
docker pull portainer/portainer-ce:latest

# Example: SQL Server 2022
docker pull mcr.microsoft.com/mssql/server:2022-latest
```

---

## 2) Save the image to a tar file

```shell
# Save with a clear filename
docker save -o portainer-ce-latest.tar portainer/portainer-ce:latest

docker save -o mssql-server-2022-latest.tar mcr.microsoft.com/mssql/server:2022-latest
```

---

## 3) Transfer the tar file

Transfer via USB, SCP, or an internal HTTP server.

Example (Apache/Nginx web directory pattern from the notes):

```shell
sudo mv *.tar /var/www/html/download/
sudo chown -R www-data:www-data /var/www/html/download/
```

---

## 4) Load on the target machine

```shell
docker load -i portainer-ce-latest.tar

docker load -i mssql-server-2022-latest.tar
```

---

## 5) Run a container from the loaded image

```shell
# Example
docker run --rm hello-world:latest
```

Tip:
- After `docker load`, verify with `docker images`.
