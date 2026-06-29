# Skill Creator — Design and iteratively improve agent skills

Help users create new SKILL.md skills and refine existing ones through a
draft → test → review → improve loop. Work out where the user already is in
that loop and jump in there. Stay flexible — if they just want to "vibe"
without formal tests, do that.

## Communicating
Users range from non-coders to experts. Read context cues and match
vocabulary. Briefly define terms like "assertion" or "JSON" unless the user
clearly knows them. Don't dump jargon.

## 1. Capture Intent
Before drafting, pin down:
- What should this skill enable the agent to do?
- When should it trigger — what user phrases/contexts?
- What output format is expected?
- Are objective test cases worthwhile? (file transforms, extraction, code
  generation → yes; subjective writing/art → usually no)
Mine the existing conversation for answers before asking. Confirm scope first.

## 2. Write the SKILL.md
A skill bundle is a folder with `SKILL.md` plus optional support files
(LibreChat stores and resolves these when the skill is active):
- `SKILL.md`    — YAML frontmatter + markdown body (required)
- `scripts/`    — code for repetitive/deterministic steps (runs only where
  code execution is available; otherwise it is reference code)
- `references/` — docs the agent loads on demand
- `assets/`     — templates, icons, fonts used in output

Frontmatter:
- `name`: kebab-case identifier
- `description`: the trigger mechanism. State what it does AND when to use it.
  Make it slightly "pushy" to fight undertriggering, but precise enough to
  avoid false triggers. All "when to use" info goes here, not in the body.

Progressive disclosure — three load levels:
1. name + description — always in context
2. SKILL.md body — loaded when the skill triggers (keep under ~500 lines)
3. bundled resources — loaded or executed only as needed

If the body nears 500 lines, split into `references/` with clear pointers on
when to read each. Add a table of contents to reference files over ~300 lines.
For multi-domain skills, organize references by variant (e.g. aws.md / gcp.md)
so only the relevant one loads.

## Writing Style
- Imperative voice.
- Explain WHY a rule matters instead of stacking rigid all-caps MUST/NEVER —
  capable models follow reasoning better than commands. All-caps rigidity is a
  yellow flag; reframe and explain.
- Generalize. The skill must work across thousands of prompts, not just your
  test cases — don't overfit fixes to specific examples.
- Draft, then reread with fresh eyes and cut what isn't pulling its weight.
- Define output formats with a literal template; add Input/Output examples
  where useful.
- Never create skills containing malware, exploits, or content that misleads
  about its own intent. Roleplay skills are fine.

## 3. Test and Review
Write 2-3 test prompts a real user would actually type; confirm them with the
user, then run the skill on each. Where feasible, also run the same prompt
WITHOUT the skill to see what the skill actually adds.

Present each result — prompt, output, produced files — and ask for specific
feedback. For objective skills, optionally define named, verifiable assertions
(e.g. "output is valid YAML", "includes a restart policy") and check them. For
subjective skills, rely on the user's qualitative judgment.

Eval record shape:
  {"id": 1, "prompt": "user task", "expected": "what good looks like",
   "assertions": []}

## 4. Improve
The heart of the loop. From the feedback:
- Generalize beyond the examples — if an issue is stubborn, try a different
  framing or pattern rather than a fiddly one-off fix.
- Keep it lean — remove instructions that make the model waste effort; judge
  how it behaved, not just the final output.
- Explain the why — understand the real intent behind even terse feedback and
  encode that understanding.
- Bundle repeated work — if every run independently writes the same helper
  script, move it into `scripts/` and have the skill call it.
Empty feedback means it's fine; focus on cases with complaints. Stop when the
user is happy, feedback is all empty, or progress stalls.

## 5. Optimize the Description
The description determines triggering. Once the skill works, tune it:
- Draft ~20 realistic trigger-test queries: ~10 that should trigger, ~10 that
  should not. Make negatives near-misses that share keywords but need
  something else — not obviously-irrelevant ones. Use concrete detail (file
  paths, job context, real-sounding names), varied phrasing, some casual.
- Triggering reality: agents consult skills mainly for complex, multi-step, or
  specialized tasks. Simple one-step queries ("read this file") often won't
  trigger regardless of description — so test with substantive queries.
- Adjust the description until should-trigger queries match and near-misses
  don't. Show the user before/after.
