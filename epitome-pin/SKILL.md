---
name: epitome-pin
description: Step 2 of the epitome workflow — and the tool for registering a new archetype mid-development. Finds all real instances in the codebase for each archetype, presents them for the human to choose one as the canonical epitome file, then writes ARCHETYPE.md. Also used when an agent hits a code pattern with no existing archetype during feature work.
license: MIT
---

# epitome-pin

Two modes:

1. **Step-2 workflow** — after `epitome-init` approval, pin an epitome file for each new archetype
2. **Mid-development** — when implementing a feature and hitting a gap (pattern has no archetype yet), register the new one before proceeding

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Key principle

**No code is copied into `.epitome/`.** The chosen file stays where it lives in the codebase.
`ARCHETYPE.md` holds a pointer (`epitome_file`) to it. When the codebase evolves, the
epitome evolves with it — there is only one source of truth.

---

## Mode 1: Step-2 workflow pinning

### Prerequisites

- `epitome-init` has been run and the human has approved an archetype list

### Process

#### 1. Identify archetypes to pin

Scan `.epitome/` for missing or empty ARCHETYPE.md files — these are the approved archetypes
that still need pinning:

```bash
# Archetypes with no ARCHETYPE.md yet
find .epitome -maxdepth 1 -type d | grep -v "^.epitome$" | while read dir; do
  [ ! -f "$dir/ARCHETYPE.md" ] && echo "MISSING: $dir"
done

# Archetypes with ARCHETYPE.md but no epitome_file set
grep -rL "epitome_file:" .epitome/*/ARCHETYPE.md 2>/dev/null
```

If no `.epitome/` directories exist yet, the human has the approved list from `epitome-init`.
Create them as you go.

#### 2. For each archetype: find and present candidates

```bash
# Example: find candidates for service_lookup
grep -rl "@Transactional(readOnly = true)" . --include="*.kt" | grep -v "build\|test" | \
  xargs ls -t 2>/dev/null
```

Present candidates (most recently modified first):

```
Archetype: service_lookup
Found 2 candidates (most recent first):

  [1] src/.../WordCountingJobLookupService.kt   (modified 3 days ago)
  [2] src/.../WordStatisticsService.kt          (modified 1 week ago)

Which file should be the epitome for service_lookup?
(Enter 1–2, a custom path, or "skip" to defer)
```

Wait for human choice per archetype.

#### 3. Read the candidates

Before writing ARCHETYPE.md, read **3–5 real instances** to understand the actual patterns:
- Which annotations are consistently present
- How dependencies are declared
- How errors are handled
- What the structural skeleton looks like

#### 4. Write ARCHETYPE.md

Create `.epitome/<archetype_id>/ARCHETYPE.md`. Follow the [ARCHETYPE.md format spec](../spec/archetype-md-format.md) exactly, including `status: pinned`.

```yaml
---
id: service_lookup
status: pinned
epitome_file: src/main/kotlin/.../WordStatisticsService.kt
detect:
  grep:
    - "@Transactional(readOnly = true)"
    - "@Service"
  path: "src/main/kotlin/**/*Service.kt"
related:
  - endpoint_get_single
  - repository_simple
---
```

Test all detection commands before writing them.

#### 5. Quality bar

- `epitome_file` points to a real, existing file
- Detection commands find genuine instances when run today
- Each rule is falsifiable (can be mechanically checked)
- Anti-patterns reflect code that exists or has existed — not strawmen

---

## Mode 2: Mid-development pinning

Used when an agent is implementing a feature and discovers the required code pattern
has no archetype. Follow the MANIFESTO protocol to stop, agree on the new archetype,
then use this procedure to register it.

### Step A: Check if this skill is installed

```bash
ls ~/.pi/agent/skills/epitome-pin 2>/dev/null || \
ls .pi/skills/epitome-pin 2>/dev/null || \
echo "NOT_FOUND"
```

**If found**: invoke `epitome-pin <archetype_id>` for the specific archetype, then return
to the feature implementation.

**If not found**: continue with steps B–E below, and warn the user:
> ⚠ epitome-pin skill not installed. Registering archetype manually. For the full workflow
> experience, install epitome-pin: see the epitome README.

### Step B: Implement the new code first

If no existing file can serve as the epitome (the gap is the reason you stopped), implement
the minimum version of the new pattern — just enough to establish the structure. Do NOT
implement the full feature yet.

### Step C: Create the archetype

```bash
mkdir -p .epitome/<archetype_id>
```

Write `.epitome/<archetype_id>/ARCHETYPE.md`. Use `status: reviewed` (not `pinned`) —
this archetype was written from a real, just-implemented instance, so no separate review
pass is needed.

```yaml
---
id: endpoint_delete
status: reviewed
epitome_file: src/.../WordCountController.kt
detect:
  grep:
    - "@DeleteMapping"
  path: "src/main/kotlin/**/*Controller.kt"
related:
  - controller_rest
  - service_deletion
---
```

### Step D: Update MANIFESTO.md

1. Add a row to the archetypes table for the new archetype
2. If the archetype appeared in "Patterns NOT yet defined" → remove it from that list
3. If it was not in "Patterns NOT yet defined" (a previously unknown gap) → only add to table; do NOT add to "NOT yet defined"

### Step E: Commit

Once implementation is complete and tests pass, commit everything in one commit:
- Production code (new pattern implementation)
- Test code
- `.epitome/<archetype_id>/ARCHETYPE.md`
- `.epitome/MANIFESTO.md`

---

## Output (Mode 1)

- `.epitome/<archetype_id>/ARCHETYPE.md` created with `status: pinned`
- `epitome_file` points to a real file chosen by the human

Next step: run `epitome-review`.

## Output (Mode 2)

- New code committed
- `.epitome/<archetype_id>/ARCHETYPE.md` created with `status: reviewed`
- `MANIFESTO.md` updated
- Return to full feature implementation
