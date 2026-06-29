---
name: librechat-tuner
description: >
  Configure, tune, and troubleshoot LibreChat (open-source LLM orchestrator).
  Use when the user mentions LibreChat config, librechat.yaml, model endpoints,
  modelSpecs, interface settings, MCP servers, agents, custom params, .env, or
  Docker deployment. Also triggers for "my LibreChat isn't doing X", "how do I
  add a provider", "LibreChat YAML config help". Always enforce version
  compatibility before recommending config keys.
---

# LibreChat Tuner

## Official References

- Docs: <https://www.librechat.ai/docs> (config, librechat_yaml/object_structure/, dotenv, quick_start/custom_endpoints, features/agents, features/mcp)
- GitHub: <https://github.com/danny-avila/LibreChat> (`librechat.example.yaml`, `.env.example`, Releases)

## Version Compatibility — Enforce This

LibreChat evolves fast. Always verify config keys exist in the user's version before recommending.

1. Ask for version if not stated (UI footer, `docker logs`, or `package.json`).
2. Verify via changelog (<https://www.librechat.ai/changelog/v{version}>), GitHub releases, or `librechat.example.yaml` on matching tag.
3. If feature is newer than user's version: state minimum required version + alternative.
4. If unsure, say so — never guess.

## Configuration Files Overview

Four config surfaces:

| File | Purpose |
|------|---------|
| `.env` or Portainer's Environments feature | Secrets, API keys, server flags |
| `librechat.yaml` | Endpoints, modelSpecs, interface, MCP, agents, customParams |
| `docker-compose.yml` | Container orchestration, volumes |
| `config/filters.js` | Custom request/response filters (optional) |

## Workflow

### 1. Gather Requirements

Ask: version, goal (add provider, tune params, change UI, agents, debug), deployment method.

### 2. Identify the Config Surface

| Task | Config surface |
|------|---------------|
| Add / switch AI provider | `librechat.yaml` → `endpoints` section |
| Set API keys | `.env`/Portainer Environments or `librechat.yaml` with `${ENV_VAR}` |
| Tune model parameters (temperature, top_p, max_tokens) | `librechat.yaml` → `customParams.defaultParamsEndpoint` or per-modelSpec |
| Create model presets / groups | `librechat.yaml` → `modelSpecs` |
| Change UI behavior (hide features, set defaults) | `librechat.yaml` → `interface` section |
| Add MCP servers | `librechat.yaml` → `mcpServers` |
| Configure agents | `librechat.yaml` → `agents` section |
| Server-level toggles (auth, file storage, rate limits) | `.env` variables |

### 3. Produce the Config Snippet

Generate valid YAML or key-value pairs. Show full config path. State if restart needed.

### 4. Verify Against Version

Confirm keys exist in user's version. Mention newer features with minimum required version if applicable.

## Common Tuning Tasks

### Adding a Custom Endpoint

```yaml
# librechat.yaml
endpoints:
  custom:
    - name: "My Provider"
      apiKey: "${MY_API_KEY}"
      baseURL: "https://api.example.com/v1"
      models:
        default: ["model-a"]
        fetch: true
```
Ref: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/custom_endpoint>

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
Ref: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/custom_params>

### Creating Model Specs (Presets)

```yaml
# librechat.yaml
modelSpecs:
  list:
    - name: "spec-name"
      label: "Display Name"
      preset:
        endpoint: "openAI"
        model: "gpt-4o"
        temperature: 0.7
```
Ref: <https://www.librechat.ai/docs/configuration/librechat_yaml/object_structure/model_specs>

### Interface Customization

```yaml
# librechat.yaml
interface:
  modelSelect: "header"
  multiModelSend: false
  ...
```

### MCP Servers

```yaml
# librechat.yaml
mcpServers:
  my-server:                       # keyed by name — an object, not a list
    type: sse                      # "sse" (default) or "stdio"
    url: "http://localhost:3001/mcp"
```
Ref: <https://www.librechat.ai/docs/features/mcp>

## Troubleshooting Checklist

When the user reports an issue:

1. Version mismatch? 2. YAML indentation errors? 3. Volume mount correct (`docker inspect`)? 4. `.env` shadowing YAML? 5. Restart needed? 6. Check `docker logs`.

## Config Schema Cheat Sheet

Read `references/config-schema.md` for full schema: keys, types, defaults, version availability, key .env vars.

Always cite LibreChat docs URLs. Never recommend unverified config keys.
