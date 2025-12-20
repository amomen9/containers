# Install Docker Desktop alongside Docker Engine (Linux)

This is a minimal checklist for installing `Docker Desktop` when `Docker Engine` is already installed.

Notes:
- On Linux, Docker Desktop typically includes its own engine components. Running two stacks can be confusing.
- If you only need `docker` CLI + daemon, consider using Docker Engine alone.

---

## 1) Uninstall conflicting packages

```shell
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do
  sudo apt-get remove -y "$pkg"
done
```

---

## 2) Clean up old Docker Engine installs (optional)

If you need a full cleanup, follow the official steps:
- https://docs.docker.com/engine/install/ubuntu/#uninstall-docker-engine

---

## 3) Install Docker Desktop

Follow Docker’s official installer and post-install steps for your distro:
- https://docs.docker.com/desktop/setup/install/linux/

After installation:

```shell
# Confirm CLI can talk to the daemon
docker version

# Check active context (Desktop often creates its own context)
docker context ls
```

If you want Desktop to be the default context:

```shell
# Example name is often 'desktop-linux' on Linux
docker context use desktop-linux
```
