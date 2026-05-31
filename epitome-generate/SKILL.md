---
name: epitome-generate
description: Step 2 of the epitome workflow. For each approved archetype in .epitome/tasks.json, creates a directory containing one or more example code files and an ARCHETYPE.md with detection commands, rules, and anti-patterns. Run after epitome-init and human approval of the archetype list.
license: MIT
---

# epitome-generate

Runs **step 2** of the epitome workflow: generate example files and `ARCHETYPE.md` for every
archetype approved in step 1.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Prerequisites

- `.epitome/tasks.json` exists (created by `epitome-init`)
- At least one archetype has `"status": "approved"`

---

## Process

### 1. Read the approved archetypes

```bash
cat .epitome/tasks.json
```

Only process archetypes with `"status": "approved"`. Skip `pending` and `rejected`.

### 2. Study real instances before writing examples

For each archetype, **read 3–5 real instances** from the codebase before writing any examples.
The example must reflect *actual patterns in this codebase*, not a generic textbook version.

```bash
# Find real instances (adapt grep patterns to the archetype)
find . -path "*/target" -prune -o -type f -name "*Service.java" -print | \
  grep -v target | grep -v test | head -5

# Read them
cat path/to/RealService.java
```

Look for:
- Which annotations are always present
- Which imports are idiomatic
- How constructors are structured
- What the class/method access modifiers are
- Any project-specific base classes or utilities being used

### 3. Create the archetype directory and example files

For each approved archetype, create `.epitome/<archetype_id>/`.

#### Naming example files

Use the naming convention from the [directory structure spec](../spec/directory-structure.md):
- File names use `<Entity>` as a placeholder where a concrete domain entity name would go
- Choose a realistic entity name from the codebase (e.g. `Shipment`, `Order`, `Stock`)
- If the archetype requires multiple files, include all of them (e.g. an interface + two implementations)

#### Writing the example code

The example must be:
- **Compilable in spirit** — no invented APIs, use real imports and annotations from the codebase
- **Minimal** — only what is needed to show the pattern; omit business logic that distracts
- **Complete** — show enough that a developer can copy the structure and fill in their domain
- **Annotated** — use inline comments (on separate lines, never at end of line) only where the
  pattern has a non-obvious rule that needs explanation

Do **not** make the example a copy of a real file. Strip it down to the structural skeleton.

### 4. Write ARCHETYPE.md

Follow the [ARCHETYPE.md format spec](../spec/archetype-md-format.md) exactly, including the
YAML frontmatter.

#### Detection commands

Write detection commands that actually work for this codebase. Test them:

```bash
# Test your detection command before writing it into ARCHETYPE.md
find . -path "*/target" -prune -o -name "*Service.java" -print | grep -v target | head -5
```

Write two kinds of detection commands:
1. **Inventory** — find all instances of this archetype
2. **Smell detection** — find instances that likely violate the rules (the more valuable one)

#### Rules / checklist

Each rule must be:
- Specific enough to verify mechanically
- About the *pattern*, not the *domain* (avoid rules like "must handle shipments correctly")
- Written as a checkbox so a reviewer can tick it off

Aim for 5–10 rules. Too few means the pattern is under-specified; too many means it is over-engineered.

#### Anti-patterns

Show the most common mistakes you actually found while reading real instances.
Each ❌ should look like code that *exists or has existed* in the codebase, not a strawman.
Pair every ❌ with a ✅.

### 5. Update tasks.json

After generating all archetypes, update `.epitome/tasks.json`:
- Mark step 2 `done`
- Mark step 3 `in-progress`
- Add a `subtask` entry for each generated archetype under step 2

```json
{
  "id": 2,
  "title": "Create code examples for each archetype",
  "status": "done",
  "subtasks": [
    { "id": "2.1", "title": "controller_rest", "status": "done" },
    { "id": "2.2", "title": "service_domain", "status": "done" }
  ]
}
```

---

## Quality bar

Before considering an archetype done, ask:

- Could a developer new to this codebase understand the pattern from the example alone?
- Would the detection commands actually surface violations if run today?
- Does every anti-pattern reflect something that genuinely happens or has happened in this codebase?
- Are the related archetypes correctly linked?

---

## Output

- `.epitome/<archetype_id>/` directory created for each approved archetype
- Each directory contains example file(s) and `ARCHETYPE.md`
- `.epitome/tasks.json` updated: step 2 `done`, step 3 `in-progress`

Next step: **human review** (step 3). The human reads each example and `ARCHETYPE.md` and
edits them until they are truly ideal. This step is intentionally manual — the human decides
what "ideal" means for their codebase.
