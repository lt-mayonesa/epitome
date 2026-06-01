---
name: epitome-manifesto
description: Step 4 of the epitome workflow. Checks for existing AGENTS.md / CLAUDE.md. Writes .epitome/MANIFESTO.md — a slim, agent-facing document covering the archetype-first development protocol and, only if no AGENTS.md/CLAUDE.md exists, a brief project overview. Patterns and guidelines live in the ARCHETYPE.md files, not here. Run after all archetypes have been reviewed.
license: MIT
---

# epitome-manifesto

Runs **step 4** of the epitome workflow: write `.epitome/MANIFESTO.md`.

The MANIFESTO has one primary job: tell the agent **how to use the archetypes** — and specifically
what to do when a code pattern has no archetype yet.

Patterns and rules live in the `ARCHETYPE.md` files. Coding guidelines live in `AGENTS.md` /
`CLAUDE.md`. The MANIFESTO does not repeat those. It is the glue between the two.

---

## What MANIFESTO.md is NOT

- Not a copy of naming conventions (those are in `AGENTS.md` or `ARCHETYPE.md`)
- Not a copy of testing standards (those are in `ARCHETYPE.md` files and `AGENTS.md`)
- Not a full project spec
- Not prose about best practices

## What MANIFESTO.md IS

- The archetype-first development protocol (mandatory section)
- A list of all archetypes with one-liner summaries (so the agent can scan without reading each file)
- A pointer to AGENTS.md / CLAUDE.md for everything else
- A brief project overview ONLY if no AGENTS.md / CLAUDE.md exists

---

## Prerequisites

- All archetypes in `.epitome/tasks.json` have `"status": "reviewed"` (or human explicitly proceeds)

---

## Process

### 1. Check for existing context files

```bash
ls -la AGENTS.md CLAUDE.md 2>/dev/null
```

- If `AGENTS.md` or `CLAUDE.md` exists: **do not repeat their content**. Reference them in MANIFESTO.
- If neither exists: write a 3–5 sentence project overview at the top of MANIFESTO.

### 2. Read all ARCHETYPE.md files

```bash
for f in .epitome/*/ARCHETYPE.md; do
  echo "=== $f ==="
  head -20 "$f"
  echo
done
```

Extract:
- Each archetype `id` and `epitome_file`
- One-line summary of what it covers (from "What it is" section)

### 3. Write MANIFESTO.md

Write `.epitome/MANIFESTO.md`. Use this exact structure:

---

```markdown
# MANIFESTO

> You are working on **<project name>**. Read this document before implementing any feature.
> The archetypes in `.epitome/` show you *how* code looks. This document tells you *when* to use them and *what to do* when they don't cover your case.

---

## Project overview
<!-- INCLUDE THIS SECTION ONLY if no AGENTS.md / CLAUDE.md exists -->
<3–5 sentences: what the project does, primary stack, scale>

---

## Archetype-first development

**Before writing any code, identify which archetype covers the pattern you are implementing.**

Check `.epitome/` for the relevant archetype. Open the `ARCHETYPE.md` and read the pinned
`epitome_file` to understand what ideal code for that pattern looks like in this codebase.

### When the pattern IS covered by an archetype

Implement following the archetype's rules and using the `epitome_file` as your reference.
The archetype is the source of truth — not general framework documentation, not external examples.

### When the pattern is NOT covered by any archetype

**Stop. Do not implement.** Follow this protocol instead:

1. Tell the user exactly which pattern is missing:
   > "There is no archetype for `<pattern name>` in this codebase. Before I implement this,
   > we need to define what ideal code looks like for this pattern."

2. Look at the codebase for related existing code that could serve as a starting point.
   Find 1–3 candidate files or describe the options if no code exists yet:
   > "I found these candidates that could be the epitome for this pattern:
   > - `src/.../ExistingFile.kt` — [one sentence why it's a good candidate]
   > - Use general Kotlin/Spring best practices to write a new example from scratch
   > Which should be the canonical example for this pattern?"

3. Wait for the user's decision. They will either:
   - Pick an existing file → run `epitome-pin` to register it
   - Say "write it from scratch" → implement the new code → immediately run `epitome-pin`
     to register the new file as the archetype

4. Only proceed with full implementation once the archetype is pinned.

**This applies even if you know general best practices.** The manifesto is project-specific.
When in doubt, ask.

---

## Archetypes

<!-- One line per archetype. Agent uses this to quickly scan what is covered. -->

| Archetype | What it covers | Epitome file |
|---|---|---|
| [`controller_rest`](.epitome/controller_rest/ARCHETYPE.md) | <one-line> | `<epitome_file>` |
| [`endpoint_get_single`](.epitome/endpoint_get_single/ARCHETYPE.md) | <one-line> | `<epitome_file>` |
| ... | | |

---

## Other guidelines

See [`AGENTS.md`](AGENTS.md) for naming conventions, module boundaries, error handling,
testing standards, and all other project-specific rules.
<!-- If no AGENTS.md exists, omit this section -->
```

---

### 4. Ask the human to review

After writing:

```
MANIFESTO.md written. Please review:
- Is the archetype-first protocol described correctly?
- Are any archetypes missing from the table?
- Is anything in AGENTS.md that should also be explicitly called out here?
```

Apply any corrections.

---

### 5. Update tasks.json

```json
{ "id": 4, "title": "Write MANIFESTO.md", "status": "done", "subtasks": [
  { "id": "4.1", "title": "Write MANIFESTO.md", "status": "done" }
]},
{ "id": 5, "title": "Refactor & iterate", "status": "in-progress", "subtasks": [] }
```

---

## Output

- `.epitome/MANIFESTO.md` written — slim, archetype-first protocol + archetype table
- `.epitome/tasks.json` updated: step 4 `done`, step 5 `in-progress`

Next step: run `epitome-refactor` to find and fix drift in the codebase.
