# LibreChat Config Schema Cheat Sheet

Based on `librechat.example.yaml` from the LibreChat main branch (as of mid-2025).
Version field in example YAML: `1.3.13`.

**Always verify keys against the user's LibreChat version before recommending them.**

---

## Top-Level Keys

| Key | Type | Purpose |
|-----|------|---------|
| `version` | string (required) | Config schema version, e.g. `"1.3.13"` |
| `cache` | boolean | Enable response caching |
| `fileStrategy` | string or object | File storage strategy (`local`, `s3`, `firebase`, `azure_blob`, `cloudfront`) |
| `cloudfront` | object | CloudFront CDN settings (used with `fileStrategy: cloudfront`) |
| `skillSync` | object | GitHub skill sync configuration |
| `interface` | object | UI behavior and feature toggles |
| `turnstile` | object | Cloudflare Turnstile CAPTCHA |
| `registration` | object | Social logins, domain restrictions |
| `balance` | object | User balance / credits system |
| `transactions` | object | Transaction record persistence |
| `speech` | object | TTS / STT configuration |
| `rateLimits` | object | Rate limiting for uploads and imports |
| `actions` | object | Agent Actions SSRF domain restrictions |
| `endpoints` | object | AI provider endpoint definitions |
| `mcpServers` | object | MCP server definitions |
| `mcpSettings` | object | MCP SSRF domain restrictions |
| `modelSpecs` | object | Model presets and grouping |
| `fileConfig` | object | Per-endpoint file upload limits |
| `webSearch` | object | Web search provider configuration |
| `memory` | object | User memory / personalization settings |
| `messageFilter` | object | PII / credential filtering on user messages |

---

## `interface` — UI Feature Toggles

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `customWelcome` | string | — | Welcome message shown to users |
| `fileSearch` | boolean | `true` | Enable file search in chat area |
| `privacyPolicy.externalUrl` | string | — | Link to privacy policy |
| `privacyPolicy.openNewTab` | boolean | `true` | Open privacy policy in new tab |
| `termsOfService.externalUrl` | string | — | Link to ToS |
| `termsOfService.modalAcceptance` | boolean | `false` | Force modal acceptance of ToS |
| `modelSelect` | boolean/string | `true` | Show model selector (`true`, `false`, `"header"`) |
| `parameters` | boolean | `true` | Show per-message parameter controls |
| `presets` | boolean | `true` | Show preset management |
| `prompts.use` | boolean | `true` | Allow using prompts |
| `prompts.create` | boolean | `true` | Allow creating prompts |
| `prompts.share` | boolean | `false` | Allow sharing prompts |
| `prompts.public` | boolean | `false` | Allow public prompts |
| `bookmarks` | boolean | `true` | Enable conversation bookmarks |
| `multiConvo` | boolean | `true` | Enable multi-conversation tabs |
| `agents.use` | boolean | `true` | Allow using agents |
| `agents.create` | boolean | `true` | Allow creating agents |
| `agents.share` | boolean | `false` | Allow sharing agents |
| `agents.public` | boolean | `false` | Allow public agents |
| `peoplePicker.users` | boolean | `true` | Show users in people picker |
| `peoplePicker.groups` | boolean | `true` | Show groups in people picker |
| `peoplePicker.roles` | boolean | `true` | Show roles in people picker |
| `marketplace.use` | boolean | `false` | Enable marketplace |
| `fileCitations` | boolean | `true` | Show file citations in responses |
| `defaultPinnedTools` | array | — | Tools pinned by default: `artifacts`, `execute_code`, `web_search`, `file_search`, `skills`, `mcp` |
| `remoteAgents.use` | boolean | `false` | Allow remote agents with external API support |
| `sharedLinks.create` | boolean | — | Allow creating shared links |
| `sharedLinks.share` | boolean | — | Allow sharing links |
| `sharedLinks.public` | boolean | — | Allow public shared links |
| `sharedLinks.snapshotFiles` | boolean | `true` | Snapshot files for shared chats |
| `mcpServers.use` | boolean | — | Allow using MCP servers |
| `mcpServers.create` | boolean | — | Allow creating MCP servers |
| `mcpServers.share` | boolean | — | Allow sharing MCP servers |
| `mcpServers.public` | boolean | — | Allow public MCP servers |
| `temporaryChatRetention` | number | `720` | Temp chat retention in hours (1–8760) |
| `retentionMode` | string | `"temporary"` | `"temporary"` or `"all"` |

