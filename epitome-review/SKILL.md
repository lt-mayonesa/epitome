---
name: epitome-review
description: Step 3 of the epitome workflow. Reads the pinned epitome file for each archetype (status pinned), finds all other instances in the codebase, identifies deviations from the archetype rules, presents them for human approval, then applies approved changes to ARCHETYPE.md and updates status to reviewed.
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

**When recent files consistently differ from the epitome → rules likely need updating,
or a better epitome file should be chosen.**
**When only old files deviate → the old files are candidates for future refactoring.**

---

## Process

### 1. Identify archetypes to review

Scan for archetypes with `status: pinned`:

```bash
grep -rl "status: pinned" .epitome/*/ARCHETYPE.md 2>/dev/null
```

If the user named a specific archetype, use that. Otherwise process all `status: pinned`
archetypes in order.

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

### 3. Rank by recency + frequency

```bash
find . -type f -name "<pattern>" | grep -v "build\|target" | \
  xargs ls -t 2>/dev/null
```

**Recent + widespread** = strongest signal for updating the archetype.
**Only old files deviate** = legacy debt, keep rule.
**Only the most recent files deviate** = emerging direction, update archetype.

---

### 4. Read the top instances

Read **5–10 most recent** instances. Focus on:
- Class-level annotations and structure
- Dependency declaration style
- Error handling approach
- Import style

Also read **2–3 older instances** to understand what changed over time.

---

### 5. Identify and categorise deviations

| Category | Meaning | Default action |
|----------|---------|----------------|
| **Archetype wrong** | Recent + frequent instances all deviate the same way | Update rules + possibly re-pin |
| **Legacy debt** | Old instances deviate; recent ones follow the rule | Keep rule, note as tech debt |
| **Emerging pattern** | Only the most recent files deviate, same direction | Update rules, flag as new direction |
| **One-off** | Single file deviates idiosyncratically | Keep rule, skip this file |

---

### 6. Present deviations

```
DEVIATION: <short title>
Category:  Archetype wrong | Legacy debt | Emerging pattern | One-off
Affects:   <N files> (<M recent, K old>)
Epitome says:    <what the current epitome_file / rules specify>
Codebase does:   <what the instances actually do>
Default action:  Update rules | Re-pin | Keep rule | Skip
```

If "Re-pin", name the specific file that better represents the current pattern.

Wait for explicit human confirmation before applying anything.

---

### 7. Apply approved changes

**If "Update rules":**
- Update rules in ARCHETYPE.md
- Add old pattern as anti-pattern (❌/✅) if not already there
- Update smell-detection commands if needed

**If "Re-pin":**
- Update `epitome_file` in frontmatter
- Update the **Epitome** section (filename link + explanation sentence)

**If "Keep rule (add as anti-pattern)":**
- Add ❌ to Anti-patterns
- Add smell-detection command if not already present

**If "Keep rule":**
- No changes

Do not change anything not explicitly approved.

---

### 8. Mark as reviewed

Update `status` in the archetype's frontmatter:

```yaml
---
id: service_lookup
status: reviewed   # ← was: pinned
epitome_file: ...
```

When all `status: pinned` archetypes are reviewed, proceed to `epitome-manifesto`.

---

## Quality bar

- [ ] Every deviation categorised with evidence (file count + recency)
- [ ] Human explicitly approved or rejected each proposed change
- [ ] `epitome_file` points to the best current example
- [ ] Every new anti-pattern has both ❌ and ✅
- [ ] Detection commands catch newly identified smells
- [ ] All processed archetypes now have `status: reviewed`

---

## Output

- `.epitome/<archetype_id>/ARCHETYPE.md` updated (rules, anti-patterns, possibly `epitome_file`, `status: reviewed`)

Next step: once all archetypes are `status: reviewed`, run `epitome-manifesto`.
