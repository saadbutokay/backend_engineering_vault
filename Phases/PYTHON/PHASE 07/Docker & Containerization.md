
```
The classic problem:
  "It works on my machine!"

  Dev laptop:   Python 3.12, Ubuntu 22.04, PostgreSQL 16
  Staging:      Python 3.11, CentOS 7,    PostgreSQL 14
  Production:   Python 3.10, Debian 11,   PostgreSQL 13
  
  Each environment is different.
  Each has different behavior, different bugs.

Docker solution:
  Package your app WITH its entire environment.
  
  [Your App + Python 3.12 + exact dependencies + config]
  = ONE container image
  
  Run that image ANYWHERE:
    ✅ Dev laptop → same
    ✅ Staging server → same
    ✅ Production server → same
    ✅ CI/CD pipeline → same
    ✅ Your teammate's Mac → same

Container = isolated, reproducible, portable environment
Docker = the tool to build and run containers
```

---

## Setup

Bash

```
# Install Docker
# Mac: Download Docker Desktop from docker.com
# Windows: Docker Desktop with WSL2

# Linux (Ubuntu):
sudo apt update
sudo apt install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) \
    signed-by=/etc/apt/keyrings/docker.gpg] \
    https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io \
    docker-buildx-plugin docker-compose-plugin

# Add user to docker group (no sudo needed)
sudo usermod -aG docker $USER
newgrp docker   # apply group change without logout

# Verify installation
docker --version
docker compose version
docker run hello-world
```

---

## 1. 🧠 Docker Concepts

text

```
Key terms:

IMAGE:     A snapshot/blueprint of your app environment.
           Like a class in OOP — a template.
           Read-only. Shareable. Stored in registries.
           
CONTAINER: A running instance of an image.
           Like an object from a class.
           Isolated process with its own filesystem.
           Can start, stop, restart, delete.

DOCKERFILE: Instructions to BUILD an image.
            Like a recipe: "start with Ubuntu,
            add Python, copy my code, run this command"

REGISTRY:  Where images are stored and shared.
           Docker Hub (public)
           AWS ECR, GitHub Container Registry (private)

VOLUME:    Persistent storage that survives container restarts.
           Container filesystem is temporary — dies with container.
           Volume data persists.

NETWORK:   How containers communicate with each other.
           Containers are isolated by default.
           Must explicitly connect them.

DOCKER COMPOSE: Tool to run MULTIPLE containers together.
           Define all services in one YAML file.
           One command starts everything.
```

### Container vs VM

text

```
VIRTUAL MACHINE:
┌─────────────────────────────────────┐
│  App A    │  App B    │  App C      │
│  OS A     │  OS B     │  OS C       │
│  2GB RAM  │  1GB RAM  │  1GB RAM    │
├───────────┼───────────┼─────────────┤
│           Hypervisor                 │
├─────────────────────────────────────┤
│           Host OS                    │
├─────────────────────────────────────┤
│           Hardware                   │
└─────────────────────────────────────┘
Each VM has its OWN OS kernel (heavy, slow)

DOCKER CONTAINERS:
┌──────────┬──────────┬──────────────┐
│  App A   │  App B   │  App C       │
│  Libs A  │  Libs B  │  Libs C      │
├──────────┴──────────┴──────────────┤
│         Docker Engine               │
├─────────────────────────────────────┤
│         Host OS (shared kernel)     │
├─────────────────────────────────────┤
│         Hardware                    │
└─────────────────────────────────────┘
Containers SHARE the host OS kernel (lightweight, fast)
Start in milliseconds vs minutes for VMs
Use MBs of RAM vs GBs for VMs
```

---

## 2. 📋 Your First Dockerfile

Bash

```
# Project setup
mkdir ~/projects/docker_demo
cd ~/projects/docker_demo

# Create a simple FastAPI app
mkdir -p app
cat > app/main.py << 'EOF'
from fastapi import FastAPI
import os

app = FastAPI()

@app.get("/")
def root():
    return {
        "message": "Hello from Docker!",
        "environment": os.getenv("APP_ENV", "development"),
        "version": "1.0.0"
    }

@app.get("/health")
def health():
    return {"status": "healthy"}
EOF

cat > requirements.txt << 'EOF'
fastapi==0.104.1
uvicorn[standard]==0.24.0
EOF
```

Dockerfile

