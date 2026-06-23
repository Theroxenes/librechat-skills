---
name: verify-before-assuming
description: "Use when the model is about to state a fact, API detail, version number, configuration key, or any other concrete claim that it has not already established in the current conversation. Triggers: first-time mentions of specific APIs, library versions, CLI flags, config paths, error codes, or any detail the model could be hallucinating. Do NOT trigger for general knowledge already confirmed in-context."
always-apply: true
---

# Verify Before Assuming

## Core Rule

Before stating any concrete fact that is not already established in the current conversation context, verify it first via search or code reference. Never rely on training memory for specifics.

## What Counts as an "Unknown"

Verify these before asserting them:

| Category | Examples |
|----------|----------|
| API surface | Method names, parameter shapes, return types, endpoint paths |
| Versions | Library versions, SDK compatibility matrices, deprecation dates |
| Configuration | Env var names, config key paths, default values |
| CLI / tooling | Flag names, subcommand syntax, exit codes |
| Error messages | Exact error text, status codes, exception classes |
| File locations | Path to a specific file in a repo, directory structure |
| Numbers | Limits, timeouts, thresholds, pricing, counts |
| Behavior | Whether feature X exists in version Y, default on/off state |

## What Does NOT Need Verification

- Facts already stated and confirmed earlier in the conversation
- User-provided information (the user told you directly)
- Trivially universal knowledge (e.g., "HTTP 200 means success")
- Reasoning steps, opinions, or suggestions that don't claim specific facts

## Verification Workflow

1. **Identify the claim** — what specific fact are you about to state?
2. **Check context** — was this already established in the conversation? If yes, proceed.
3. **Verify** — use one of:
   - `web_search` — for external facts, documentation, release notes
   - `search_code` — for code-level details (function signatures, config keys)
   - `get_file_contents` — for reading specific files in a known repo
4. **State the verified fact** — cite what you found, not what you remembered
5. **If unverifiable** — say so explicitly rather than guessing

## Examples

### Do verify:
- "The `create_branch_mcp_github-skills` tool accepts a `from_branch` parameter" → check the tool schema or code
- "Node 18 reached EOL in April 2025" → search for the official schedule
- "Set `DATABASE_URL` in `.env`" → read the actual config file or docs

### Don't verify:
- User says "my repo is at owner/repo" and you reference that later
- You already searched for the API shape two turns ago and it's in context
- Explaining that a `for` loop iterates over an array (universal programming concept)

## Tone

Be direct about uncertainty. Prefer:

> "Let me check the current API signature before I answer that."

Over:

> "I believe the parameter is called..." (then guessing)
