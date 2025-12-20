# Portainer admin password recovery

Use this procedure when you’ve lost the Portainer admin password and your Portainer data is stored in the `portainer_data` volume.

---

## Steps

1. Stop the Portainer container.

```shell
docker stop <portainer-container-id-or-name>
```

2. Pull the reset helper.

```shell
docker pull portainer/helper-reset-password
```

3. Run the helper against the Portainer data volume.

```shell
docker run --rm -v portainer_data:/data portainer/helper-reset-password
```

4. Start Portainer again.

```shell
docker start <portainer-container-id-or-name>
```
