---
name: epitome-pin
description: Step 2 of the epitome workflow. For each approved archetype in .epitome/tasks.json, finds all real instances in the codebase, presents them for the human to choose one as the canonical epitome file, then writes ARCHETYPE.md with that pointer plus detection commands and rules. Run after epitome-init and human approval.
license: MIT
---

# epitome-pin

Runs **step 2** of the epitome workflow: for each approved archetype, find all real instances,
let the human pick one as the canonical example, and write `ARCHETYPE.md` pointing to it.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Key principle

**No code is copied into `.epitome/`.** The chosen file stays where it lives in the codebase.
`ARCHETYPE.md` holds a pointer (`epitome_file`) to it. When the codebase evolves, the
epitome evolves with it — there is only one source of truth.

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

Only process archetypes with `"status": "approved"`. Skip others.

---

### 2. For each archetype: find all instances

Use naming patterns and framework signatures to locate candidate files. Adapt commands
to the project's language and framework.

```bash
# Example: find all REST controllers in a Kotlin/Spring project
find . -type f -name "*Controller.kt" | grep -v "build\|target\|test"

# Cross-check with framework annotation to confirm pattern
grep -rl "@RestController" . --include="*.kt" | grep -v "build\|target"

# Example: find all services
find . -type f -name "*Service.kt" | grep -v "build\|target\|test"
```

List all candidate files for each archetype. Sort by most recently modified first — recent
files represent current team thinking.

```bash
find . -type f -name "*Controller.kt" | grep -v "build\|target\|test" | \
  xargs ls -t 2>/dev/null
```

---

### 3. Read the candidates

For each archetype, read **3–5 candidates** to understand:
- What annotations/decorators are consistently present
- How dependencies are declared and injected
- What the class/method structure looks like
- How errors are handled
- What imports are idiomatic

This reading is essential — you need it to write good rules in `ARCHETYPE.md`.

---

### 4. Present candidates to the human

Show the candidate files for each archetype. Format:

```
Archetype: controller_rest
Found 3 candidates (most recent first):

  [1] src/main/kotlin/.../OrderController.kt        (modified 2 days ago)
  [2] src/main/kotlin/.../UserController.kt         (modified 1 week ago)
  [3] src/main/kotlin/.../ProductController.kt      (modified 3 weeks ago)

Which file should be the epitome for controller_rest?
(Enter 1–3, or a custom path, or "skip" to defer this archetype)
```

Show this prompt for **one archetype at a time**. Wait for the human's choice before
moving to the next.

If the human picks a file, proceed to step 5.
If the human says "skip", mark the archetype `pending` in tasks.json and move on.
If the human provides a path not in the list, use that path (verify it exists first).

---

### 5. Write ARCHETYPE.md

Create the directory `.epitome/<archetype_id>/` and write `ARCHETYPE.md`.

Follow the [ARCHETYPE.md format spec](../spec/archetype-md-format.md) exactly.

#### Frontmatter

```yaml
---
id: controller_rest
epitome_file: src/main/kotlin/com/example/app/orders/OrderController.kt
detect:
  grep:
    - "@RestController"
  path: "src/main/kotlin/**/*Controller.kt"
related:
  - service_domain
  - test_controller
---
```

- `epitome_file`: the path the human chose, relative to the project root
- `detect.grep`: one or more patterns that reliably identify this archetype (annotation,
  interface name, base class, etc.)
- `detect.path`: glob pattern for where instances live

#### Detection commands

Write commands that actually work. Test them before writing:

```bash
# Test before writing into ARCHETYPE.md
grep -rl "@RestController" . --include="*.kt" | grep -v "build\|target"
```

Write two kinds:
1. **Inventory** — find all instances
2. **Smell detection** — find instances that likely violate the rules

Smell detection is the most valuable part. Examples:

```bash
# Find controllers that don't use constructor injection (field injection smell)
grep -rl "@RestController" . --include="*.kt" | grep -v "build\|target" | \
  xargs grep -l "@Autowired"

# Find services missing @Transactional on write methods
grep -rl "@Service" . --include="*.kt" | grep -v "build\|target\|test" | \
  xargs grep -L "@Transactional"
```

#### Rules / checklist

Derive rules by reading the epitome file and 2–3 other instances. A rule captures a
structural decision that:
- Is consistently followed in recent instances
- Would matter if violated
- Can be mechanically verified

Examples of good rules:
- `[ ] Constructor injection only — no @Autowired field injection`
- `[ ] No business logic — delegates entirely to the service layer`
- `[ ] Response type declared explicitly, not ResponseEntity<*>`

Examples of bad rules (too vague):
- `[ ] Code is clean`
- `[ ] Follows best practices`

Aim for 5–10 rules.

#### Anti-patterns

Look at the older/worse instances you read in step 3. What do they do that the epitome
file avoids? Show each anti-pattern as ❌/✅ pair in the actual language of the project.

---

### 6. Update tasks.json

After writing ARCHETYPE.md for an archetype, mark it `pinned`:

```json
{ "id": "controller_rest", "folder": ".epitome/controller_rest/", "status": "pinned" }
```

After all approved archetypes are processed, update the steps:

```json
{ "id": 2, "title": "Pin epitome file for each archetype", "status": "done",
  "subtasks": [
    { "id": "2.1", "title": "controller_rest", "status": "done" },
    { "id": "2.2", "title": "service_domain", "status": "done" }
  ]
},
{ "id": 3, "title": "Review each archetype against real instances", "status": "in-progress", "subtasks": [] }
```

---

## Quality bar

Before considering an archetype done, ask:

- Does `epitome_file` point to a real, existing file in the codebase?
- Would the detection commands surface all genuine instances if run today?
- Does each rule reflect something verifiable in the code, not a preference?
- Does every anti-pattern reflect something that actually exists or existed in this codebase?
- Are related archetypes correctly linked?

---

## Output

- `.epitome/<archetype_id>/ARCHETYPE.md` created for each processed archetype
- `epitome_file` points to a real file chosen by the human
- `.epitome/tasks.json` updated: step 2 `done`, step 3 `in-progress`, archetypes `pinned`

Next step: run `epitome-review`.
