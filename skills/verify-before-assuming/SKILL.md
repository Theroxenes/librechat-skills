---
name: verify-before-assuming
description: "Use when about to state a fact, API detail, version number, config key, or any concrete claim not yet established in conversation. Triggers: first-time mentions of specific APIs, library versions, CLI flags, config paths, error codes, or any hallucinable detail. Do NOT trigger for general knowledge already confirmed in-context."
always-apply: true
---

# Verify Before Assuming

## Core Rule

Verify any concrete fact not yet established in conversation via other skill definitions, `web_search` or `get_file_contents`. Never rely on training memory for specifics.

## What Counts as an "Unknown"

Verify: API surfaces (methods, params, return types), versions/compatibility, config keys/env vars, CLI flags/syntax, error messages/codes, file paths, numbers (limits, timeouts, pricing), behavior (feature existence, defaults).

## What Does NOT Need Verification

Skip: facts already confirmed in-context, user-provided info, trivially universal knowledge, reasoning/opinions without factual claims.

## Verification Workflow

1. Identify the claim. 2. Check if already established in context → proceed if yes. 3. Verify via `web_search` (external) or `get_file_contents` (repo files). 4. Cite what you found, not what you remembered. 5. If unverifiable, say so — never guess.

## Examples

**Do verify:** tool parameter names, version EOL dates, env var names → check schema/code/docs.

**Don't verify:** user-provided info, facts already in context, universal programming concepts.

Prefer "Let me check X before answering" over "I believe..." followed by guessing.
