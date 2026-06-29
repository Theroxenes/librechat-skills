# LibreChat Agent Skills & Prompts

A collection of **skills** and **system prompts** for Qwen3.6:27b agents running
in **LibreChat 0.8.7**.

This README is the authoring reference for the two meta-agents that maintain this
repository. Read it before generating or editing any artifact here.

- **System Prompt Architect** — [`prompts/promptwriter/promptwriter.md`](prompts/promptwriter/promptwriter.md) — writes and optimizes system prompts.
- **Skill Creator** — [`prompts/skillwriter/skillwriter.md`](prompts/skillwriter/skillwriter.md) — designs and iterates `SKILL.md` skills.

## Design goals (priority order)

1. **Accuracy** — every concrete claim (tool params, config keys, API surfaces,
   versions) must be verified, not recalled. The [`verify-before-assuming`](skills/verify-before-assuming/SKILL.md)
   skill governs this and is always-apply.
2. **Compactness** — dense and high-signal, no filler. System prompts target
   ~800-1500 tokens.
3. **Reliability** — structured so a mid-size local model parses and follows it
   consistently.

## Repository layout

```
prompts/
  coding-agents/   coding / repo agents (e.g. therocode.md)
  deepresearch/    orchestrator + subagents (+ README.md for LibreChat setup)
  promptwriter/    System Prompt Architect
  skillwriter/     Skill Creator
skills/
  <skill-name>/SKILL.md   (+ optional references/, scripts/, assets/)
```

## Target environment — LibreChat 0.8.7

Authoring must match how this runtime actually behaves (verified, not assumed):

- **Model:** Qwen3.6:27b (Ollama). Optimize for it — `##` headers as section
  boundaries; bullets over numbered lists for non-sequential rules; state the
  response language if non-English; give an explicit tool-call format when tools
  exist; avoid nested JSON, use flat key-value or delimited strings.
- **Tools take a query string**, not bespoke parameters. Never invent flags like
  `news=true` or `after:` — put operators and recency cues in the query text.
- **web_search** is an integrated pipeline: search provider → **Firecrawl**
  scrape → rerank. It returns full page content and accepts a direct URL. Ground
  claims in the returned text, not snippets.
- **Skills** are a `SKILL.md` bundle; LibreChat stores and resolves bundled
  `references/`, `scripts/`, and `assets/` when the skill is active. `scripts/`
  run only where code execution is enabled — otherwise treat them as reference
  code.
- **Multi-agent** uses the **Subagents** feature: a parent spawns a child agent
  via a `subagent` tool call (isolated context, compact result). Allowed children
  are set in the Agent Builder picker / `subagents.agent_ids` (max 10). See
  [`prompts/deepresearch/README.md`](prompts/deepresearch/README.md).
- **GitHub MCP** and **Firecrawl** are configured in this deployment.

## Writing conventions (both agents)

- Imperative voice; one constraint per line.
- Explain the *why* behind a rule instead of rigid all-caps MUST/NEVER — capable
  models follow reasoning better than commands.
- Generalize; don't overfit instructions to specific examples.
- Cite official docs URLs for domain facts; verify version-specific config keys
  before recommending them.
- `SKILL.md` frontmatter: `name` (kebab-case) + `description` (what it does AND
  when to trigger — slightly "pushy" but precise enough to avoid false triggers).
  Keep the body under ~500 lines; offload large material to `references/`.

## Current inventory

**Skills:** discord-py-dev · docker-deploy · foundry-vtt-developer ·
librechat-tuner (+ `references/config-schema.md`) · pathfinder-2e-rules ·
pf2e-system · verify-before-assuming

**Prompts:** therocode (repo analysis) · deepresearch (orchestrator + 4
subagents) · promptwriter (System Prompt Architect) · skillwriter (Skill Creator)
