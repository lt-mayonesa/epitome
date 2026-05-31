---
name: epitome-init
description: Step 1 of the epitome workflow. Scans a codebase to identify recurring code archetypes — both technical (controller, service, repository) and domain-specific (warehouse variant, DB-driven config). Presents them for human approval and creates .epitome/tasks.json. Use when starting an epitome for a new project or expanding an existing one.
license: MIT
---

# epitome-init

Runs **step 1** of the epitome workflow: discover archetypes in the codebase and present them
for human approval before any files are generated.

Read the [directory structure spec](../spec/directory-structure.md) and
[tasks.json format](../spec/tasks-json-format.md) before proceeding.

---

## Goal

Produce a list of archetypes — recurring code patterns worth defining as examples — and write
them to `.epitome/tasks.json` as `pending` entries awaiting human approval.

Do **not** create archetype directories yet. That is step 2 (`epitome-generate`).

---

## Process

### 1. Orient yourself

Understand the project's language, framework, and module structure before looking for patterns.

```bash
# Get the high-level layout
find . -maxdepth 3 -type d | grep -v ".git\|target\|node_modules\|__pycache__\|.epitome" | head -60

# Identify the primary language(s)
find . -type f | grep -v ".git\|target\|node_modules" | sed 's|.*\.||' | sort | uniq -c | sort -rn | head -15
```

### 2. Find technical archetypes

Technical archetypes are structural patterns imposed by the framework or language — things that
repeat because of *how the system is built*, not *what it does*.

Look for:
- Naming suffixes that recur at scale (`*Controller`, `*Service`, `*Repository`, `*Handler`, `*Config`, `*Test`, `*Builder`, etc.)
- Count occurrences to distinguish real archetypes from one-offs

```bash
# Count recurring name suffixes (adapt the pattern to the language)
find . -path "*/target" -prune -o -type f -name "*.java" -print | \
  grep -v "/test/" | sed 's|.*/||' | \
  grep -oE '[A-Z][a-zA-Z]+(Controller|Service|Repository|Config|Handler|Builder|Factory|Test)\.java' | \
  sed 's/\.java//' | grep -oE '(Controller|Service|Repository|Config|Handler|Builder|Factory|Test)$' | \
  sort | uniq -c | sort -rn
```

Only propose a technical archetype if it appears **10+ times** in the codebase.

### 3. Find domain archetypes

Domain archetypes are patterns that repeat because of *what the business does*. They are harder
to find but more valuable — they encode knowledge that new team members need to absorb.

Look for:
- **Conditional behaviour** — same operation implemented differently depending on a runtime context
  (e.g. warehouse type, tenant, environment, feature flag)
- **Lifecycle patterns** — entities that go through a consistent plan → execute → confirm lifecycle
- **Configuration patterns** — configuration stored in a database rather than static files
- **Audit/observability patterns** — operations that are systematically logged or traced
- **Reusable query fragments** — complex queries that appear in multiple places in slightly different forms

```bash
# Find conditional/strategy patterns (Java example)
grep -rn "ConditionalOn\|instanceof.*check\|switch.*type" \
  src/main/java --include="*.java" | grep -v target | wc -l

# Find pairs of classes sharing a name but prefixed differently (FC/DC, Dev/Prod, etc.)
find src/main/java -name "*.java" | grep -v target | sed 's|.*/||; s|\.java||' | \
  grep -oE '^[A-Z][a-z]+' | sort | uniq -c | sort -rn | head -20

# Find configuration patterns
grep -rn "CONFIG_KEY\|@ConfigurationProperties\|@Value.Immutable" \
  src/main/java --include="*.java" | grep -v target | wc -l
```

Ask yourself: *would a new developer need to know this pattern to write correct code in this codebase?*
If yes, it is a domain archetype worth capturing.

### 4. Rule out one-offs

An archetype must be:
- **Recurring** — appears in many places, not just one or two
- **Structural** — has a consistent shape that can be shown in an example
- **Meaningful** — knowing the pattern helps you write better code in this codebase

If a pattern only appears 2–3 times or its shape varies too much to generalise, skip it.

### 5. Present for approval

Show the proposed archetypes grouped by type (technical vs domain). For each, provide:
- A one-line description of what it is
- The count of instances found
- Any notes about complexity or variations

Wait for explicit human approval before proceeding.

### 6. Write tasks.json

Once the human approves (or modifies) the list, create or update `.epitome/tasks.json`.

Follow the [tasks.json format spec](../spec/tasks-json-format.md) exactly.

- Step 1 status: `done`
- Step 2 status: `in-progress`
- All approved archetypes: `status: "approved"`
- Any human-rejected proposals: omit entirely

```json
{
  "project": "<name of the project>",
  "steps": [
    { "id": 1, "title": "Identify archetypes", "status": "done", "subtasks": [...] },
    { "id": 2, "title": "Create code examples for each archetype", "status": "in-progress", "subtasks": [] },
    { "id": 3, "title": "Manual review & update of each epitome archetype", "status": "todo", "subtasks": [] },
    { "id": 4, "title": "Write MANIFESTO.md", "status": "todo", "subtasks": [] },
    { "id": 5, "title": "Test & iterate", "status": "todo", "subtasks": [] }
  ],
  "archetypes": [
    { "id": "controller_rest", "folder": ".epitome/controller_rest/", "status": "approved" }
  ]
}
```

---

## Output

- `.epitome/tasks.json` created or updated
- Human has seen and approved the archetype list
- No archetype directories created yet

Next step: run `epitome-generate`.