```
# Dockerfile — instructions to build an image

# ─── STAGE 1: Base image ──────────────────────────────────────
# FROM: start with this base image
# python:3.12-slim = official Python 3.12 on Debian slim (small!)
FROM python:3.12-slim

# ─── Metadata ─────────────────────────────────────────────────
LABEL maintainer="you@example.com"
LABEL description="FastAPI Application"
LABEL version="1.0.0"

# ─── Environment variables ────────────────────────────────────
# PYTHONDONTWRITEBYTECODE: don't write .pyc files (smaller image)
# PYTHONUNBUFFERED: print stdout/stderr immediately (better logs)
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONFAULTHANDLER=1 \
    APP_ENV=production \
    PORT=8000

# ─── Working directory ────────────────────────────────────────
# All subsequent commands run from here
# Creates the directory if it doesn't exist
WORKDIR /app

# ─── Install system dependencies ──────────────────────────────
# Combine apt commands in one RUN to reduce layers
# Clean up apt cache in same layer to reduce image size
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        curl \
        gcc \
    && rm -rf /var/lib/apt/lists/*

# ─── Install Python dependencies ──────────────────────────────
# Copy ONLY requirements first
# This is key for layer caching:
# If requirements.txt unchanged → Docker uses cached layer
# If requirements.txt changed → reinstall packages
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# ─── Copy application code ────────────────────────────────────
# Copy code AFTER installing dependencies
# Code changes more often than dependencies
# This way: code changes don't invalidate pip cache
COPY . .

# ─── Non-root user ────────────────────────────────────────────
# Security: don't run as root!
RUN adduser --disabled-password --gecos "" --uid 1001 appuser
USER appuser

# ─── Port ─────────────────────────────────────────────────────
# Document which port the app uses
# EXPOSE doesn't actually open ports — just documentation
EXPOSE 8000

# ─── Healthcheck ──────────────────────────────────────────────
# Docker checks this to know if container is healthy
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# ─── Start command ────────────────────────────────────────────
# CMD: default command when container starts
# Use exec form (JSON array) not shell form
# Shell form: CMD uvicorn ...  (runs as shell child)
# Exec form: CMD ["uvicorn", ...]  (runs directly, receives signals)
CMD ["uvicorn", "app.main:app", \
     "--host", "0.0.0.0", \
     "--port", "8000", \
     "--workers", "1"]
```

### Building and Running

Bash

```
# ─────────────────────────────────────────
# BUILD the image
# ─────────────────────────────────────────
# -t = tag (name:version)
# . = build context (current directory)
docker build -t myapp:latest .
docker build -t myapp:1.0.0 .

# Watch the layers being built:
# Step 1/9: FROM python:3.12-slim
# Step 2/9: ENV PYTHONDONTWRITEBYTECODE=1...
# Step 3/9: WORKDIR /app
# ...

# Build with no cache (force rebuild everything)
docker build --no-cache -t myapp:latest .

# Build and show progress
docker build --progress=plain -t myapp:latest .

# ─────────────────────────────────────────
# VIEW images
# ─────────────────────────────────────────
docker images
docker images myapp
docker image ls

# Output:
# REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
# myapp        latest    abc123def456   2 minutes ago   182MB
# myapp        1.0.0     abc123def456   2 minutes ago   182MB

# ─────────────────────────────────────────
# RUN a container
# ─────────────────────────────────────────
# -d = detached (background)
# -p = port mapping: HOST_PORT:CONTAINER_PORT
# --name = give container a name
docker run -d -p 8000:8000 --name myapp myapp:latest

# Test it
curl http://localhost:8000/
curl http://localhost:8000/health

# Environment variables
docker run -d -p 8000:8000 \
    -e APP_ENV=development \
    -e DATABASE_URL=postgresql://... \
    --name myapp-dev \
    myapp:latest

# Mount a volume
docker run -d -p 8000:8000 \
    -v $(pwd)/logs:/app/logs \
    --name myapp \
    myapp:latest

# ─────────────────────────────────────────
# CONTAINER MANAGEMENT
# ─────────────────────────────────────────
docker ps                       # running containers
docker ps -a                    # all containers (including stopped)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

docker stop myapp               # graceful stop (SIGTERM)
docker start myapp              # start stopped container
docker restart myapp            # restart
docker rm myapp                 # delete container (must be stopped)
docker rm -f myapp              # force delete (even if running)

# ─────────────────────────────────────────
# CONTAINER INTERACTION
# ─────────────────────────────────────────
# View logs
docker logs myapp               # all logs
docker logs -f myapp            # follow (live)
docker logs --tail 100 myapp    # last 100 lines
docker logs --since 1h myapp    # last hour

# Execute command in running container
docker exec myapp ls -la        # run command
docker exec -it myapp bash      # interactive shell
docker exec -it myapp sh        # if no bash
docker exec myapp cat /etc/os-release

# Copy files to/from container
docker cp myapp:/app/logs/error.log ./
docker cp ./config.json myapp:/app/config.json

# Container stats
docker stats                    # real-time resource usage
docker stats myapp              # specific container
docker top myapp                # processes in container

# Container inspection
docker inspect myapp            # full details (JSON)
docker inspect myapp | grep IPAddress  # container IP

# ─────────────────────────────────────────
# CLEANUP
# ─────────────────────────────────────────
docker rm $(docker ps -aq)      # remove all stopped containers
docker rmi myapp:latest         # remove image
docker rmi $(docker images -q)  # remove all images
docker system prune             # remove ALL unused resources
docker system prune -a          # remove ALL including unused images
docker volume prune             # remove unused volumes
```

---

## 3. 🏗️ Multi-Stage Builds

Dockerfile

