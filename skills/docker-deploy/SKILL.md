---
name: docker-deploy
description: >
  Develop, deploy, and troubleshoot Docker containers — especially for homelab
  and self-hosted services. Use when the user mentions Docker Compose,
  docker-compose.yml, container deployment, Portainer stacks, Docker Hub image
  selection, container networking, volume mounts, health checks, container logs,
  or "how do I run X in Docker". Also triggers for "my container won't start",
  "compose file help", "Portainer stack from compose", "update my containers",
  or any Docker/Portainer troubleshooting task. Always prefer official or
  well-maintained images from Docker Hub and produce Portainer-compatible
  compose files (no .env files — all values inline). Make sure to use this
  skill whenever the user mentions containers, Docker, Portainer, stacks,
  docker-compose, or self-hosted service deployment, even if they don't
  explicitly say "Docker".
---

# Docker Deploy

Help users develop, deploy, and troubleshoot Docker containers for homelab and
self-hosted services. Prefer official or well-maintained images from Docker Hub.
All compose files must be Portainer-compatible: no `.env` files, all values
inline in the compose YAML.

## Core Principles

1. **Portainer-first** — Compose files must work when pasted into Portainer's
   "Web Editor" stack creation UI. No `.env` references (`${VAR}`), no external
   env_file directives. All config values go inline in the YAML.
2. **Docker Hub preferred** — Recommend official images or high-quality community
   images from Docker Hub. Verify image existence and pull counts before
   recommending. Use `web_search` to check Docker Hub pages when unsure.
3. **Pure compose** — Generate complete, self-contained `docker-compose.yml`
   files. No build steps unless the service genuinely requires it (rare for
   homelab services).
4. **Safety defaults** — Include restart policies, health checks where sensible,
   and explicit port mappings. Never expose database ports to host without
   warning.

## Compose File Standards

### Required Elements

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

| Rule | Reason |
|------|--------|
| No `${ENV_VAR}` interpolation | Portainer requires inline values or Portainer Environments |
| No `env_file:` directive | Portainer doesn't load external .env files in Web Editor mode |
| Use `environment:` key for env vars | Inline environment variables work everywhere |
| Avoid `secrets:` and `configs:` | Not supported in Portainer's basic stack editor |
| Pin image tags | Prevents surprise updates; user controls when to upgrade |

### Portainer Environments (Optional Enhancement)

If the user mentions sensitive values (passwords, API keys), suggest using
Portainer's built-in **Environments** feature instead of `.env` files:

1. In Portainer UI: Stack → Edit → Environments tab
2. Define variables there, reference in compose as `${VAR_NAME}`
3. This is the Portainer-native equivalent of `.env` and works seamlessly

Mention this option but still provide a working inline version by default.

### Volume Mount Conventions

```yaml
volumes:
  # Data persistence — use named volumes for databases, bind mounts for configs
  - service-data:/var/lib/data
  - ./config:/etc/config          # Bind mount for user-editable config files
  - /etc/localtime:/etc/localtime:ro  # Timezone sync

volumes:                          # Named volume declarations
  service-data:
```

### Network Conventions

- Default bridge network is fine for single-stack services.
- For multi-service stacks, define an explicit network:

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

Include health checks for services that expose HTTP endpoints or have a
health-check command:

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

Ask (if not stated):
- What service does the user want to deploy?
- Any specific requirements? (data persistence, networking, reverse proxy)
- Target architecture? (x86_64, ARM/ARM64 for Raspberry Pi etc.) — verify image
  supports the target platform.

### 2. Research the Image

Use `web_search` to:
1. Find the official or recommended Docker Hub image
2. Check pull count / star rating to gauge popularity and maintenance
3. Read the image's README for required environment variables, ports, volumes
4. Verify the latest stable tag

### 3. Produce the Compose File

Generate a complete `docker-compose.yml` with:
- Pinned image tag (not `latest`)
- All required environment variables inline
- Port mappings
- Volume mounts for data persistence
- Restart policy
- Health check if applicable
- Brief comments explaining non-obvious settings

