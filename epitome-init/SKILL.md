---
name: epitome-init
description: Step 1 of the epitome workflow. Scans a codebase to identify recurring code archetypes — structural patterns worth defining as canonical examples. Presents them for human approval and creates .epitome/tasks.json. Use when starting an epitome for a new project or expanding an existing one.
license: MIT
---

# epitome-init

Runs **step 1** of the epitome workflow: discover archetypes in the codebase and present
them for human approval before any files are generated.

Read the [directory structure spec](../spec/directory-structure.md) and
[tasks.json format](../spec/tasks-json-format.md) before proceeding.

---

## Goal

Produce a list of archetypes — recurring code patterns worth defining as canonical examples —
and write them to `.epitome/tasks.json` as `pending` entries awaiting human approval.

Do **not** create archetype directories yet. That is step 2 (`epitome-pin`).

---

## Process

### 1. Orient yourself

Understand the project's language, framework, and structure before looking for patterns.

```bash
# High-level layout
find . -maxdepth 4 -type d | grep -v ".git\|build\|target\|node_modules\|__pycache__\|.epitome\|.gradle" | head -60

# Primary language(s)
find . -type f | grep -v ".git\|build\|target\|node_modules\|dist" | \
  sed 's|.*\.||' | sort | uniq -c | sort -rn | head -15
```

Identify: language, framework, module structure, test framework.

---

### 2. Find technical archetypes

Technical archetypes are structural patterns imposed by the framework or language — things that
repeat because of *how the system is built*, not *what it does*.

Look for recurring naming suffixes that map to a consistent class/file structure:

```bash
# Count files by name suffix — adapt extension to the project language
# Kotlin/Java
find . -type f -name "*.kt" -o -name "*.java" | grep -v "build\|target\|node_modules" | \
  sed 's|.*/||; s|\.[^.]*$||' | grep -oE '[A-Z][a-zA-Z]*(Controller|Service|Repository|Handler|Config|Processor|Factory|Builder|Mapper|Test|Tests|Spec|IntegrationTest)$' | \
  sort | uniq -c | sort -rn

# TypeScript/JavaScript
find . -type f \( -name "*.ts" -o -name "*.tsx" \) | grep -v "node_modules\|dist" | \
  sed 's|.*/||; s|\.[^.]*$||' | grep -oE '\.(controller|service|repository|handler|middleware|test|spec)$' | \
  sort | uniq -c | sort -rn

# Python
find . -type f -name "*.py" | grep -v "__pycache__\|.venv\|site-packages" | \
  sed 's|.*/||; s|\.py$||' | grep -oE '(controller|service|repository|handler|test_|_test)' | \
  sort | uniq -c | sort -rn
```

**Threshold**: propose a technical archetype if it appears **5+ times** (for projects with
<50 files) or **10+ times** (for larger codebases). Fewer instances may still qualify if
the pattern is structurally rigid and important to get right.

---

### 3. Find domain archetypes

Domain archetypes are patterns that repeat because of *what the business does*. Harder to
find, more valuable — they encode knowledge new team members need to absorb.

Look for:
- **Conditional behaviour** — same operation implemented differently for different runtime
  contexts (tenant, environment, feature flag, user role)
- **Lifecycle patterns** — entities with a consistent multi-step lifecycle
- **External integration patterns** — a consistent way of calling external APIs or queues
- **Audit/observability patterns** — operations systematically logged or traced
- **Reusable query fragments** — complex queries appearing in multiple places in slightly
  different forms

```bash
# Find conditional/strategy pairs: classes sharing a name but with different prefixes/suffixes
find . -type f | grep -v "build\|target\|node_modules" | \
  sed 's|.*/||; s|\.[^.]*$||' | sort | uniq -c | sort -rn | head -30

# Find any annotation/decorator applied consistently across many files
grep -rn "@Logged\|@Audited\|@Cached\|@Retry\|@RateLimit\|@Transactional" \
  . --include="*.kt" --include="*.java" --include="*.ts" --include="*.py" \
  2>/dev/null | grep -v "build\|target\|node_modules" | wc -l
```

Ask: *would a new developer need to know this pattern to write correct code in this codebase?*
If yes, it is a domain archetype worth capturing.

---

### 4. Rule out one-offs

An archetype must be:
- **Recurring** — appears in multiple places with a consistent shape
- **Structural** — has a consistent skeleton that can be shown by pointing at one file
- **Meaningful** — knowing the pattern helps you write better code in this codebase

If a pattern only appears 1–2 times or varies too much to generalise, skip it.

---

### 5. Present for approval

Show proposed archetypes grouped by type (technical vs domain). For each, provide:
- A one-line description of what it is
- Count of instances found
- A representative file path (the kind of file that would be the epitome)
- Any notes about complexity or variations

Wait for explicit human approval before proceeding.

---

### 6. Write tasks.json

Once the human approves (or modifies) the list, create or update `.epitome/tasks.json`.

Follow the [tasks.json format spec](../spec/tasks-json-format.md) exactly.

- Step 1 status: `done`
- Step 2 status: `in-progress`
- All approved archetypes: `status: "approved"`
- Rejected proposals: omit entirely

```json
{
  "project": "<name of the project>",
  "steps": [
    { "id": 1, "title": "Identify archetypes", "status": "done", "subtasks": [
      { "id": "1.1", "title": "Explore codebase for patterns", "status": "done" },
      { "id": "1.2", "title": "Present archetypes for approval", "status": "done" }
    ]},
    { "id": 2, "title": "Pin epitome file for each archetype", "status": "in-progress", "subtasks": [] },
    { "id": 3, "title": "Review each archetype against real instances", "status": "todo", "subtasks": [] },
    { "id": 4, "title": "Write MANIFESTO.md", "status": "todo", "subtasks": [] },
    { "id": 5, "title": "Refactor & iterate", "status": "todo", "subtasks": [] }
  ],
  "archetypes": [
    { "id": "controller_rest", "folder": ".epitome/controller_rest/", "status": "approved" },
    { "id": "service_domain", "folder": ".epitome/service_domain/", "status": "approved" }
  ]
}
```

---

## Output

- `.epitome/tasks.json` created
- Human has seen and approved the archetype list
- No archetype directories created yet

Next step: run `epitome-pin`.