```
# Dockerfile.multistage
# Multi-stage builds = use multiple FROM statements
# Only the last stage goes into the final image
# Huge size reduction: build deps not in production image

# ─── STAGE 1: Builder ─────────────────────────────────────────
# Install build tools and compile dependencies
FROM python:3.12-slim AS builder

WORKDIR /build

# Install build dependencies (only needed to build, not to run)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        build-essential \
        gcc \
        libpq-dev \    
    && rm -rf /var/lib/apt/lists/*

# Install Python dependencies into a target directory
COPY requirements.txt .
RUN pip install --no-cache-dir \
    --prefix=/install \
    -r requirements.txt

# ─── STAGE 2: Production ──────────────────────────────────────
# Start fresh — no build tools!
FROM python:3.12-slim AS production

# Only copy COMPILED packages from builder stage
COPY --from=builder /install /usr/local

# Install ONLY runtime system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libpq5 \
        curl \
    && rm -rf /var/lib/apt/lists/*

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PORT=8000

WORKDIR /app

# Create non-root user
RUN adduser --disabled-password --gecos "" --uid 1001 appuser

# Copy only the application code
COPY --chown=appuser:appuser ./app ./app
COPY --chown=appuser:appuser alembic.ini .
COPY --chown=appuser:appuser alembic/ ./alembic/

USER appuser

EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD curl -f http://localhost:${PORT}/health || exit 1

CMD ["uvicorn", "app.main:app", \
     "--host", "0.0.0.0", \
     "--port", "8000"]


# ─── STAGE 3: Development (optional) ──────────────────────────
# Extra tools for development
FROM production AS development

USER root
RUN pip install --no-cache-dir \
    pytest pytest-asyncio httpx \
    black ruff mypy

# Development has hot-reload
USER appuser
CMD ["uvicorn", "app.main:app", \
     "--host", "0.0.0.0", \
     "--port", "8000", \
     "--reload"]
```

Bash

```
# Build specific stage
docker build --target production -t myapp:prod .
docker build --target development -t myapp:dev .

# Size comparison:
docker images | grep myapp
# myapp    prod    abc123    182MB   ← smaller (no build tools)
# myapp    dev     def456    380MB   ← larger (dev tools included)
```

---

## 4. 📋 .dockerignore

Bash

```
# .dockerignore — files to EXCLUDE from build context
# Like .gitignore but for Docker
# Faster builds + smaller images + no secrets leaked

# Virtual environments
venv/
.venv/
env/

# Python cache
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
*.egg-info/
dist/
build/

# Testing
.pytest_cache/
.coverage
htmlcov/
.mypy_cache/
.ruff_cache/

# Version control
.git/
.gitignore
.github/

# IDE
.vscode/
.idea/
*.swp
*.swo

# Docker files (don't include Docker files in image)
Dockerfile*
docker-compose*.yml
.dockerignore

# Secrets (NEVER include these!)
.env
.env.*
secrets/
*.pem
*.key

# Documentation
README.md
docs/
CHANGELOG.md

# OS files
.DS_Store
Thumbs.db

# Logs
logs/
*.log

# Node (if you have any frontend)
node_modules/
npm-debug.log

# Development data
data/
uploads/
media/
```

---

## 5. 🐙 Docker Compose — Multi-Container Apps

### What is Docker Compose?

text

```
Real apps need multiple services:
  - Your FastAPI application
  - PostgreSQL database
  - Redis cache
  - Celery worker
  - Nginx reverse proxy

Without Compose: run each container manually, manage networks manually
With Compose: ONE file defines everything, ONE command starts all

docker compose up    → start everything
docker compose down  → stop everything
```

### Complete docker-compose.yml

YAML