### 4. Deployment Instructions

Provide deployment steps for both paths:

**Portainer UI:**
1. Portainer → Stacks → Add Stack
2. Give stack a name
3. Select "Web Editor" and paste the compose YAML
4. Click "Deploy the stack"

**CLI:**
```bash
mkdir my-service && cd my-service
# paste compose file into docker-compose.yml
docker compose up -d
```

### 5. Post-Deployment Checklist

Remind the user to:
- Check logs: Portainer → Stack → Containers → Logs, or `docker logs <name>`
- Verify the service is reachable on the mapped port
- Test data persistence by restarting the container

## Troubleshooting Workflow

When a container fails or misbehaves:

### Step 1: Check Container Status

```bash
docker ps -a | grep <service-name>   # Is it running, exited, restarting?
docker logs <container-name>         # Recent logs
docker inspect <container-name>      # Detailed state, mounts, network
```

In Portainer: Stacks → <stack> → Containers → click container → Logs / Inspect

### Step 2: Common Failure Modes

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| Container exits immediately | Missing required env vars or config | Check image README for mandatory variables |
| Port already in use | Another container or service on same port | Change host port mapping |
| Permission denied on volume | Wrong ownership/permissions on bind mount | `chmod` / `chown` the host directory, or check UID/GID env vars |
| Service unreachable | Firewall, wrong port, or health check failing | Verify port mapping, check logs for startup errors |
| High CPU / memory | Misconfigured worker count, missing limits | Add `deploy.resources.limits` or container-level `mem_limit` |

### Step 3: Resource Limits (Optional but Recommended)

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

Note: `deploy` keys work in Docker Compose v2 with Swarm or with
`docker compose` (v2). For Portainer, they are supported in stack mode.

### Step 4: Update Procedure

To update a service to a newer image tag:
1. Edit the compose file (Portainer → Stack → Edit Stack)
2. Change the image tag
3. Redeploy — Portainer will pull new image and recreate container
4. Check logs after redeploy

## Common Homelab Services Reference

Use these as starting points. Always verify current tags and config via
`web_search` before generating, as images evolve.

### Jellyfin (Media Server)

- Image: `jellyfin/jellyfin`
- Ports: 8096 (HTTP), 8920 (HTTPS optional)
- Key volumes: `/config`, `/media` (read-only)
- Requires hardware transcoding GPU passthrough for NVidia/AMD

### LibreChat (LLM Web UI)

- Image: `ghcr.io/danny-avila/librechat`
- Ports: 3080
- Key env vars: `OPENAI_API_KEY`, `DOMAIN`, `HOST`, `PORT`
- Config via mounted `librechat.yaml` and inline env vars
- See `librechat-tuner` skill for deep config guidance

### Gluetun (VPN Container)

- Image: `qmcgaw/gluetun`
- Acts as VPN gateway — other containers use it via `network_mode` or DNS
- Key env vars: `VPN_SERVICE_PROVIDER`, `VPN_TYPE`, `OPENVPN_USER`, `OPENVPN_PASSWORD`
- Supports WireGuard and OpenVPN
- Exposes DNS on port 8053 for client containers

### SearXNG (Metasearch Engine)

- Image: `alandoyle/searxng` or `searxng/searxng`
- Ports: 8080
- Key env vars: `SECRET_KEY`, `BASE_URL`
- Requires `settings.yml` and `limiter.toml` for production use
- Often paired with SearXNG frontend container

## Image Verification Checklist

Before recommending an image, verify:

1. **Source** — Official Docker Hub badge or well-known maintainer
2. **Activity** — Recent builds/updates (within 6 months for active projects)
3. **Pull count** — Higher is generally better (indicates community trust)
4. **Architecture support** — Does it support the user's platform (x86_64, ARM64)?
5. **Documentation** — README with env vars, ports, volumes clearly listed

## Tone

Be direct and practical. Show complete compose files the user can copy-paste
into Portainer or save as `docker-compose.yml`. Always explain non-obvious
settings with inline comments. Warn about security implications (exposed
ports, default passwords) without being alarmist.
