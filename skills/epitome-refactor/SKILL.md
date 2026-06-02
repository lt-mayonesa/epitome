---
name: epitome-refactor
description: Step 5 of the epitome workflow. Runs smell-detection commands from all ARCHETYPE.md files to find drift from the defined archetypes, presents violations to the human, and applies approved fixes. Run periodically or on demand after at least one archetype has status pinned or reviewed.
license: MIT
---

# epitome-refactor

Runs **step 5** of the epitome workflow: detect drift from the archetypes and fix it.

This skill is **periodic** — run at any time after step 2. Works with whatever ARCHETYPE.md
files exist regardless of status.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Mental model

Archetypes define what ideal code looks like. `epitome-refactor` is the enforcement arm:
it finds files that drift from the ideal and brings them into alignment.

The smell-detection commands in each `ARCHETYPE.md` are the engine. The human approves
each fix before it is applied.

---

## Prerequisites

At least one `.epitome/*/ARCHETYPE.md` exists with `status: pinned` or `status: reviewed`.

---

## Process

### 1. Collect archetypes to check

```bash
# All archetypes with status pinned or reviewed
grep -rl "status: pinned\|status: reviewed" .epitome/*/ARCHETYPE.md 2>/dev/null

# Or check a specific one: epitome-refactor controller_rest
```

By default, check all. The human can restrict to a specific archetype.

---

### 2. Run smell-detection commands

For each archetype, extract and run the smell-detection commands (the "Find instances that
may violate the rules" sections from `ARCHETYPE.md`).

Record every file returned, along with which archetype and rule it violates.

---

### 3. Read the epitome file for each archetype with violations

```bash
grep "epitome_file:" .epitome/<archetype_id>/ARCHETYPE.md | sed 's/.*epitome_file: //'
cat <epitome_file>
```

Read the canonical example to understand what the fixed version should look like.

---

### 4. Read each violating file

Understand which rule(s) it violates and what change is needed.

---

### 5. Present violations one at a time

```
VIOLATION: <filename>
Archetype: <archetype_id>
Rule:      <the rule being violated>
Epitome:   <epitome_file> — shows the correct pattern

Current code:
  <relevant snippet showing the violation>

Proposed fix:
  <relevant snippet showing the corrected version>

Apply? [Y/n/skip-file/skip-archetype]
```

Responses:
- `Y` or Enter — apply fix
- `n` — skip this violation
- `skip-file` — skip all violations in this file for this run
- `skip-archetype` — skip all remaining violations for this archetype

---

### 6. Apply approved fixes

For each approved fix:
- Make the minimum change needed to comply with the rule
- Do not reformat unrelated code
- Do not rename things unless the rule explicitly requires it
- Preserve all comments and business logic
- If a fix requires understanding imports or base classes, read the relevant files first

Confirm after each:
```
✅ Fixed: <short description>
   File: <path>
```

---

### 7. Handle edge cases

**Fix would change behaviour** → explain risk, require explicit confirmation before applying.

**`// epitome-ignore: <reason>` comment near violation** → skip, do not propose fix.

**Generated code** → skip. Do not manually edit generated files.

**Multiple rules violated in same file** → fix all in one pass.

---

### 8. Summary report

```
epitome-refactor summary
─────────────────────────────────────────────
Archetypes checked:  <N>
Files scanned:       <N>
Violations found:    <N>
  Fixed:             <N>
  Skipped:           <N>

Fixed files:
  src/.../ProductController.kt  (controller_rest: constructor injection)
  ...
```

---

### 9. Optionally improve smell detection

If you found violations the current smell-detection commands missed, offer to add them:

```
I found a violation the smell-detection command missed:
  <description>
Add a new smell-detection command to <archetype_id>/ARCHETYPE.md? [Y/n]
```

---

## Running periodically

```bash
# Scope to one archetype
epitome-refactor service_lookup

# Scope to recently modified files (adapt date syntax for OS)
find . -type f -name "*.kt" -newer <reference-file> | grep -v "build\|target"
```

---

## Output

- Modified source files (violations fixed)
- Optional: `.epitome/<archetype_id>/ARCHETYPE.md` updated (new smell-detection commands)
- Summary printed