```
# docker-compose.yml
# Complete development environment

version: "3.9"

# ─── Services ─────────────────────────────────────────────────
services:

  # ─── PostgreSQL Database ────────────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: myapp_db

    environment:
      POSTGRES_USER: ${DB_USER:-myapp}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-myapp_secret}
      POSTGRES_DB: ${DB_NAME:-myapp_db}
      PGDATA: /var/lib/postgresql/data/pgdata

    volumes:
      # Named volume: data persists between docker compose down/up
      - postgres_data:/var/lib/postgresql/data
      # Optional: initialization SQL scripts
      # - ./scripts/init.sql:/docker-entrypoint-initdb.d/init.sql

    ports:
      # Expose DB to host for debugging (remove in production)
      - "5432:5432"

    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-myapp} -d ${DB_NAME:-myapp_db}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── Redis Cache ────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: myapp_redis

    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD:-redis_secret}
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru

    volumes:
      - redis_data:/data

    ports:
      - "6379:6379"

    healthcheck:
      test: ["CMD", "redis-cli", "--pass", "${REDIS_PASSWORD:-redis_secret}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── FastAPI Application ─────────────────────────────────────
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production         # which stage to build

    container_name: myapp_app
    image: myapp:latest

    environment:
      # Database
      DATABASE_URL: postgresql://${DB_USER:-myapp}:${DB_PASSWORD:-myapp_secret}@db:5432/${DB_NAME:-myapp_db}
      # Note: "db" = service name, Docker resolves it to container IP

      # Redis
      REDIS_URL: redis://:${REDIS_PASSWORD:-redis_secret}@redis:6379/0

      # App
      SECRET_KEY: ${SECRET_KEY:-dev-secret-change-in-production}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY:-jwt-secret-change-in-production}
      APP_ENV: ${APP_ENV:-development}
      DEBUG: ${DEBUG:-false}

    volumes:
      # Mount uploads directory for file persistence
      - uploads:/app/uploads
      # Hot reload in development (mount code)
      # Comment out in production:
      - ./app:/app/app

    ports:
      - "8000:8000"

    depends_on:
      # Don't start until DB and Redis are healthy
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── Celery Worker ──────────────────────────────────────────
  worker:
    build:
      context: .
      dockerfile: Dockerfile
      target: production

    container_name: myapp_worker

    # Override the CMD to run Celery instead of uvicorn
    command: celery -A app.tasks worker --loglevel=info --concurrency=4

    environment:
      DATABASE_URL: postgresql://${DB_USER:-myapp}:${DB_PASSWORD:-myapp_secret}@db:5432/${DB_NAME:-myapp_db}
      REDIS_URL: redis://:${REDIS_PASSWORD:-redis_secret}@redis:6379/0
      SECRET_KEY: ${SECRET_KEY:-dev-secret-change-in-production}

    volumes:
      - ./app:/app/app

    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── Celery Beat (Scheduler) ─────────────────────────────────
  beat:
    build:
      context: .
      dockerfile: Dockerfile
      target: production

    container_name: myapp_beat

    command: celery -A app.tasks beat --loglevel=info

    environment:
      DATABASE_URL: postgresql://${DB_USER:-myapp}:${DB_PASSWORD:-myapp_secret}@db:5432/${DB_NAME:-myapp_db}
      REDIS_URL: redis://:${REDIS_PASSWORD:-redis_secret}@redis:6379/0

    depends_on:
      - worker

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── Nginx Reverse Proxy ─────────────────────────────────────
  nginx:
    image: nginx:1.25-alpine
    container_name: myapp_nginx

    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      # Static files
      - ./static:/var/www/static:ro
      # SSL certificates (for production)
      # - ./certbot/conf:/etc/letsencrypt:ro

    ports:
      - "80:80"
      - "443:443"

    depends_on:
      app:
        condition: service_healthy

    restart: unless-stopped

    networks:
      - myapp_network

  # ─── Database Admin UI ──────────────────────────────────────
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: myapp_pgadmin

    environment:
      PGADMIN_DEFAULT_EMAIL: admin@myapp.com
      PGADMIN_DEFAULT_PASSWORD: admin_secret

    ports:
      - "5050:80"

    depends_on:
      - db

    networks:
      - myapp_network

    # Development only — comment out for production
    profiles:
      - tools    # only starts with: docker compose --profile tools up

# ─── Volumes ──────────────────────────────────────────────────
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
  uploads:
    driver: local

# ─── Networks ─────────────────────────────────────────────────
networks:
  myapp_network:
    driver: bridge
    # Optional: custom subnet
    # ipam:
    #   config:
    #     - subnet: 172.20.0.0/16
```

### Environment Files for Compose

Bash

```
# .env (loaded automatically by docker compose)
# One file for all services

# Database
DB_USER=myapp
DB_PASSWORD=super_secret_db_password
DB_NAME=myapp_db

# Redis
REDIS_PASSWORD=super_secret_redis_password

# Application
SECRET_KEY=your-super-secret-key-at-least-32-chars!!
JWT_SECRET_KEY=your-jwt-secret-key-at-least-32-chars!!
APP_ENV=development
DEBUG=true

# .env.production (for production deploys)
DB_PASSWORD=even_more_secret_production_password
SECRET_KEY=production-secret-key-never-commit-this!!
JWT_SECRET_KEY=production-jwt-secret-never-commit-this!!
APP_ENV=production
DEBUG=false
```

### docker-compose Commands

Bash

```
# ─────────────────────────────────────────
# STARTING AND STOPPING
# ─────────────────────────────────────────

# Start all services (build if needed)
docker compose up

# Start in background (detached)
docker compose up -d

# Start and rebuild images
docker compose up --build -d

# Start specific services only
docker compose up -d db redis

# Stop all services
docker compose down

# Stop and remove volumes (WARNING: deletes data!)
docker compose down -v

# Stop and remove everything
docker compose down --rmi all --volumes --remove-orphans

# ─────────────────────────────────────────
# MONITORING
# ─────────────────────────────────────────

# See all running services
docker compose ps

# View logs
docker compose logs                 # all services
docker compose logs -f              # follow all
docker compose logs -f app          # follow one service
docker compose logs --tail 50 app   # last 50 lines

# Resource usage
docker compose stats

# ─────────────────────────────────────────
# RUNNING COMMANDS
# ─────────────────────────────────────────

# Run command in a service container
docker compose exec app bash        # shell in running container
docker compose exec app python manage.py shell
docker compose exec db psql -U myapp myapp_db
docker compose exec redis redis-cli --pass redis_secret

# Run one-off command (new container)
docker compose run --rm app alembic upgrade head
docker compose run --rm app pytest tests/
docker compose run --rm app python scripts/seed_data.py

# ─────────────────────────────────────────
# MANAGING SERVICES
# ─────────────────────────────────────────

# Restart specific service
docker compose restart app

# Scale a service (multiple instances)
docker compose up -d --scale worker=3   # 3 Celery workers

# Pull latest images
docker compose pull

# Build images
docker compose build
docker compose build app            # build specific service
docker compose build --no-cache app # force rebuild

# ─────────────────────────────────────────
# PROFILES
# ─────────────────────────────────────────

# Start with tools profile (includes pgadmin)
docker compose --profile tools up -d

# ─────────────────────────────────────────
# ENVIRONMENT FILES
# ─────────────────────────────────────────

# Use specific env file
docker compose --env-file .env.production up -d

# Override compose file
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 6. 📁 Multiple Compose Files

YAML

```
# docker-compose.yml — BASE (shared between all environments)
version: "3.9"

