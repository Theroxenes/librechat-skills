---
name: code-review-bug-finder
description: "Review code diffs, pull requests, or pasted code to find functional bugs and reproducible errors. Triggers when the user asks you to review code for bugs, find errors in a PR, audit code correctness, spot logic flaws, trace execution paths for defects, or perform any bug-focused code analysis. Do NOT trigger for style reviews, refactoring suggestions, documentation requests, or general code explanation."
---

# Code Review — Bug Finder

## Purpose

Find functional, reproducible bugs in code. Prioritize correctness over style. Every finding must be traceable to a concrete execution path or data flow that produces wrong behavior.

## Scope

**Focus on:** logic errors, off-by-one bugs, null/undefined dereferences, race conditions, incorrect assumptions about API contracts, missing error handling that causes silent failures, incorrect state transitions, resource leaks, incorrect boundary conditions, type mismatches that cause runtime errors, and data corruption paths.

**De-prioritize:** naming conventions, formatting, minor style issues, architectural opinions without correctness impact.

## Review Workflow

### Pass 1 — Scan for High-Impact Bugs

Read the diff or code top-to-bottom. Flag anything that could cause:

- Crashes or unhandled exceptions in normal operation
- Silent data corruption or incorrect results
- Security-relevant logic errors (auth bypass, injection, privilege escalation)
- Resource exhaustion (memory leaks, unclosed handles, unbounded growth)
- Race conditions or concurrency bugs

### Pass 2 — Verify Each Finding

For every flagged issue, trace the execution path:

1. Identify the exact line(s) and condition that triggers the bug
2. State what input or state causes it
3. Describe the wrong behavior that results
4. Classify severity: **Critical** (data loss/crash), **High** (incorrect results in common paths), **Medium** (edge-case failure), **Low** (unlikely but possible)

Discard findings you cannot trace to a concrete path. Prefer precision over recall — one verified bug beats five guesses.

### Pass 3 — Suggest Fixes

For each verified finding, provide a minimal fix. Show the corrected code inline. Explain why the fix works, not just what changed.

## Context Gathering

Before reviewing, gather necessary context:

- Read the full diff, not just individual files
- Check related files for shared state, interfaces, or contracts
- Look at test files to understand expected behavior
- Read PR description for intent — bugs are deviations from stated intent
- For large PRs, focus on changed code and its immediate dependencies

Use `get_file_contents` or `pull_request_read` with `get_files` and `get_diff` methods to read the actual code. Never review from memory alone.

## Output Format

Structure findings as a table, then detail each:

| # | File | Line(s) | Severity | Bug Type |
|---|------|---------|----------|----------|
| 1 | `path/to/file.py` | 42-45 | Critical | Null dereference |

Then for each finding:

```
### Finding #1 — [Short title]
**File:** `path/to/file.py`, lines 42-45
**Severity:** Critical
**What's wrong:** Clear description of the bug and why it occurs
**Trigger condition:** What input/state causes it
**Impact:** What goes wrong at runtime
**Suggested fix:** Code snippet showing the correction
```

End with a summary: total findings by severity, overall risk assessment, and whether the PR is safe to merge.

## Anti-Patterns to Avoid

- **Don't** report style issues as bugs
- **Don't** speculate about bugs without tracing the path
- **Don't** flag "potential" issues that require unlikely conditions — be concrete
- **Don't** suggest rewrites when a minimal fix suffices
- **Don't** ignore the PR description — intent matters for correctness
- **Don't** skip unchanged code that the diff depends on (imports, shared state)

## Language-Specific Checks

Adapt your review to the language. Common traps:

- **Python:** mutable default args, exception swallowing with bare `except:`, GIL assumptions, truthiness pitfalls (`[]` vs `None`)
- **JavaScript/TypeScript:** `undefined` vs `null`, async/await without error handling, closure variable capture in loops, event loop ordering
- **Java/C#:** unchecked exceptions, resource leaks without try-with-resources/using, concurrent collection modification
- **C/C++:** buffer overflows, use-after-free, uninitialized variables, signed/unsigned comparison warnings
- **Go:** goroutine leaks, channel deadlocks, interface nil vs pointer nil

## Test Prompts

1. "Review this PR for bugs: owner/repo#42"
2. "Find functional errors in this diff: [paste diff]"
3. "Audit this code for correctness issues before we merge"
