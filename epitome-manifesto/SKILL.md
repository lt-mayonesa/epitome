---
name: epitome-manifesto
description: Step 4 of the epitome workflow. Checks for existing AGENTS.md / CLAUDE.md. Writes .epitome/MANIFESTO.md — a slim, agent-facing document covering the archetype-first development protocol (including the skill-aware pinning protocol) and, only if no AGENTS.md/CLAUDE.md exists, a brief project overview. Run after all archetypes have status reviewed.
license: MIT
---

# epitome-manifesto

Runs **step 4** of the epitome workflow: write `.epitome/MANIFESTO.md`.

The MANIFESTO has one primary job: tell the agent **how to use the archetypes** — and
specifically what to do when a code pattern has no archetype yet (the gap protocol).

Patterns and rules live in `ARCHETYPE.md` files. Coding guidelines live in `AGENTS.md` /
`CLAUDE.md`. The MANIFESTO does not repeat those. It is the glue between them.

---

## What MANIFESTO.md is NOT

- Not a copy of naming conventions (those are in `AGENTS.md` or `ARCHETYPE.md`)
- Not a copy of testing standards
- Not a full project spec
- Not prose about best practices

## What MANIFESTO.md IS

- The archetype-first gap protocol (mandatory — this is the core value)
- A table of all archetypes with one-liner summaries
- A "Patterns NOT yet defined" list as a convenience hint
- A pointer to AGENTS.md / CLAUDE.md for everything else
- A brief project overview ONLY if no AGENTS.md / CLAUDE.md exists

---

## Prerequisites

- All archetypes in `.epitome/` have `status: reviewed`
- (Or the human has explicitly asked to proceed with partially reviewed archetypes)

---

## Process

### 1. Check for existing context files

```bash
ls -la AGENTS.md CLAUDE.md 2>/dev/null
```

- If either exists: reference it in MANIFESTO, do not repeat its content
- If neither exists: write a 3–5 sentence project overview at the top

### 2. Read all ARCHETYPE.md files

```bash
for f in .epitome/*/ARCHETYPE.md; do
  echo "=== $f ==="
  head -15 "$f"
  echo
done
```

Extract each archetype `id`, `epitome_file`, and one-line summary from "What it is".

### 3. Write MANIFESTO.md

Use this exact structure:

---

```markdown
# MANIFESTO

> You are working on **<project name>**. Read this document before implementing any feature.
> The archetypes in `.epitome/` show you *how* code looks.
> This document tells you *when* to use them and *what to do* when they don't cover your case.
> For all other project guidelines — naming, testing, module boundaries, error handling — see [`AGENTS.md`](../AGENTS.md).

---

## Project overview
<!-- INCLUDE ONLY if no AGENTS.md / CLAUDE.md exists -->
<3–5 sentences>

---

## Archetype-first development

**Before writing any code, identify which archetype covers the pattern you are implementing.**

Open the relevant `.epitome/<archetype>/ARCHETYPE.md` and read the pinned `epitome_file`
to understand what ideal code looks like in this codebase.

### When the pattern IS covered

Implement following the archetype's rules, using its `epitome_file` as the reference.
The archetype is the source of truth — not general framework documentation, not external examples.

### When the pattern is NOT covered by any archetype

**Stop. Do not implement.** Follow this protocol:

1. Tell the user exactly which pattern is missing:
   > "There is no archetype for `<pattern>` in this codebase. We need to define what ideal
   > code looks like for this pattern before I implement it."

2. Look in the codebase for related existing code that could serve as a starting point.
   Suggest 1–3 candidates, or note that none exist yet:
   > "Candidates for the epitome:
   > - `src/.../ExistingFile.kt` — [why it's a good candidate]
   > - No existing example — I can write the first instance and register it"

3. Wait for the user's choice, then register the new archetype:

   **Check if `epitome-pin` skill is installed:**
   ```bash
   ls ~/.pi/agent/skills/epitome-pin 2>/dev/null || \
   ls .pi/skills/epitome-pin 2>/dev/null || echo "NOT_FOUND"
   ```
   - If found → run `epitome-pin` for the new archetype id
   - If not found → create `.epitome/<id>/ARCHETYPE.md` manually with `status: reviewed`,
     then update this MANIFESTO (add to table, remove from "NOT yet defined" if present),
     and warn: ⚠ epitome-pin not installed — registered manually

4. Only proceed with full feature implementation once the archetype is registered.

**This applies even if you know general best practices.** One canonical example per pattern.
When in doubt, stop and ask.

---

## Archetypes

<!-- One row per archetype. Agent scans this to know what is covered. -->
<!-- The .epitome/ directory is authoritative; this table is a convenience summary. -->

| Archetype | What it covers | Epitome file |
|---|---|---|
| [`controller_rest`](.epitome/controller_rest/ARCHETYPE.md) | <one-line> | `<filename>` |
| [`endpoint_get_single`](.epitome/endpoint_get_single/ARCHETYPE.md) | <one-line> | `<filename>` |
| ... | | |

**Patterns NOT yet defined** (= stop and define before implementing):
- `endpoint_get_list` — GET returning a list with no filter params
- `endpoint_post_create` — synchronous POST returning 201 CREATED
<!-- Add any other known gaps here -->

---

## Other guidelines

See [`AGENTS.md`](AGENTS.md) for naming conventions, module boundaries, error handling,
testing standards, and all other project-specific rules.
<!-- Omit if no AGENTS.md / CLAUDE.md exists -->
```

---

### 4. Ask the human to review

```
MANIFESTO.md written. Please review:
- Is the archetype-first protocol correct for this project?
- Are any archetypes missing from the table?
- Is the "Patterns NOT yet defined" list accurate?
```

Apply corrections.

---

## Output

- `.epitome/MANIFESTO.md` written and human-approved

Next step: run `epitome-refactor` to find and fix drift.