services:
  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      retries: 5
    networks:
      - myapp_network

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    networks:
      - myapp_network

  app:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASSWORD}@db:5432/${DB_NAME}
      REDIS_URL: redis://:${REDIS_PASSWORD}@redis:6379/0
      SECRET_KEY: ${SECRET_KEY}
    depends_on:
      db:
        condition: service_healthy
    networks:
      - myapp_network

volumes:
  postgres_data:
  redis_data:

networks:
  myapp_network:
```

YAML

```
# docker-compose.dev.yml — DEVELOPMENT overrides
version: "3.9"

services:
  app:
    build:
      target: development          # use dev stage
    volumes:
      - ./app:/app/app             # hot reload
    ports:
      - "8000:8000"
    environment:
      APP_ENV: development
      DEBUG: "true"
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    ports:
      - "5432:5432"               # expose DB for tools

  redis:
    ports:
      - "6379:6379"               # expose Redis for tools

  # Dev-only: mail catcher
  mailhog:
    image: mailhog/mailhog:latest
    ports:
      - "8025:8025"               # web UI for email
      - "1025:1025"               # SMTP
    networks:
      - myapp_network
```

YAML

```
# docker-compose.prod.yml — PRODUCTION overrides
version: "3.9"

services:
  app:
    build:
      target: production
    restart: always
    deploy:
      replicas: 2                 # 2 instances
      resources:
        limits:
          cpus: "0.5"
          memory: 512M
    # No volume mount in production (use baked-in code)
    environment:
      APP_ENV: production
      DEBUG: "false"

  nginx:
    image: nginx:1.25-alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/prod.conf:/etc/nginx/conf.d/default.conf:ro
      - ./certbot/conf:/etc/letsencrypt:ro
    depends_on:
      - app
    restart: always
    networks:
      - myapp_network

  worker:
    restart: always
    deploy:
      replicas: 2

  # No pgadmin in production!
```

Bash

```
# Development:
docker compose -f docker-compose.yml -f docker-compose.dev.yml up -d

# Production:
docker compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Make aliases in your shell:
alias dc-dev='docker compose -f docker-compose.yml -f docker-compose.dev.yml'
alias dc-prod='docker compose -f docker-compose.yml -f docker-compose.prod.yml'

dc-dev up -d
dc-prod down
```

---

## 7. 🏭 Production-Ready Dockerfile

Dockerfile

```
# Dockerfile — Production-ready with Poetry
# Multi-stage, minimal size, security best practices

# ─── Stage 1: Dependencies ────────────────────────────────────
FROM python:3.12-slim AS deps

# Install Poetry
ENV POETRY_VERSION=1.7.1 \
    POETRY_HOME=/opt/poetry \
    POETRY_VIRTUALENVS_CREATE=false \
    POETRY_NO_INTERACTION=1

RUN pip install --no-cache-dir poetry==$POETRY_VERSION

WORKDIR /build

# Copy dependency files only
COPY pyproject.toml poetry.lock ./

# Install production dependencies (no dev deps)
RUN poetry install --only=main --no-root

# ─── Stage 2: Production ──────────────────────────────────────
FROM python:3.12-slim AS production

# Install runtime system deps only
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libpq5 \
        curl \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean

# Copy installed Python packages from deps stage
COPY --from=deps /usr/local/lib/python3.12/site-packages \
                 /usr/local/lib/python3.12/site-packages
COPY --from=deps /usr/local/bin /usr/local/bin

# Environment
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PYTHONFAULTHANDLER=1 \
    PORT=8000

WORKDIR /app

# Security: create non-root user
RUN groupadd --gid 1001 appgroup && \
    useradd --uid 1001 --gid appgroup \
            --no-create-home --shell /bin/false \
            appuser

# Copy application code (owned by appuser)
COPY --chown=appuser:appgroup ./app ./app
COPY --chown=appuser:appgroup ./alembic ./alembic
COPY --chown=appuser:appgroup ./alembic.ini .

# Create writable directories
RUN mkdir -p /app/logs /app/uploads && \
    chown -R appuser:appgroup /app/logs /app/uploads

# Switch to non-root
USER appuser

EXPOSE $PORT

# Healthcheck
HEALTHCHECK --interval=30s \
            --timeout=10s \
            --start-period=30s \
            --retries=3 \
    CMD curl -fsS http://localhost:${PORT}/health || exit 1

# Tini: proper init process (handles signals, zombies)
# (install in system deps stage if needed)
# ENTRYPOINT ["/usr/bin/tini", "--"]