---

## `endpoints.custom[]` — Custom Provider Endpoints

Each entry is an object with these keys:

| Key | Type | Description |
|-----|------|-------------|
| `name` | string (required) | Unique endpoint name |
| `provider` | string | Provider type: `openai` (default), `anthropic`, `google`, etc. |
| `apiKey` | string | API key; use `${ENV_VAR}` for env var substitution |
| `baseURL` | string | API base URL |
| `headers` | object | Custom headers forwarded on every request |
| `models.default` | array | Default model list |
| `models.fetch` | boolean | Fetch models from provider API at runtime |
| `titleConvo` | boolean | Enable auto-title generation |
| `titleModel` | string | Model to use for conversation titles |
| `titleMethod` | string | `"completion"` or `"functions"` |
| `summarize` | boolean | Enable response summarization |
| `summaryModel` | string | Model for summarization |
| `modelDisplayLabel` | string | Label shown in UI |
| `iconURL` | string | Custom icon URL for the endpoint |
| `addParams` | object | Extra params to add to every request |
| `dropParams` | array | Param keys to remove from requests |

### Built-in Endpoint Overrides

Under `endpoints.` you can also override built-in providers:

- `endpoints.openAI.headers` — custom headers for OpenAI
- `endpoints.google.headers` — custom headers for Google
- `endpoints.anthropic.streamRate` — stream rate limit in ms
- `endpoints.anthropic.titleModel` — title model for Anthropic
- `endpoints.anthropic.vertex.*` — Vertex AI configuration (region, service key, model mappings)
- `endpoints.bedrock.models` — AWS Bedrock model list
- `endpoints.bedrock.inferenceProfiles` — Bedrock inference profile ARNs
- `endpoints.bedrock.guardrailConfig` — Bedrock guardrails
- `endpoints.assistants.disableBuilder` — disable Assistants builder UI
- `endpoints.assistants.supportedIds` / `excludedIds` — filter assistants
- `endpoints.assistants.capabilities` — available assistant capabilities
- `endpoints.agents.recursionLimit` — default agent recursion depth
- `endpoints.agents.maxRecursionLimit` — max agent recursion depth
- `endpoints.agents.disableBuilder` — disable Agents builder UI
- `endpoints.agents.titleTiming` — `"immediate"` or `"final"`
- `endpoints.agents.capabilities` — available agent capabilities

---

## `modelSpecs` — Model Presets

```yaml
modelSpecs:
  list:
    - name: "spec-name"
      label: "Display Name"
      description: "Optional description"
      default: false        # Hard admin default
      softDefault: false    # First-time user default only
      group: "groupName"    # Endpoint name or custom string
      groupIcon: "groq"     # Built-in icon key or URL
      hideBadgeRow: false   # Hide tool badge row
      skills: true          # Enable skills (true/false/name list)
      subagents:            # Subagent configuration
        enabled: false
        allowSelf: false
        agent_ids: []
      preset:               # Model parameters for this spec
        endpoint: "openAI"
        model: "gpt-4o"
        instructions: "..."
        temperature: 0.7
```

---

## `mcpServers` — MCP Server Definitions

```yaml
mcpServers:
  server-name:
    type: sse              # "sse" (default) or "stdio"
    url: "http://..."      # For SSE/HTTP transports
    command: npx           # For stdio transport
    args: [...]            # For stdio transport
    timeout: 60000         # Timeout in ms (default: 60000)
    iconPath: "/path/to/icon.svg"
    proxy: "${MCP_PROXY_URL}"  # Optional outbound proxy
```

---

## `customParams` — Default Model Parameters

```yaml
customParams:
  defaultParamsEndpoint:
    openai:
      temperature: 0.7
      top_p: 1
      max_tokens: 4096
    anthropic:
      temperature: 0.5
      max_tokens: 4096
```

Per-endpoint params override these defaults. Per-modelSpec `preset` values override endpoint defaults.

---

## `fileConfig` — File Upload Limits

