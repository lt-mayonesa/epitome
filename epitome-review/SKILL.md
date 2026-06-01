---
name: epitome-review
description: Step 3 of the epitome workflow. Finds all codebase instances of a given archetype, ranks them by recency and frequency, identifies deviations from the archetype definition, presents them to the human with a default recommendation, then applies approved changes to the ARCHETYPE.md and example files.
license: MIT
---

# epitome-review

Runs **step 3** of the epitome workflow: compare the archetype definition against real
codebase instances, surface deviations, and update the archetype to reflect what the
codebase actually considers ideal.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Mental model

The most recently modified files in the codebase are the closest thing to "current team
thinking". Widely-present patterns are the de-facto standard. Deviations found in old
files may be legacy debt; deviations found in recent files may mean the archetype is wrong.

**When recent files deviate from the archetype → the archetype likely needs updating.**
**When only old files deviate → the old files are candidates for future refactoring.**

---

## Process

### 1. Identify the archetype to review

If the user named an archetype, use it. Otherwise, check `.epitome/tasks.json` for the
first archetype in step 3 with status `in-progress` or no status.

```bash
cat .epitome/tasks.json
```

Read the archetype's definition:

```bash
cat .epitome/<archetype_id>/ARCHETYPE.md
```

---

### 2. Find all instances in the codebase

Use the `detect` patterns from the archetype's frontmatter to locate instances.

```bash
# From detect.path glob — find candidate files
find . -path "*/target" -prune -o \
  -path "*/node_modules" -prune -o \
  -type f -name "<pattern from detect.path>" -print | grep -v target

# From detect.grep — narrow to files actually matching the structural signature
grep -rl "<pattern from detect.grep>" . \
  --include="<file pattern>" | grep -v target | grep -v node_modules
```

Intersect both: files that match the path pattern **and** contain the grep signature.
These are the confirmed instances.

---

### 3. Rank instances

Rank by two signals combined:

**Signal A — Recency** (primary): most recently modified files first.
These represent current team thinking and carry the most weight.

```bash
# Sort confirmed instances by modification time, most recent first
find . -path "*/target" -prune -o -type f -name "<pattern>" -print | \
  grep -v target | xargs ls -t 2>/dev/null
```

**Signal B — Frequency** (secondary): patterns that appear in many files outweigh
patterns that appear in one or two.

Compute frequency by scanning all instances for each structural element (annotation,
method signature shape, field declaration style, etc.) and counting occurrences.

```bash
# Example: count how many service test files use field-level mocks vs @BeforeEach
grep -rl "private final.*= mock()" . --include="*ServiceTest.java" | grep -v target | wc -l
grep -rl "@BeforeEach" . --include="*ServiceTest.java" | grep -v target | wc -l
```

**Combined rank**: a pattern that is both recent AND frequent is the strongest signal
for what the epitome should say. A pattern that is only frequent but old is legacy.
A pattern that is only recent but rare is an emerging preference.

---

### 4. Read the top instances

Read the **top 5–10 most recent** instances in full. For large files, focus on:
- Class-level annotations and structure
- Field declarations
- Constructor shape
- Method signatures and annotations
- Test method naming (for test archetypes)

Also read **2–3 older instances** to understand what changed over time.

---

### 5. Identify deviations

Compare each instance against the archetype's **Rules / checklist**. A deviation is any
rule that is not followed in a significant number of instances.

Categorise each deviation:

| Category | Meaning | Default action |
|----------|---------|----------------|
| **Archetype wrong** | Recent + frequent instances all deviate in the same way | Update archetype to match |
| **Legacy debt** | Old instances deviate, recent ones follow the rule | Keep rule, note as tech debt |
| **Emerging pattern** | Only the most recent files deviate, but in the same way | Update archetype, flag as new direction |
| **One-off** | Single file deviates in an idiosyncratic way | Keep rule, skip this file |

For each deviation, also determine:
- How many files are affected
- Whether the most recent files show the deviation or follow the rule

---

### 6. Present deviations to the human

Show a structured summary. For each deviation:

```
DEVIATION: <short title>
Category:  <Archetype wrong | Legacy debt | Emerging pattern | One-off>
Affects:   <N files> (<M recent, K old>)
Current archetype says: <what the archetype currently specifies>
Codebase does:          <what the instances actually do>
Default action:         <Update archetype | Keep rule | Skip>
```

Then list all deviations with their default actions, and ask the human to confirm,
change, or exclude any of them before applying.

Example presentation:

```
Found 4 deviations in test_service (reviewed 15 instances):

[1] Mock initialization style            DEFAULT: Update archetype
    Category: Archetype wrong
    Affects: 12 files (10 recent, 2 old)
    Archetype says: mocks created in @BeforeEach
    Codebase does:  private final X x = mock() at field level

[2] reset() in @BeforeEach               DEFAULT: Keep rule (add as anti-pattern)
    Category: Legacy debt
    Affects: 6 files (1 recent, 5 old)
    Archetype says: (not mentioned)
    Codebase does:  reset() called to clear shared mock state

[3] @DisplayName missing on some tests   DEFAULT: Keep rule
    Category: Legacy debt
    Affects: 3 files (0 recent, 3 old)
    Archetype says: every test must have @DisplayName
    Codebase does:  some old tests omit it

[4] Test method naming                   DEFAULT: Update archetype
    Category: Archetype wrong
    Affects: 9 files (8 recent, 1 old)
    Archetype says: methodName + sequential number (e.g. release1)
    Codebase does:  descriptive camelCase (e.g. releasePublishesEvent)

Proceed with defaults? Or specify which to change:
```

Wait for human confirmation before applying any changes.

---

### 7. Apply approved changes

For each approved change, update the relevant part of the archetype:

**If "Update archetype":**
- Update the example code file(s) to reflect the pattern found in recent instances
- Update the **Rules / checklist** to match the new pattern
- Add the old (wrong) pattern as an **Anti-pattern** with ❌/✅ pair if not already there
- Update detection commands if the smell-detection grep needs to change

**If "Keep rule (add as anti-pattern)":**
- Add the observed bad pattern to the **Anti-patterns** section with ❌ if not already there
- Add a smell-detection command to **How to detect it** if not already there

**If "Keep rule" (no change needed):**
- No file changes required

Do not change anything that was not explicitly approved.

---

### 8. Update tasks.json

After applying all changes, mark the archetype as reviewed in `.epitome/tasks.json`:

```json
{
  "id": "3.<N>",
  "title": "<archetype_id>",
  "status": "done"
}
```

If all archetypes in step 3 are done, mark step 3 as `done` and step 4 as `in-progress`.

---

## Quality bar

Before finishing a review:

- [ ] Every deviation was categorised with evidence (file count + recency)
- [ ] Human explicitly approved or rejected each proposed change
- [ ] Example code now matches what the most recent codebase instances actually do
- [ ] Every new anti-pattern has both ❌ and ✅ code examples
- [ ] Detection commands would catch the newly identified smells
- [ ] tasks.json updated

---

## Output

- `.epitome/<archetype_id>/ARCHETYPE.md` updated
- `.epitome/<archetype_id>/` example files updated (if example needed changing)
- `.epitome/tasks.json` updated with archetype marked done