CMD ["sh", "-c", \
     "uvicorn app.main:app \
      --host 0.0.0.0 \
      --port ${PORT} \
      --workers ${WORKERS:-1} \
      --log-level ${LOG_LEVEL:-info}"]
```

---

## 8. 📦 Volumes — Persistent Data

Bash

```
# ─────────────────────────────────────────
# TYPES OF VOLUMES
# ─────────────────────────────────────────

# 1. NAMED VOLUME (recommended for databases)
# Managed by Docker, stored in Docker's area
docker volume create mydata
docker run -v mydata:/app/data myimage

# 2. BIND MOUNT (development: hot reload)
# Mounts a specific host directory
docker run -v $(pwd)/app:/app/app myimage
# Or: -v /home/ubuntu/myapp:/app

# 3. TMPFS MOUNT (temporary, in-memory only)
docker run --tmpfs /tmp myimage

# ─────────────────────────────────────────
# VOLUME MANAGEMENT
# ─────────────────────────────────────────
docker volume ls                    # list volumes
docker volume inspect mydata        # volume details
docker volume rm mydata             # delete volume
docker volume prune                 # delete unused volumes

# Backup a volume
docker run --rm \
    -v mydata:/source:ro \
    -v $(pwd):/backup \
    alpine \
    tar czf /backup/mydata_backup.tar.gz -C /source .

# Restore a volume
docker run --rm \
    -v mydata:/target \
    -v $(pwd):/backup \
    alpine \
    tar xzf /backup/mydata_backup.tar.gz -C /target

# ─────────────────────────────────────────
# POSTGRES DATA PERSISTENCE
# ─────────────────────────────────────────
# Named volume = data survives container recreations
# docker compose down → data persists in 'postgres_data' volume
# docker compose down -v → DELETES the volume (data gone!)

# Backup PostgreSQL in Docker:
docker compose exec db pg_dump \
    -U myapp myapp_db \
    > backup_$(date +%Y%m%d).sql

# Restore:
docker compose exec -T db psql \
    -U myapp myapp_db \
    < backup_20240115.sql
```

---

## 9. 🌐 Networking

Bash

```
# ─────────────────────────────────────────
# DOCKER NETWORKS
# ─────────────────────────────────────────

# List networks
docker network ls
# NETWORK ID     NAME              DRIVER    SCOPE
# abc123def456   bridge            bridge    local   ← default
# xyz789ghi012   host              host      local   ← use host networking
# mno345pqr678   none              null      local   ← no networking
# stu901vwx234   myapp_network     bridge    local   ← custom (from compose)

# Create network
docker network create myapp_network
docker network create --driver bridge --subnet 172.20.0.0/16 myapp_net

# Connect container to network
docker network connect myapp_network mycontainer

# Inspect network
docker network inspect myapp_network

# Remove network
docker network rm myapp_network

# ─────────────────────────────────────────
# HOW CONTAINERS COMMUNICATE
# ─────────────────────────────────────────
# Containers on same network use SERVICE NAMES as hostnames

# In docker-compose.yml:
# services:
#   db:           ← service name
#     image: postgres
#   app:
#     environment:
#       DATABASE_URL: postgresql://user:pass@db:5432/mydb
#                                              ↑
#                                     "db" = service name
#                                     Docker resolves to container IP

# This ONLY works if both are on the SAME network!

# ─────────────────────────────────────────
# PORT MAPPING
# ─────────────────────────────────────────
# -p HOST_PORT:CONTAINER_PORT
# -p 8080:8000   ← host's 8080 maps to container's 8000
# -p 127.0.0.1:8000:8000  ← bind only on localhost (security)
# -p 0.0.0.0:8000:8000    ← bind on all interfaces

# Don't expose ports in production if behind nginx:
# app service: no ports: section
# nginx service: ports: ["80:80", "443:443"]
```

---

## 10. 🔐 Docker Security

Dockerfile

```
# Security best practices in Dockerfile

# ─── 1. Use specific base image versions ──────────────────────
# ❌ FROM python:latest  (unpredictable, changes without notice)
# ✅ FROM python:3.12.2-slim  (pinned exact version)

# ─── 2. Non-root user ─────────────────────────────────────────
RUN groupadd --gid 1001 appgroup && \
    useradd --uid 1001 --gid appgroup \
            --no-create-home \
            appuser
USER appuser
# Attacker who compromises container can't become root on host

# ─── 3. Read-only filesystem ──────────────────────────────────
# docker run --read-only --tmpfs /tmp myimage
# Or in compose:
# services:
#   app:
#     read_only: true
#     tmpfs:
#       - /tmp
#       - /var/run

# ─── 4. Minimal base image ────────────────────────────────────
# python:3.12-slim  (smaller attack surface than full Debian)
# python:3.12-alpine  (even smaller, uses musl libc)

# ─── 5. No secrets in Dockerfile ──────────────────────────────
# ❌ ENV DATABASE_PASSWORD=supersecret  (visible in image layers!)
# ✅ Pass secrets via environment variables at runtime

