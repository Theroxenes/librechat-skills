# Deep Research Agent — LibreChat 0.8.7 Setup

An orchestrator agent that decomposes a research question, dispatches focused
sub-questions to specialized subagents, and synthesizes their findings into one
cited answer.

## How it maps onto LibreChat

LibreChat 0.8.7 has two multi-agent primitives. This design uses **Subagents**,
not Agent Chain:

| Primitive | What it does | Fit |
|-----------|--------------|-----|
| **Subagents** | Parent spawns a child agent as a *tool call*; child runs in an isolated context and returns a compact result | ✅ matches the orchestrator → sub-question → summary loop |
| Agent Chain | Fixed Mixture-of-Agents graph; each layer sees the previous layer's output | ✗ no dynamic dispatch or per-question routing |

Mechanism: when subagents are enabled, the parent receives a `subagent` tool.
Its reasoning loop calls that tool with a target agent + task; the child runs
isolated and returns its result as an expandable activity part. This is exactly
what `orchestrator.md` assumes ("dispatch sub-agents… never forward raw
outputs").

## Components

| File | Agent role | Tools it needs |
|------|-----------|----------------|
| `orchestrator.md` | Parent: decompose, dispatch, synthesize, verify | Subagents (no web search required) |
| `search_agent.md` | Precise factual lookup | Web Search |
| `web_search_agent.md` | Broad landscape / multi-angle exploration | Web Search |
| `news_agent.md` | Recent developments | Web Search |
| `code_search_agent.md` | Code / technical lookup | GitHub MCP (`search_code`) |

## Prerequisites

- LibreChat **0.8.7**.
- `endpoints.agents.capabilities` includes `subagents` and `web_search` — both
  are on by default. Only set this block if you've customized capabilities:
  ```yaml
  # librechat.yaml
  endpoints:
    agents:
      capabilities:
        - web_search
        - subagents
        - tools
        - actions
        # ...keep the rest of the defaults you want
      recursionLimit: 25      # default; raise if the orchestrator runs long
  ```
- Web search configured (provider keys in `.env` + `webSearch:` in
  `librechat.yaml`) with **Firecrawl** as `scraperProvider` — `web_search` then
  returns full scraped page text, not just snippets, and accepts a direct URL to
  read a source in full. See the `librechat-tuner` skill / `config-schema.md`.
- For `code_search_agent`: a **GitHub MCP** server configured under `mcpServers`
  exposing `search_code`. Skip this agent if you don't need code lookups.

## Step 1 — Create the four subagents

For each of `search_agent`, `web_search_agent`, `news_agent`,
`code_search_agent`:

1. **Agent Builder → New Agent.** Name it to match the role.
2. Paste the matching `prompts/deepresearch/<name>.md` into **Instructions**.
3. Select your model (e.g. `qwen3.6:27b`).
4. **Tools:** enable **Web Search** for the three search/news agents; for
   `code_search_agent` enable the GitHub MCP tools instead.
5. Save. Each agent must be **visible to your user** — the orchestrator can only
   spawn agents you're allowed to see (LibreChat rejects unauthorized
   `agent_ids`).

## Step 2 — Create the orchestrator

1. **Agent Builder → New Agent** → "Deep Research Orchestrator".
2. Paste `orchestrator.md` into **Instructions**; select the same model.
3. **Advanced Settings → Enable subagents** → ON.
   - **Allow self-spawn:** OFF (not needed here).
   - **Additional subagents:** add the four agents from Step 1.
4. Save.

The picker writes an `agent_ids` list on the parent. Equivalent config:

```yaml
subagents:
  enabled: true
  allowSelf: false
  agent_ids:        # the four agents' IDs (max 10)
    - <search_agent-id>
    - <web_search_agent-id>
    - <news_agent-id>
    - <code_search_agent-id>
```

## Step 3 (optional) — Expose as a preset

To make it selectable from the model menu, add a `modelSpecs.list` entry
pointing at the orchestrator agent. v0.8.7 supports model-spec subagents, so the
`subagents` block above can also live on a modelSpec. See `config-schema.md` →
`modelSpecs`.

## Operational notes & caveats

- **recursionLimit budget.** One run = decompose + N dispatches + synthesis,
  and each `subagent` call is ≥1 step. The orchestrator caps itself at 6
  dispatches, so the default `recursionLimit: 25` is comfortable; lower it only
  deliberately.
- **Parallel dispatch is best-effort.** "Dispatch in parallel" depends on the
  model emitting multiple `subagent` tool calls in one step. A 27B model often
  serializes them — still correct, just slower. Don't rely on true concurrency.
- **`agent_ids` cap is 10.** Four here leaves headroom for more roles.
- **Subagents don't inherit parent context.** Each child starts fresh, so the
  orchestrator must pass a self-contained sub-question in the dispatch — which
  the prompt already does.

## Tool-use notes

- `news_agent.md` uses query-text recency cues (year, "latest") rather than
  unsupported `news=true` / `after:` parameters — LibreChat web search takes a
  query string, not date filters.
- `code_search_agent.md` `search_code` is provided by the configured GitHub MCP
  server.

## Role coverage assessment

**Covered well:** planning + synthesis (orchestrator), precise facts
(search_agent), broad exploration (web_search_agent), recency (news_agent), and
code/technical (code_search_agent). Cross-validation and contradiction handling
are built into the orchestrator. This is a sound core for a deep-research stack.

**Earlier gaps, now addressed:**

1. **Clarify-first** — `orchestrator.md` now asks 2-3 scoping questions when a
   request is underspecified before dispatching.
2. **Beyond snippets** — with Firecrawl as the scraper, `web_search` returns
   full page text; `orchestrator.md` and the search agents are directed to
   ground findings in that content (and to pass URLs to `web_search` to read a
   source in full) rather than relying on titles/snippets.
