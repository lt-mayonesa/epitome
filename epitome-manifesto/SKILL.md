---
name: epitome-manifesto
description: Step 4 of the epitome workflow. Reads all reviewed ARCHETYPE.md files and synthesises a MANIFESTO.md — a prompt-style document covering naming conventions, architectural decisions, commit hygiene, and cross-cutting rules that cannot be expressed in code examples. Run after all archetypes have been reviewed.
license: MIT
---

# epitome-manifesto

Runs **step 4** of the epitome workflow: synthesise what the archetypes say into a
`MANIFESTO.md` — a document written for an AI agent to read when implementing a new
feature or reviewing existing code.

Read the [directory structure spec](../spec/directory-structure.md) before proceeding.

---

## What MANIFESTO.md is

The archetypes cover *structural* patterns — what a controller looks like, how a service
is organised. The manifesto covers everything else:

- **Naming conventions** — file names, class names, method names, variable names, constants
- **Module/package organisation** — what belongs where, what may not depend on what
- **Architectural decisions** — choices the team made and why (even if the code doesn't enforce them)
- **Cross-cutting rules** — things that apply everywhere but don't fit in one archetype
- **Commit and PR hygiene** — how to write commit messages, how large a PR should be
- **What to do when unsure** — guidance for edge cases

MANIFESTO.md is written **in the second person, addressed to an AI agent**. It should
read like a briefing: direct, specific, no filler. An agent reading it before implementing
a feature should come away knowing exactly what decisions have already been made.

---

## Prerequisites

- All archetypes in `.epitome/tasks.json` have `"status": "reviewed"`
- (Or the human has explicitly asked to proceed with partially reviewed archetypes)

---

## Process

### 1. Read all ARCHETYPE.md files

```bash
for f in .epitome/*/ARCHETYPE.md; do echo "=== $f ==="; cat "$f"; echo; done
```

Note:
- Naming patterns across archetypes (class suffixes, file placement)
- Common imports and dependencies (framework choices)
- Patterns that span multiple archetypes (e.g. logging style, error handling)
- Any rules that appeared in multiple ARCHETYPE.md files

---

### 2. Read the epitome files

```bash
# Get all epitome_file paths from frontmatter
grep "epitome_file:" .epitome/*/ARCHETYPE.md | sed 's/.*epitome_file: //'
```

Skim each epitome file for:
- Package/import structure
- Common base classes or interfaces
- Shared utilities (logging, error types, response wrappers)
- Patterns not captured by any single archetype

---

### 3. Explore the codebase for manifesto-level patterns

Look for things that affect all code but aren't captured in a single archetype:

```bash
# Naming: what suffixes exist that aren't archetypes?
find . -type f | grep -v "build\|target\|node_modules\|.git" | \
  sed 's|.*/||; s|\.[^.]*$||' | sort -u | head -50

# Package/module organisation
find . -type d | grep "src/main\|src/test\|app\|lib" | \
  grep -v "build\|target\|node_modules" | sort

# Error handling patterns
grep -rn "throw\|catch\|Exception\|Error\|Result\|Either" \
  . --include="*.kt" --include="*.ts" --include="*.py" --include="*.java" \
  2>/dev/null | grep -v "build\|target\|test\|node_modules" | wc -l
```

---

### 4. Ask the human for explicit rules

Before writing, ask the human if there are rules the archetypes don't capture:

```
I've read all archetypes and the epitome files. Before writing MANIFESTO.md, are there
any rules, conventions, or decisions you want explicitly included that the archetypes
don't already show?

For example:
- Naming conventions not obvious from the files?
- Module dependency rules?
- Commit message format?
- Rules about when NOT to use a pattern?
- Team agreements not visible in the code?
```

Wait for their answer (or explicit "nothing to add") before proceeding.

---

### 5. Write MANIFESTO.md

Write `.epitome/MANIFESTO.md`. Structure:

```markdown
# MANIFESTO

> You are an AI agent working on <project name>. This document is your briefing.
> Read it before implementing any feature or reviewing any code.
> The archetypes in `.epitome/` show you *how* things look. This document tells you *why*
> and covers everything that code examples cannot.

## Project overview

<2–3 sentences: what this project does, its primary tech stack, its scale>

## Architecture

<Module/package layout. What depends on what. What must never depend on what.>

## Naming conventions

<Class names, file names, method names, constant names. Be specific.>

## Patterns to always follow

<Cross-cutting rules. Things that apply everywhere.>

## Patterns to never use

<Anti-patterns at the project level. Things explicitly banned.>

## Error handling

<How errors propagate. What exception types exist. When to recover vs propagate.>

## Testing

<What to test. What not to test. Test naming conventions. Coverage expectations.>

## Commit and PR hygiene

<Commit message format. PR size. Branch naming.>

## When you are unsure

<What to do when the archetypes and manifesto don't cover the case.>
```

#### Writing style rules

- Address the agent as "you" / second person
- Every rule must be actionable: "Do X" or "Never do Y" or "When Z, do W"
- No filler, no hedging, no "it is recommended that"
- Reference archetypes by name when relevant: "see `controller_rest`"
- If a rule has a *why*, put it in parentheses after the rule, briefly

Example of good manifesto prose:

```markdown
## Naming conventions

File names match the primary class/object they contain, exactly.
Class names use PascalCase. No abbreviations except well-known ones (HTTP, URL, ID).
Service classes are suffixed `Service`. Controllers are suffixed `Controller`.
Test files are suffixed `Test` for unit tests, `IntegrationTest` for integration tests.
Test data factories are suffixed `TestData` and are `object` singletons.

## Patterns to never use

Never use field injection (`@Autowired` on a field). Constructor injection only.
(Reason: makes dependencies explicit and tests easier to write without Spring context.)

Never put business logic in a controller. Controllers delegate to services. Period.
```

---

### 6. Review with the human

After writing, present the manifesto for review:

```
MANIFESTO.md written. Please review:
- Is anything important missing?
- Are any rules too strict or too vague?
- Are any architectural decisions described incorrectly?
```

Apply any corrections, then proceed.

---

### 7. Update tasks.json

```json
{ "id": 4, "title": "Write MANIFESTO.md", "status": "done", "subtasks": [
  { "id": "4.1", "title": "Write MANIFESTO.md", "status": "done" }
]},
{ "id": 5, "title": "Refactor & iterate", "status": "in-progress", "subtasks": [] }
```

---

## Output

- `.epitome/MANIFESTO.md` written and human-approved
- `.epitome/tasks.json` updated: step 4 `done`, step 5 `in-progress`

Next step: run `epitome-refactor` to find and fix drift in the codebase.