# ─── 6. Scan for vulnerabilities ──────────────────────────────
# docker scout cves myapp:latest  (Docker's built-in scanner)
# trivy image myapp:latest        (Trivy security scanner)

# ─── 7. .dockerignore ─────────────────────────────────────────
# Never include .env, private keys, etc. in build context

# ─── Security in docker-compose ───────────────────────────────
# security_opt:
#   - no-new-privileges:true
# cap_drop:
#   - ALL
# cap_add:
#   - NET_BIND_SERVICE  (only if needed)
```

---

## 11. 🚀 Complete Production Setup

YAML

```
# docker-compose.prod.yml — Production setup
version: "3.9"

services:

  # ─── Nginx with SSL ─────────────────────────────────────────
  nginx:
    image: nginx:1.25-alpine
    container_name: prod_nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - certbot_conf:/etc/letsencrypt:ro
      - certbot_www:/var/www/certbot:ro
      - static_files:/var/www/static:ro
    depends_on:
      - app
    restart: always
    networks:
      - prod_network
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # ─── Certbot for SSL ────────────────────────────────────────
  certbot:
    image: certbot/certbot:latest
    volumes:
      - certbot_conf:/etc/letsencrypt
      - certbot_www:/var/www/certbot
    # Run certbot renewal check
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do certbot renew; sleep 12h & wait $${!}; done;'"
    networks:
      - prod_network

  # ─── FastAPI App ─────────────────────────────────────────────
  app:
    image: ${REGISTRY:-ghcr.io}/${REPO}:${TAG:-latest}
    container_name: prod_app
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      SECRET_KEY: ${SECRET_KEY}
      JWT_SECRET_KEY: ${JWT_SECRET_KEY}
      APP_ENV: production
      DEBUG: "false"
      WORKERS: "4"
    volumes:
      - uploads:/app/uploads
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: always
    networks:
      - prod_network
    security_opt:
      - no-new-privileges:true
    read_only: true
    tmpfs:
      - /tmp
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "5"
    deploy:
      resources:
        limits:
          cpus: "1.0"
          memory: 1G
        reservations:
          cpus: "0.25"
          memory: 256M

  # ─── PostgreSQL ──────────────────────────────────────────────
  db:
    image: postgres:16-alpine
    container_name: prod_db
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
      PGDATA: /var/lib/postgresql/data/pgdata
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./postgres/postgresql.conf:/etc/postgresql/postgresql.conf:ro
    command: postgres -c config_file=/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    restart: always
    networks:
      - prod_network
    # Don't expose DB port to internet!
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"

  # ─── Redis ───────────────────────────────────────────────────
  redis:
    image: redis:7-alpine
    container_name: prod_redis
    command: >
      redis-server
      --requirepass ${REDIS_PASSWORD}
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --save 60 1000
      --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--pass", "${REDIS_PASSWORD}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: always
    networks:
      - prod_network

  # ─── Celery Worker ───────────────────────────────────────────
  worker:
    image: ${REGISTRY:-ghcr.io}/${REPO}:${TAG:-latest}
    container_name: prod_worker
    command: celery -A app.tasks worker --loglevel=info --concurrency=4
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      APP_ENV: production
    depends_on:
      - db
      - redis
    restart: always
    networks:
      - prod_network

volumes:
  postgres_data:
  redis_data:
  uploads:
  certbot_conf:
  certbot_www:
  static_files:

networks:
  prod_network:
    driver: bridge
```

nginx

```
# nginx/conf.d/myapp.conf — Nginx for Docker deployment

upstream app {
    server app:8000;    # "app" = service name in compose
    keepalive 32;
}

