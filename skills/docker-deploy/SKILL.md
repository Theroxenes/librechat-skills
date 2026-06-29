---
name: docker-deploy
description: >
  Develop, deploy, and troubleshoot Docker containers for homelab/self-hosted
  services. Use when the user mentions Docker Compose, Portainer stacks,
  container deployment, volume mounts, health checks, container logs, or
  "how do I run X in Docker". Also triggers for "my container won't start",
  "compose file help", "update my containers", or any Docker/Portainer task.
  Always prefer official/well-maintained Docker Hub images. Produce
  Portainer-compatible compose files (no .env — all values inline).
---

# Docker Deploy (Portainer-Compatible)

## Core Principles

All compose files must be Portainer-compatible: no `${VAR}` interpolation, no `env_file:`, all values inline. Prefer official Docker Hub images — verify with Github MCP or `web_search` before recommending. Include restart policies, health checks, explicit port mappings. Warn before exposing database ports.

## Compose File Standards

### Template

Every compose file should include:

```yaml
services:
  service-name:
    image: registry/image:tag          # Pinned tag, never "latest" unless user requests
    container_name: service-name        # Explicit name for easy log/stop access
    restart: unless-stopped            # Default policy; adjust per-service
    ports:
      - "host:container"              # Explicit port mappings
    volumes:
      - ./data:/path/in/container     # Named or bind mounts as appropriate
```

### Portainer Compatibility Rules

- No `${ENV_VAR}` interpolation, no `env_file:`, no `secrets:`/`configs:`
- Use `environment:` key for inline env vars
- Pin image tags (never `latest` unless requested)

### Portainer Environments (Optional Enhancement)

For sensitive values, suggest Portainer's **Environments** feature (Stack → Edit → Environments tab). Still provide a working inline version by default.

### Volume Mount Conventions

```yaml
volumes:
  - service-data:/var/lib/data
  - ./config:/etc/config          # Bind mount for user-editable config files
  - /etc/localtime:/etc/localtime:ro  # Timezone sync
```

Named volumes for databases, bind mounts for configs.

### Network Conventions

- Default bridge for single-service; explicit network for multi-service:

```yaml
networks:
  app-net:
    driver: bridge

services:
  web:
    networks:
      - app-net
  db:
    networks:
      - app-net
```

### Health Checks

For services with HTTP endpoints or health commands:

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost:8080/"]
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

## Workflow

### 1. Understand the Service

Ask: what service, any special requirements (persistence, networking, reverse proxy), and target architecture (verify image supports it).

### 2. Research the Image

Use `web_search` to:
- Find official/recommended Docker Hub image, check pull count/activity
- Read image README for required env vars, ports, volumes, latest stable tag

### 3. Produce the Compose File

Generate a complete `docker-compose.yml` with:
- Pinned tag, inline env vars, port mappings, volume mounts, restart policy, health check, inline comments

### 4. Deployment Instructions

**Portainer:** Stacks → Add Stack → name it → Web Editor → paste YAML → Deploy.
**CLI:** `mkdir svc && cd svc` → save as `docker-compose.yml` → `docker compose up -d`.

### 5. Post-Deployment Checklist

Remind the user to:
- Check logs (`docker logs <name>`), verify reachability on mapped port, test persistence by restarting

## Troubleshooting Workflow

When a container fails or misbehaves:

1. **Status:** `docker ps -a | grep <name>`, `docker logs <name>`, `docker inspect <name>`

2. **Common failures:**
- Exits immediately → missing required env vars (check image README)
- Port in use → change host port mapping
- Permission denied on volume → `chmod`/`chown` host dir, check UID/GID env vars
- Unreachable → firewall, wrong port, or startup error in logs
- High CPU/memory → add `deploy.resources.limits`

3. **Resource limits** (optional):

```yaml
services:
  service-name:
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          memory: 128M
```

Works in Compose v2 / Portainer stack mode.

4. **Update:** Edit compose → change image tag → redeploy → check logs.

## Image Verification Checklist

Before recommending an image, verify:
- Official source or well-known maintainer, recent activity (<6mo), high pull count, architecture support matches user's platform, clear README

Show complete copy-paste-ready compose files. Inline comments for non-obvious settings. Warn about exposed ports and default passwords.
