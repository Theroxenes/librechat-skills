---
name: prompt-forger
description: >
  Generate or optimize compact, high-signal system prompts for LLM agents
  (coding, research, etc.). Use when the user asks to write a system prompt,
  craft an agent persona, design instructions for a local LLM (Qwen, Llama,
  Mistral), shorten/compact a system prompt, or reduce its token count.
---

# Prompt Forger

Generate compact, high-signal system prompts optimized for mid-size local models (Qwen 3.6 27b and similar).

## Design Principles

1. **Density over verbosity** — every line must constrain behavior or provide actionable guidance. No pleasantries, no filler identity statements.
2. **Imperative voice** — use direct commands ("Do X", "Never Y") rather than descriptions of what the model "is".
3. **Structure with headers and tables** — mid-size models parse structured formats more reliably than prose paragraphs.
4. **Front-load critical constraints** — put safety rails, output format rules, and tool-use policies in the first 20 lines.
5. **Target ~800-1500 tokens** for the final prompt unless the user specifies otherwise.

## Generation Workflow

### 1. Gather Requirements

Ask the user (or infer from context):

| Parameter | Default | Notes |
|-----------|---------|-------|
| Agent role | General coding assistant | Coding, research, DevOps, analysis, etc. |
| Target model | Qwen 3.6 27b | Affects tone and structure assumptions |
| Tool access | None specified | Shell, file read/write, web search, MCP tools, etc. |
| Output format | Free-form with code blocks | JSON-only, structured markdown, etc. |
| Safety constraints | Standard | No speculation, verify-before-asserting, etc. |
| Existing prompt | None | If provided, optimize rather than write from scratch |

### 2. Build the Prompt Skeleton

Use this structure as a starting point:

```markdown
# [Agent Role]

## Core Directives
- [3-5 imperative rules that define behavior]

## Tool Use Policy
[When/how to use available tools, or "No tool access"]

## Output Rules
[Format requirements, code block conventions, response length guidance]

## Domain Knowledge
[Brief domain-specific guidance if applicable — keep under 200 words]

## Safety & Verification
[What to do when uncertain, how to handle errors, citation rules]
```

### 3. Optimize for the Target Model

**Qwen 3.x / Qwen 2.5-Coder specific optimizations:**

- Use `##` headers — Qwen parses markdown headers reliably as section boundaries
- Prefer bullet lists over numbered lists for non-sequential rules
- Include explicit tool-call format instructions if tools are available (Qwen tends to wrap tool calls in code blocks unless told not to)
- Avoid nested JSON examples in the system prompt — inline them as flat key-value pairs instead
- State the response language explicitly if non-English is desired

**General mid-size model optimizations:**

- Keep each bullet under 80 characters when possible
- Use `|` tables for mapping relationships (task → tool, error → action)
- Replace long explanations with concrete examples in code fences
- Remove any sentence that doesn't change the model's behavior if deleted

### 4. Token Budget Check

After drafting, estimate token count:

```
Rough estimate: character_count / 4 ≈ tokens (English text)
Target: 800-1500 tokens unless user specifies otherwise
```

If over budget, cut in this order:
1. Domain knowledge examples (keep the rule, drop the illustration)
2. Safety section elaboration (keep the constraint, drop the rationale)
3. Tool use edge cases (keep primary patterns, drop exceptions)

### 5. Output Format

Present the result as:

1. **The system prompt** in a code block (ready to copy-paste)
2. **Token estimate** and what was optimized for
3. **Trade-off notes** — what was sacrificed for compactness, if anything

## Example

**Input:** "I need a system prompt for a Qwen 27b coding agent that can read files and run shell commands"

**Output skeleton:**

```markdown
# Code Assistant

## Core Directives
- Answer coding questions with working code, not descriptions
- Read files before modifying them; never guess file contents
- Run shell commands to verify changes work
- Explain non-obvious decisions in 1-2 sentences max

## Tool Use Policy
- `read_file`: Always read a file before editing or deleting it
- `bash`: Run tests, compile checks, and verification commands after changes
- Never execute destructive commands (rm -rf, format, etc.) without explicit user confirmation

## Output Rules
- Code in fenced blocks with language tags
- Shell commands in separate fenced blocks
- Keep explanations under 3 paragraphs unless the user asks for detail

## Safety & Verification
- If unsure about an API or library behavior, search or test it — do not guess
- State assumptions explicitly when they cannot be verified
```

## Optimization Mode

When the user provides an existing system prompt:

1. Count current tokens (rough estimate)
2. Identify redundant sections (overlapping rules, pleasantries, identity statements)
3. Merge related constraints into single bullets
4. Convert prose to structured format (headers, lists, tables)
5. Report before/after token count and what was removed

Always show the diff between original and optimized versions so the user can see what changed.
