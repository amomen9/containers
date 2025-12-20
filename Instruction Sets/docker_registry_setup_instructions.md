# Setup a local Docker registry

This guide shows (A) a quick local registry and (B) a more complete setup that uses a non-default port and allows (optional) authentication.

---

## A) Quick registry (single command)

```shell
docker run -d \
  --name registry \
  --restart=always \
  -p 5000:5000 \
  registry:2
```

Tag and push:

```shell
docker tag my-image localhost:5000/my-image

docker push localhost:5000/my-image
```

Persist data:

```shell
docker run -d \
  --name registry \
  --restart=always \
  -p 5000:5000 \
  -v /path/on/host:/var/lib/registry \
  registry:2
```

---

## B) Registry on port 10000 + daemon config (insecure registry)

### 1) Pull a registry image and run it

```shell
docker pull registry:2.8.3

docker run -d \
  --name docker-registry-2.8.3 \
  -p 10000:5000 \
  registry:2.8.3
```

### 2) Allow the insecure registry (Docker daemon)

Edit `/etc/docker/daemon.json` (create if missing) and add:

```json
{
  "insecure-registries": ["localhost:10000"]
}
```

Restart Docker:

```shell
sudo systemctl restart docker
```

---

## C) Registry configuration file (optional)

If you want to manage registry settings explicitly, create configuration paths:

```shell
sudo mkdir -p /opt/registry/config
sudo nano /opt/registry/config/config.yml
```

Example config (from the notes, kept as-is and formatted):

```yaml
version: 0.1
storage:
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: ":10000"
  headers:
    X-Content-Type-Options: "nosniff"
health:
  storages:
    - filesystem
notifications:
  events:
    push:
      enabled: true
auth:
  service: registry
  realm: http://localhost:10000/v2/
  htpasswd:
    file: /opt/registry/auth/passwd
```

---

## D) Authentication password file (optional)

Create the auth directory:

```shell
sudo mkdir -p /opt/registry/auth
```

Create users with `htpasswd`:

```shell
# Create a new password file and add user 'admin'
htpasswd -c /opt/registry/auth/passwd admin

# Add additional users (without -c)
htpasswd /opt/registry/auth/passwd <user>
```

Clarification:
- If you enable `htpasswd` in `config.yml`, your registry container must mount the password file at the same path the config expects.
