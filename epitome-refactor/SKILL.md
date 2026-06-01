---
name: epitome-refactor
description: Step 5 of the epitome workflow. Runs smell-detection commands from all ARCHETYPE.md files to find drift from the defined archetypes, presents violations to the human, and applies approved fixes to bring the codebase toward its epitome state. Run periodically or on demand.
license: MIT
---

# epitome-refactor

Runs **step 5** of the epitome workflow: detect drift from the archetypes and fix it.

This skill is **periodic** — it can be run at any time after step 2 is complete. It does
not need all archetypes to be reviewed; it works with whatever ARCHETYPE.md files exist.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Mental model

Archetypes define what ideal code looks like. `epitome-refactor` is the enforcement arm:
it finds files that drift from the ideal and brings them into alignment.

The smell-detection commands in each `ARCHETYPE.md` are the engine. They find violations
mechanically. The human approves each fix before it is applied.

---

## Prerequisites

- At least one archetype has status `pinned` or `reviewed` in `.epitome/tasks.json`
- Each `ARCHETYPE.md` has smell-detection commands in the **How to detect it** section

---

## Process

### 1. Collect archetypes to check

```bash
cat .epitome/tasks.json
```

By default, check all archetypes with status `pinned` or `reviewed`.
The human can restrict to a specific archetype: `epitome-refactor controller_rest`.

---

### 2. Run all smell-detection commands

For each archetype, extract and run the smell-detection command from `ARCHETYPE.md`:

```bash
cat .epitome/<archetype_id>/ARCHETYPE.md
```

Run the smell-detection commands (the ones under "Find instances that may violate the rules"):

```bash
# Example smell detection
grep -rl "@RestController" . --include="*.kt" | grep -v "build\|target" | \
  xargs grep -l "@Autowired"
```

Record every file returned by any smell-detection command, along with which archetype
and which rule it violates.

---

### 3. Read the epitome file for each archetype with violations

Before proposing fixes, read the canonical example to understand what the fixed version
should look like:

```bash
# Get epitome_file path from frontmatter
grep "epitome_file:" .epitome/<archetype_id>/ARCHETYPE.md | sed 's/.*epitome_file: //'

cat <epitome_file>
```

---

### 4. Read each violating file

Read the full content of each file that triggered a smell. Understand:
- Which rule(s) it violates
- What change is needed to comply
- Whether the change is safe (no behaviour change expected)

---

### 5. Present violations to the human

Group violations by archetype. For each violation:

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

Present violations one at a time. Wait for a response before proceeding to the next.

Response options:
- `Y` or Enter — apply the fix
- `n` — skip this violation, keep the file as-is
- `skip-file` — skip all violations in this file for this run
- `skip-archetype` — skip all remaining violations for this archetype

---

### 6. Apply approved fixes

For each approved fix:
1. Make the minimum change needed to comply with the rule
2. Do not reformat unrelated code
3. Do not rename things unless the rule explicitly requires it
4. Preserve all comments and business logic

If a fix requires understanding more context (imports, base classes, etc.), read the
relevant files before making the change.

After applying, briefly confirm what was changed:
```
✅ Fixed: removed @Autowired field injection, converted to constructor injection
   File: src/main/kotlin/.../ProductController.kt
```

---

### 7. Handle conflicts and edge cases

**If a fix would change behaviour:**
Explain the risk clearly and ask for explicit confirmation before applying.

**If a file has been intentionally exempted:**
If a comment like `// epitome-ignore: <reason>` appears near the violation, skip it
and do not propose a fix.

**If the violation is in generated code:**
Skip generated files. They should not be manually edited.

**If multiple rules are violated in the same file:**
Fix them all in one pass rather than touching the file multiple times.

---

### 8. Report summary

After processing all violations, show a summary:

```
epitome-refactor summary
─────────────────────────────────────────────
Archetypes checked:  4
Files scanned:       23
Violations found:    11
  Fixed:             7
  Skipped:           2
  Deferred:          2

Fixed files:
  src/.../ProductController.kt       (controller_rest: constructor injection)
  src/.../OrderService.kt            (service_domain: @Transactional on write)
  ...

Skipped:
  src/.../LegacyAdapter.kt           (controller_rest: intentional deviation noted)
  ...
```

---

### 9. Optionally update ARCHETYPE.md

If during this run you found violations the smell-detection commands did NOT catch
(because you noticed them while reading files), offer to add a new smell-detection command:

```
I found a violation that the current smell-detection command missed:
  <description>

Add a new smell-detection command to controller_rest/ARCHETYPE.md? [Y/n]
```

Only add if the human confirms.

---

## Running periodically

`epitome-refactor` is designed to be run regularly — e.g. weekly, or before merging a
large feature. Each run finds *new* drift introduced since the last run.

It does not need to be run on the full codebase each time. You can scope it:

```
# Check only controller_rest
epitome-refactor controller_rest

# Check files modified in the last 7 days
epitome-refactor --since 7d
```

When scoping by `--since`, filter candidate files by modification time:

```bash
find . -type f -name "*.kt" -newer $(date -d "7 days ago" +%Y-%m-%d 2>/dev/null || \
  date -v-7d +%Y-%m-%d) | grep -v "build\|target"
```

---

## Output

- Modified source files (violations fixed)
- Optional: `.epitome/<archetype_id>/ARCHETYPE.md` updated (new smell-detection commands)
- Summary printed to output
- `.epitome/tasks.json` NOT modified (this skill is stateless / periodic)