```yaml
fileConfig:
  endpoints:
    assistants:
      fileLimit: 5
      fileSizeLimit: 10        # MB per file
      totalSizeLimit: 50       # MB total per request
      supportedMimeTypes: [...]
    default:
      totalSizeLimit: 20
  serverFileSizeLimit: 100     # Global max in MB
  avatarSizeLimit: 2           # Avatar max in MB
  imageGeneration:
    percentage: 100
    px: 1024
  clientImageResize:
    enabled: false
    maxWidth: 1900
    maxHeight: 1900
    quality: 0.92
```

---

## `webSearch` — Web Search Configuration

```yaml
webSearch:
  searchProvider: tavily       # "tavily", "serper", "searxng"
  scraperProvider: tavily      # "tavily", "firecrawl"
  jinaApiKey: '${JINA_API_KEY}'
  cohereApiKey: '${COHERE_API_KEY}'
  serperApiKey: '${SERPER_API_KEY}'
  searxngInstanceUrl: '${SEARXNG_INSTANCE_URL}'
  tavilyApiKey: '${TAVILY_API_KEY}'
  firecrawlApiKey: '${FIRECRAWL_API_KEY}'
```

---

## `memory` — User Memory / Personalization

```yaml
memory:
  disabled: false
  validKeys: ["preferences", "work_info", "personal_info"]
  tokenLimit: 10000
  maxInputTokens: 12000
  personalize: true
  agent:
    enabled: true
    id: "your-memory-agent-id"
```

---

## Key `.env` Variables

| Variable | Purpose |
|----------|---------|
| `OPENAI_API_KEY` | OpenAI API key |
| `ANTHROPIC_API_KEY` | Anthropic API key |
| `GOOGLE_GENERATIVE_AI_API_KEY` | Google Gemini key |
| `AZURE_OPENAI_API_KEY` | Azure OpenAI key |
| `GROQ_API_KEY` | Groq API key |
| `OLLAMA_BASE_URL` | Ollama instance URL |
| `MISTRAL_API_KEY` | Mistral AI key |
| `BEDROCK_AWS_ACCESS_KEY_ID` | AWS Bedrock access key |
| `BEDROCK_AWS_SECRET_ACCESS_KEY` | AWS Bedrock secret key |
| `BEDROCK_AWS_REGION` | AWS region for Bedrock |
| `JWT_SECRET` | JWT signing secret (required for auth) |
| `DOMAIN_CLIENT` | Client domain for CORS |
| `DOMAIN_SERVER` | Server domain |
| `MEILI_HOST` | Meilisearch host for search indexing |
| `CREDS_TTL` | Credentials token TTL in hours |
| `SESSION_SECRET` | Session signing secret |
| `TTS_API_KEY` | TTS provider API key |
| `STT_API_KEY` | STT provider API key |
| `SERPER_API_KEY` | Serper web search key |
| `JINA_API_KEY` | Jina reranking key |
| `TAVILY_API_KEY` | Tavily search/scraper key |
| `FIRECRAWL_API_KEY` | Firecrawl scraper key |

For the full `.env` reference, see: <https://www.librechat.ai/docs/configuration/dotenv>

---

## Version Compatibility Notes

| Feature | Minimum Version | Notes |
|---------|----------------|-------|
| `modelSpecs.groupIcon` | v0.7.x+ | Icon support for custom groups |
| `fileStrategy` as object (granular) | v0.8.x+ | Per-file-type storage strategies |
| `interface.mcpServers.*` | v0.8.x+ | MCP user permission toggles |
| `endpoints.agents.titleTiming` | v0.8.x+ | Agent title generation timing |
| `cloudfront.imageSigning: "cookies"` | v0.8.x+ | Signed cookie support |
| `skillSync.github.*` | v0.8.x+ | GitHub skill sync |
| `memory.agent.enabled` | v0.7.8+ | Automatic memory updates from chat |
| `interface.remoteAgents.*` | v0.8.x+ | Remote agents with external API |
| `messageFilter.pii.*` | v0.8.x+ | PII/credential message filtering |
| `endpoints.anthropic.vertex.*` | v0.7.8+ | Vertex AI for Anthropic models |
| `webSearch.tavilyScraperOptions` | v0.8.x+ | Tavily as scraper provider |

**Always verify against the user's actual version — this table is a guide, not authoritative.**

---

## Precedence Order (highest → lowest)

1. Per-message UI overrides (temperature slider, etc.)
2. `modelSpecs.preset` values
3. `customParams.defaultParamsEndpoint` values
4. Provider SDK defaults