server {
    listen 80;
    server_name myapp.com www.myapp.com;

    # Let's Encrypt challenge
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl http2;
    server_name myapp.com www.myapp.com;

    ssl_certificate /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;

    add_header Strict-Transport-Security "max-age=63072000" always;

    client_max_body_size 20M;

    # Static files served directly by Nginx
    location /static/ {
        root /var/www;
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # API traffic → FastAPI
    location / {
        proxy_pass http://app;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_read_timeout 60s;
    }
}
```

---

## 12. 🔄 Docker in CI/CD

Bash

```
# ─────────────────────────────────────────
# BUILD, TAG, AND PUSH TO REGISTRY
# ─────────────────────────────────────────

# Login to GitHub Container Registry
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Build with multiple tags
docker build \
    -t ghcr.io/myorg/myapp:latest \
    -t ghcr.io/myorg/myapp:v1.2.3 \
    -t ghcr.io/myorg/myapp:$(git rev-parse --short HEAD) \
    .

# Push all tags
docker push ghcr.io/myorg/myapp:latest
docker push ghcr.io/myorg/myapp:v1.2.3
docker push ghcr.io/myorg/myapp:$(git rev-parse --short HEAD)

# Or push all at once with buildx
docker buildx build \
    --platform linux/amd64,linux/arm64 \
    --tag ghcr.io/myorg/myapp:latest \
    --push \
    .

# ─────────────────────────────────────────
# DEPLOY ON SERVER
# ─────────────────────────────────────────

# Pull new image
docker compose pull app

# Deploy with zero downtime
docker compose up -d --no-deps app

# --no-deps = don't recreate dependencies (db, redis)
# This replaces only the app container

# Wait for health check
sleep 30
docker compose ps app
# Should show "healthy"

# Rollback if needed
docker compose pull app  # pulls previous version
docker compose up -d --no-deps app
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│                   DOCKER ESSENTIALS                            │
│                                                                │
│  DOCKERFILE:                                                   │
│  FROM base → RUN install → COPY code → CMD start             │
│  Multi-stage: builder → production (smaller image)            │
│  .dockerignore: exclude .env, venv, __pycache__              │
│                                                                │
│  KEY COMMANDS:                                                 │
│  docker build -t name:tag .          ← build image           │
│  docker run -d -p 8000:8000 image    ← run container         │
│  docker logs -f container            ← view logs             │
│  docker exec -it container bash      ← shell inside          │
│  docker compose up -d                ← start all services    │
│  docker compose down                 ← stop all              │
│  docker system prune                 ← cleanup              │
│                                                                │
│  LAYER CACHING:                                                │
│  Copy requirements.txt BEFORE copying code                   │
│  Requirements change rarely → fast rebuilds                  │
│  Code changes often → only last layers rebuild               │
│                                                                │
│  DOCKER COMPOSE:                                               │
│  db, redis, app, worker, nginx in one YAML                   │
│  depends_on + healthcheck = proper startup order             │
│  Named volumes = persistent data for DB                      │
│  Networks = containers talk by service name                   │
│                                                                │
│  SECURITY:                                                     │
│  Non-root user in container                                   │
│  No secrets in Dockerfile (use env vars at runtime)          │
│  Specific base image versions (not :latest)                  │
│  .dockerignore to exclude .env files                         │
│  read_only: true where possible                              │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the difference between an image and a container?
2.  What is a Dockerfile and what does each instruction do?
3.  Why do you COPY requirements.txt before COPY . . ?
4.  What is a multi-stage build and why use it?
5.  What is .dockerignore and what should it contain?
6.  What does -p 8080:8000 mean?
7.  What is the difference between CMD and ENTRYPOINT?
8.  What is a Docker volume and why do databases need them?
9.  What is docker compose and when would you use it?
10. What does depends_on with condition: service_healthy do?
11. How do containers communicate by name in Docker Compose?
12. What does docker compose down -v do? Why is it dangerous?
13. Why should you run containers as non-root?
14. What is docker system prune?
```

---

## 🛠️ Practice Exercises

Bash

```
# Exercise 1: Containerize Your FastAPI App
# Take your Phase 4.2 FastAPI project and:
# a) Write a Dockerfile (multi-stage)
# b) Write a .dockerignore
# c) Build the image
# d) Run it with docker run, test /health
# e) Add a HEALTHCHECK directive
# f) Verify image size < 300MB
# g) Verify app runs as non-root (docker exec app whoami)

# Exercise 2: Docker Compose Full Stack
# Create docker-compose.yml with:
# - Your FastAPI app
# - PostgreSQL 16
# - Redis 7
# - Nginx reverse proxy
# Requirements:
# - App doesn't start until DB is healthy
# - PostgreSQL data persists (named volume)
# - Can access app at http://localhost
# - Nginx serves static files directly
# - docker compose down → data preserved
# - docker compose down -v → data gone

# Exercise 3: Development vs Production
# Create two compose override files:
# docker-compose.dev.yml:
#   - Hot reload (mount code)
#   - Expose all ports for debugging
#   - Include pgAdmin at port 5050
#   - Include Mailhog for email testing
# docker-compose.prod.yml:
#   - 4 uvicorn workers
#   - No exposed database ports
#   - Resource limits (CPU, memory)
#   - read_only: true on app
# Test both work correctly

# Exercise 4: Registry Push
# a) Create a GitHub account and GitHub Container Registry
# b) Build your image with proper tags:
#    ghcr.io/yourusername/yourapp:latest
#    ghcr.io/yourusername/yourapp:1.0.0
#    ghcr.io/yourusername/yourapp:sha-abc123
# c) Push to the registry
# d) Pull it on a different machine/directory
# e) Run from the registry image (no local build)

# Exercise 5: Docker Troubleshooting
# Debug these intentionally broken scenarios:
# a) Container exits immediately — how do you find why?
# b) App can't connect to database — how do you debug?
# c) Port 8000 already in use on host — how do you resolve?
# d) Container runs out of memory — how do you see this?
# e) Image is 2GB — how do you reduce it?
# For each: show the commands you use to diagnose and fix
```

---

## ✅ Phase 7.2 Complete!

**You now know:**

text

```
✅ Docker concepts (image, container, volume, network)
✅ Writing Dockerfiles with best practices
✅ Layer caching optimization
✅ Multi-stage builds for smaller images
✅ .dockerignore
✅ docker run, exec, logs, inspect commands
✅ Docker Compose for multi-container apps
✅ depends_on with health checks
✅ Named volumes for persistent data
✅ Docker networking (service name DNS)
✅ Multiple compose files (dev/prod)
✅ Security best practices
✅ Registry push/pull workflow
✅ Production-ready compose setup with Nginx
✅ Docker in CI/CD pipelines
```