# Run PostgreSQL in Docker

This guide creates a single PostgreSQL container with a named volume for persistence.

---

## 1) Get the image

Choose one:

```shell
# Online
docker pull postgres:latest

# Or a specific version
docker pull postgres:14.2
```

If you have an offline tarball:

```shell
docker load -i postgres-latest.tar
```

---

## 2) Create a container

```shell
docker run -d \
  --name postgresql-container \
  -e POSTGRES_PASSWORD=<strong-password> \
  -p 5432:5432 \
  -v postgresql-data:/var/lib/postgresql/data \
  postgres:latest
```

---

## 3) Validate

```shell
docker ps
```

Then connect to PostgreSQL on `localhost:5432` (or from remote hosts using the Docker host IP).
