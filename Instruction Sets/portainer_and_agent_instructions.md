# Portainer (CE/EE) + Portainer Agent

This guide sets up `Portainer` and (optionally) the `Portainer Agent`. It includes an offline image transfer pattern and a WSL-specific note for Windows.

---

## 1) Get the Portainer image

Choose one:

### Option A: Online pull (simple)

```shell
# Community Edition (CE)
docker pull portainer/portainer-ce:latest

# Business Edition (EE)
docker pull portainer/portainer-ee:latest
```

### Option B: Offline transfer (restricted networks)

**On a machine that can pull:**

```shell
docker pull portainer/portainer-ce:latest

docker save -o portainer-ce-latest.tar portainer/portainer-ce:latest

# Optional: place in a web-download directory
sudo mv portainer-ce-latest.tar /var/www/html/download/
sudo chown -R www-data:www-data /var/www/html/download/
```

**On the target machine:**

```shell
docker load -i portainer-ce-latest.tar
```

---

## 2) Create the Portainer data volume

```shell
docker volume create portainer_data
```

---

## 3) Run Portainer

### Portainer CE (common)

```shell
docker run -d \
  --name portainer \
  --restart=unless-stopped \
  -p 9000:9000 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:latest
```

### Portainer EE (example variants from the notes)

```shell
# EE variant exposing 8000 + 9443
docker run -d \
  --name portainer \
  --restart=always \
  -p 8000:8000 \
  -p 9443:9443 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ee:latest
```

---

## 4) Validate and access UI

```shell
docker ps
```

Open (example):
- `http://<host-ip>:9000/`

Then set the initial admin password.

---

## 5) Install Portainer Agent (optional)

### Online pull

```shell
docker pull portainer/agent:latest
```

### Offline transfer

**On a machine that can pull:**

```shell
docker pull portainer/agent:latest

docker save -o portainer-agent-latest.tar portainer/agent:latest
```

**On the target machine:**

```shell
docker load -i portainer-agent-latest.tar
```

### Run the agent

```shell
docker run -d \
  --name portainer_agent \
  --restart=always \
  -p 9001:9001 \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /var/lib/docker/volumes:/var/lib/docker/volumes \
  portainer/agent:latest
```

---

## 6) Windows + WSL note (Portainer access from other machines)

If Portainer runs inside WSL and remote machines can’t reach it directly, you can forward a Windows port to the WSL IP.

```powershell
# Replace <WSL_INTERNAL_IP> with the WSL distro IP
netsh interface portproxy add v4tov4 listenport=9000 listenaddress=0.0.0.0 connectport=9000 connectaddress=<WSL_INTERNAL_IP>
```

Tip: you can typically get the WSL IP using:

```powershell
wsl hostname -I
```
