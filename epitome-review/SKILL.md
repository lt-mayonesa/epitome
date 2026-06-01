---
name: epitome-review
description: Step 3 of the epitome workflow. Reads the pinned epitome file for each archetype, finds all other instances in the codebase, identifies deviations from the archetype rules, presents them for human approval, then applies approved changes to ARCHETYPE.md (rules, anti-patterns, or epitome_file pointer).
license: MIT
---

# epitome-review

Runs **step 3** of the epitome workflow: compare each archetype's pinned epitome against
real codebase instances, surface deviations, and update the archetype to reflect what the
codebase actually considers ideal.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Mental model

The pinned `epitome_file` is the current canonical example. Other instances in the codebase
are compared against it. The most recently modified files represent current team thinking.

**When recent files consistently differ from the epitome → the rules likely need updating,
or a better epitome file should be chosen.**
**When only old files deviate → the old files are candidates for future refactoring.**

---

## Process

### 1. Identify the archetype to review

If the user named an archetype, use it. Otherwise, check `.epitome/tasks.json` for the
first archetype with status `pinned`.

```bash
cat .epitome/tasks.json
```

Read the archetype's definition:

```bash
cat .epitome/<archetype_id>/ARCHETYPE.md
```

Note the `epitome_file` path — this is the canonical example. Read it:

```bash
cat <epitome_file>
```

---

### 2. Find all other instances

Use the `detect` patterns from the archetype's frontmatter:

```bash
# Find by path pattern
find . -path "*/build" -prune -o -path "*/target" -prune -o \
  -path "*/node_modules" -prune -o \
  -type f -name "<pattern from detect.path>" -print | grep -v "build\|target"

# Narrow to files matching the grep signature
grep -rl "<pattern from detect.grep>" . \
  --include="<file extension>" | grep -v "build\|target\|node_modules"
```

Exclude the `epitome_file` itself from the comparison set.

---

### 3. Rank instances

**Primary signal — Recency**: most recently modified files first.

```bash
find . -type f -name "<pattern>" | grep -v "build\|target" | \
  xargs ls -t 2>/dev/null
```

**Secondary signal — Frequency**: patterns that appear across many files outweigh
those in one or two.

**Combined**: recent AND widespread = strongest signal for updating the archetype.
Only recent but rare = emerging preference. Only old = legacy debt.

---

### 4. Read the top instances

Read the **top 5–10 most recent** instances. Focus on:
- Class/file-level annotations and structure
- Dependency declaration style
- Method signatures and patterns
- Error handling approach
- Import style

Also read **2–3 older instances** to understand what changed.

---

### 5. Identify deviations

Compare each instance against the archetype's **Rules / checklist**. A deviation is any
rule not followed in a significant number of instances.

Categorise each deviation:

| Category | Meaning | Default action |
|----------|---------|----------------|
| **Archetype wrong** | Recent + frequent instances all deviate in the same way | Update rules + possibly re-pin |
| **Legacy debt** | Old instances deviate, recent ones follow the rule | Keep rule, note as tech debt |
| **Emerging pattern** | Only the most recent files deviate, in the same direction | Update rules, flag as new direction |
| **One-off** | Single file deviates idiosyncratically | Keep rule, skip this file |

---

### 6. Present deviations to the human

For each deviation:

```
DEVIATION: <short title>
Category:  <Archetype wrong | Legacy debt | Emerging pattern | One-off>
Affects:   <N files> (<M recent, K old>)
Epitome says:       <what the current epitome_file / rules specify>
Codebase does:      <what the instances actually do>
Default action:     <Update rules | Re-pin to better file | Keep rule | Skip>
```

If the default action is "Re-pin", suggest the specific file that better represents
the current pattern.

Example presentation:

```
Found 3 deviations in service_domain (reviewed 8 instances):

[1] Logging style                         DEFAULT: Update rules
    Category: Archetype wrong
    Affects: 6 files (5 recent, 1 old)
    Epitome says: logger created via companion object
    Codebase does: logger created as private val in class body

[2] @Transactional on write methods       DEFAULT: Keep rule (add as anti-pattern)
    Category: Legacy debt
    Affects: 4 files (0 recent, 4 old)
    Epitome says: write methods annotated @Transactional
    Codebase does: some old files omit @Transactional on writes

[3] Better epitome candidate              DEFAULT: Re-pin
    Category: Archetype wrong
    Affects: all instances
    Epitome says: WordCountingService (written 3 weeks ago)
    Codebase does: WordStatisticsService is simpler, more recent, better demonstrates the pattern

Proceed with defaults? Or specify changes:
```

Wait for explicit human confirmation before applying anything.

---

### 7. Apply approved changes

**If "Update rules":**
- Update the relevant rules in **Rules / checklist** of `ARCHETYPE.md`
- Add the old (wrong) pattern as an **Anti-pattern** with ❌/✅ if not already there
- Update smell-detection commands if they need to change

**If "Re-pin":**
- Update `epitome_file` in `ARCHETYPE.md` frontmatter to the new path
- Update the **Epitome** section (filename link and explanation sentence)
- Optionally update rules if the new file shows slightly different patterns

**If "Keep rule (add as anti-pattern)":**
- Add the observed bad pattern to **Anti-patterns** with ❌
- Add a smell-detection command if not already present

**If "Keep rule" (no change):**
- No file changes required

Do not change anything not explicitly approved.

---

### 8. Update tasks.json

Mark the archetype `reviewed`:

```json
{ "id": "controller_rest", "folder": ".epitome/controller_rest/", "status": "reviewed" }
```

Add a subtask entry under step 3:

```json
{ "id": "3.1", "title": "controller_rest", "status": "done" }
```

When all archetypes in step 3 are reviewed, mark step 3 `done` and step 4 `in-progress`.

---

## Quality bar

Before finishing a review:

- [ ] Every deviation was categorised with evidence (file count + recency)
- [ ] Human explicitly approved or rejected each proposed change
- [ ] `epitome_file` points to the best available example in the current codebase
- [ ] Every new anti-pattern has both ❌ and ✅ examples
- [ ] Detection commands would catch newly identified smells
- [ ] tasks.json updated

---

## Output

- `.epitome/<archetype_id>/ARCHETYPE.md` updated (rules, anti-patterns, possibly `epitome_file`)
- `.epitome/tasks.json` updated: archetype marked `reviewed`

Next step: run `epitome-manifesto` once all archetypes are reviewed.
