---
name: librechat-tuner
description: >
  Configure, tune, and troubleshoot LibreChat — the open-source LLM orchestrator
  and web UI. Use when the user mentions LibreChat configuration, librechat.yaml,
  model endpoints, modelSpecs, interface settings, MCP servers, agents, custom
  parameters, .env setup, Docker deployment, or any LibreChat tuning task. Also
  triggers for "my LibreChat isn't doing X", "how do I add a provider to
  LibreChat", "LibreChat YAML config help", or version-specific feature checks.
  Always enforce version compatibility: if the user states their LibreChat
  version, verify that the requested feature or config key exists in that
  release before recommending it.
---

# LibreChat Tuner

Help users configure and tune LibreChat — a self-hosted LLM orchestrator / web UI
that unifies multiple AI providers (OpenAI, Anthropic, Google, Azure, Ollama,
Groq, etc.) in one interface.

## Official References

- **Docs:** <https://www.librechat.ai/docs>
  - Configuration overview: `/docs/configuration`
  - librechat.yaml guide: `/docs/configuration/librechat_yaml`
  - Object structure (endpoints, modelSpecs, customParams, interface):
    `/docs/configuration/librechat_yaml/object_structure/`
  - Environment variables: `/docs/configuration/dotenv`
  - Custom endpoints quick start: `/docs/quick_start/custom_endpoints`
  - Agents: `/docs/features/agents`
  - MCP: `/docs/features/mcp`
- **GitHub:** <https://github.com/danny-avila/LibreChat>
  - Example config: `librechat.example.yaml`
  - Example env: `.env.example`
  - Releases / changelog: GitHub Releases page

## Version Compatibility — Enforce This

LibreChat evolves fast. Config keys, providers, and features change between
minor versions. **Always check version compatibility before recommending a
config option.**

1. Ask the user for their LibreChat version if not stated. Check via:
   - UI footer or Settings → About
   - `docker logs <container>` startup banner
   - `package.json` `"version"` field in the repo root
2. If the user states a version (e.g., "v0.8.6"), verify that the feature or
   config key exists in that release:
   - Search the LibreChat changelog at
     <https://www.librechat.ai/changelog/v{version}>
   - Check the GitHub releases page for the tagged version
   - Read `librechat.example.yaml` on the matching git tag/branch
3. If a feature was added **after** the user's version, tell them:
   - The minimum version that supports it
   - What to do instead on their current version (if anything)
4. If unsure whether a key exists in a given version, say so — never guess.

## Configuration Files Overview

LibreChat uses four main config surfaces:

| File | Purpose |
|------|---------|
| `.env` | Secrets, API keys, server-level feature flags |
| `librechat.yaml` | Custom endpoints, modelSpecs, interface settings, MCP servers, agents, customParams |
| `docker-compose.yml` / `docker-compose.override.yml` | Container orchestration, volume mounts |
| `config/filters.js` (optional) | Custom request/response filters |

## Workflow

### 1. Understand the Goal

Ask:
- What LibreChat version are you running?
- What are you trying to achieve? (add a provider, tune model params, change UI, set up agents, debug an error, etc.)
- How is LibreChat deployed? (Docker Compose, Docker Swarm, Kubernetes, bare metal)

### 2. Identify the Config Surface

| Task | Config surface |
|------|---------------|
| Add / switch AI provider | `librechat.yaml` → `endpoint` section |
| Set API keys | `.env` or `librechat.yaml` with `${ENV_VAR}` |
| Tune model parameters (temperature, top_p, max_tokens) | `librechat.yaml` → `customParams.defaultParamsEndpoint` or per-modelSpec |
| Create model presets / groups | `librechat.yaml` → `modelSpecs` |
| Change UI behavior (hide features, set defaults) | `librechat.yaml` → `interface` section |
| Add MCP servers | `librechat.yaml` → `mcpServers` |
| Configure agents | `librechat.yaml` → `agents` section |
| Server-level toggles (auth, file storage, rate limits) | `.env` variables |

### 3. Produce the Config Snippet

- Generate valid YAML for `librechat.yaml` or key-value pairs for `.env`.
- Always show the **full path** within the config hierarchy so the user knows
  where to paste it.
- If the change requires a restart, say so explicitly.
- If environment variables are involved, remind the user to restart the
  container / process after changing `.env`.

### 4. Verify Against Version

Before presenting the final answer:
- Confirm the config keys used exist in the user's version.
- If the user is on an older version and a newer feature would be better,
  mention it with the minimum required version.

## Common Tuning Tasks

### Adding a Custom Endpoint

```yaml
# librechat.yaml
endpoint:
  customProvider:
    # provider name — must match LibreChat's internal provider key
    # e.g., "openai", "anthropic", "google", "ollama", "groq"
    ...
```

Reference: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/custom_endpoint>

### Setting Default Model Parameters

```yaml
# librechat.yaml
customParams:
  defaultParamsEndpoint:
    openai:
      temperature: 0.7
      max_tokens: 4096
    anthropic:
      temperature: 0.5
```

Reference: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/custom_params>

### Creating Model Specs (Presets)

```yaml
# librechat.yaml
modelSpecs:
  include:
    - spec: ...
  exclude: []
  sort: ""
```

Reference: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/model_specs>

### Interface Customization

```yaml
# librechat.yaml
interface:
  modelSelect: "header"
  multiModelSend: false
  ...
```

### MCP Server Configuration

```yaml
# librechat.yaml
mcpServers:
  - name: "my-server"
    url: "http://localhost:3001/mcp"
    ...
```

Reference: <https://www.librechat.ai/docs/features/mcp>

## Troubleshooting Checklist

When the user reports an issue:

1. **Version mismatch** — is the config key available in their version?
2. **YAML syntax** — indentation errors are the #1 cause of silent failures.
3. **Volume mount** — is `librechat.yaml` actually mounted into the container?
   Check `docker inspect` or the compose volume mapping.
4. **Env var override** — `.env` values can shadow YAML settings.
5. **Restart needed** — config changes require a container / process restart.
6. **Logs** — check `docker logs <container>` for startup errors.

## Config Schema Cheat Sheet

Read `references/config-schema.md` when the user needs a detailed breakdown of
available config keys, their types, defaults, and version availability. That file
covers the full librechat.yaml schema surface plus key .env variables.

## Tone

Be direct and technical. Show config snippets the user can copy-paste. Always
cite the relevant LibreChat docs URL so the user can read further. Never
recommend a config key you haven't verified exists in their version.
