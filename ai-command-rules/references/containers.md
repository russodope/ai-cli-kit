# Docker & Containers Reference

## Core Principles for AI Coding Contexts

1. **Always use `--rm`** for one-off containers — avoid orphans
2. **Always tag images explicitly** — never rely on `latest`
3. **Prefer `docker compose`** over bare `docker run` for multi-service setups
4. **Non-interactive by default** — scripts must not need tty input

---

## Image Operations

```bash
# Build
docker build -t myapp:1.0 .
docker build -t myapp:1.0 -f Dockerfile.prod .
docker build --no-cache -t myapp:1.0 .      # Force fresh build

# Pull
docker pull python:3.11-slim               # Always pin version
docker pull --platform linux/amd64 image  # Force platform (Apple Silicon)

# List / remove images
docker images
docker rmi myapp:1.0
docker image prune -f                      # Remove dangling images
```

---

## Container Run Patterns

```bash
# One-off command — always use --rm
docker run --rm python:3.11-slim python --version

# Interactive shell — debugging only
docker run --rm -it python:3.11-slim bash

# Run app in background
docker run -d --name myapp -p 8080:8080 myapp:1.0

# With environment variables
docker run --rm \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  --env-file .env \
  myapp:1.0

# With volume mounts
docker run --rm \
  -v "$(pwd)/data:/app/data" \       # Unix
  myapp:1.0

# Windows PowerShell volume mount
docker run --rm `
  -v "${PWD}/data:/app/data" `
  myapp:1.0

# With network
docker run --rm --network mynetwork myapp:1.0

# Resource limits
docker run --rm --memory="512m" --cpus="1.0" myapp:1.0
```

---

## Exec Into Running Container

```bash
# Execute command in running container
docker exec myapp python script.py
docker exec -it myapp bash              # Interactive shell

# Run as specific user
docker exec -u root myapp bash
docker exec -u 1000:1000 myapp bash
```

---

## Docker Compose

```bash
# Start services
docker compose up -d                    # Background
docker compose up --build              # Rebuild images first

# Stop
docker compose down
docker compose down -v                 # Also remove volumes

# Logs
docker compose logs -f service_name

# Execute in service
docker compose exec app bash
docker compose exec app python manage.py migrate

# Run one-off command (new container, auto-removed)
docker compose run --rm app pytest
docker compose run --rm app python script.py

# Scale
docker compose up -d --scale worker=3
```

---

## Sandbox / CI Context Rules

When running inside Docker as part of a pipeline or sandbox:

```bash
# Check if already inside Docker
[ -f /.dockerenv ] && echo "Inside Docker"

# Usually running as root in containers — don't use sudo
whoami    # Verify current user

# Temp files — use /tmp
TMPFILE=$(mktemp /tmp/myapp.XXXXXX)
trap "rm -f $TMPFILE" EXIT

# Network check before pulling
curl -sf --max-time 5 https://registry-1.docker.io/v2/ || {
  echo "No Docker Hub access — using local images only"
  exit 1
}
```

---

## Dockerfile Best Practices

```dockerfile
# Pin base image with digest for reproducibility
FROM python:3.11-slim

# Non-root user (security)
RUN useradd -m -u 1000 appuser

# Layer caching — copy deps first
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Then copy app code
COPY --chown=appuser:appuser . .

USER appuser

# Explicit entrypoint
ENTRYPOINT ["python"]
CMD ["app.py"]
```

---

## Common Errors & Fixes

| Error | Cause | Fix |
|---|---|---|
| `permission denied` on volume | Host file owned by different UID | `--user $(id -u):$(id -g)` or `chmod` |
| `exec format error` | Wrong CPU arch | `--platform linux/amd64` |
| Container exits immediately | No foreground process | Add `tail -f /dev/null` or fix CMD |
| `network not found` | Network not created | `docker network create mynetwork` |
| `bind: address already in use` | Port conflict | Change `-p 8081:8080` |
| `no space left on device` | Docker disk full | `docker system prune -f` |
| `image not found` locally | Forgot to build/pull | `docker build` or `docker pull` first |

---

## Cleanup

```bash
# Remove stopped containers
docker container prune -f

# Remove unused images
docker image prune -f

# Remove unused volumes
docker volume prune -f

# Nuclear option — remove everything unused
docker system prune -af --volumes
```
