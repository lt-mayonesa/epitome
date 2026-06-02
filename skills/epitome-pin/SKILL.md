---
name: epitome-pin
description: Step 2 of the epitome workflow — and the tool for registering a new archetype mid-development. Works through archetypes with status:discovered one at a time, analyzing candidate files and letting the human choose the epitome file or reject the archetype entirely. Also used when an agent hits a code pattern with no existing archetype during feature work.
license: MIT
---

# epitome-pin

Two modes:

1. **Step-2 workflow** — after `epitome-init`, work through `status: discovered` archetypes
   one at a time to select the canonical epitome file for each
2. **Mid-development** — when implementing a feature and hitting a gap (pattern has no
   archetype yet), register the new one before proceeding

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

- `epitome-init` has been run and stub `ARCHETYPE.md` files exist with `status: discovered`

### Process

#### 1. Find archetypes to pin

```bash
grep -rl "status: discovered" .epitome/*/ARCHETYPE.md 2>/dev/null | sort
```

Work through them **one at a time** in the order returned. Do not batch-present all at once.

---

#### 2. For each archetype: present context, find candidates, analyze, ask

##### 2a. Show the archetype context

Read the stub ARCHETYPE.md and display:
- The archetype `id`
- The `## Reasons` section (why it was detected, instance count, representative files)

##### 2b. Find all candidate files

Use the `detect` patterns from the stub to find real instances:

```bash
# Example using the detect.grep and detect.path from the stub
grep -rl "<detect.grep[0]>" . --include="<extension>" | \
  grep -v "build\|target\|test" | \
  xargs ls -t 2>/dev/null
```

List candidates most-recently-modified first. Show the 5–8 most recent.

##### 2c. Analyze each candidate

For each candidate file, run a focused analysis covering two dimensions:

**Code quality signals** — extract structural markers that indicate cleanliness:

```bash
# Line count
wc -l <file>

# Complexity proxies: nested control flow, long methods
grep -c "if \|else \|switch \|for \|while \|catch " <file>

# TODO / FIXME / HACK markers
grep -n "TODO\|FIXME\|HACK\|XXX" <file>

# Dependency count (constructor params or field injections)
grep -c "@Autowired\|private final\|private val" <file>

# Key structural lines (class declaration, annotations, method signatures)
grep -n "^@\|^public \|^  public \|^class \|^  @" <file> | head -30
```

**Project guidelines alignment** — check the candidate against the project's own rules.
Read `AGENTS.md` or `CLAUDE.md` if they exist, then verify the candidate against the
rules that apply to this archetype type. Report which rules it follows and which it
violates. Examples of what to check (adapt to the actual project guidelines):

```bash
# Does it use the project's preferred annotation style?
# Does it follow the naming conventions?
# Does it avoid banned patterns (e.g. @Autowired field injection instead of constructor)?
# Is it in the correct package/module?
grep -n "<project-specific-marker>" <file>
```

##### 2d. Present the analysis

Show one block per candidate. Keep it scannable — use a consistent layout:

```
────────────────────────────────────────────────────────
Archetype: service_lookup  (3 of 23)
────────────────────────────────────────────────────────

Why detected: @Transactional(readOnly = true) at class level in Service files
Instances: 47 files

Candidates (most recent first):

[1] wms/wms-logic/src/main/java/com/picnic/wms/stock/StockLookupService.java
    Modified: 3 days ago  |  Lines: 82  |  Complexity proxies: 4
    TODOs: none
    Quality: ✓ constructor injection  ✓ final class  ✓ no @Autowired
    Guidelines: ✓ @Secured present  ✓ LOG field  ✗ missing @Transactional(readOnly=true) at class level

[2] wms/wms-logic/src/main/java/com/picnic/wms/order/OrderLookupService.java
    Modified: 2 weeks ago  |  Lines: 134  |  Complexity proxies: 11
    TODOs: 1 (line 67: "TODO: extract to separate method")
    Quality: ✓ constructor injection  ✓ final class  ✗ one @Autowired field
    Guidelines: ✓ @Secured present  ✓ LOG field  ✓ @Transactional(readOnly=true) at class level

[3] ...

Options:
  1–N  Pick this file as the epitome
  s    Skip — defer this archetype (leave status: discovered)
  r    Reject — this is not a real archetype (explain why when prompted)
```

Wait for the human's answer before proceeding to the next archetype.

##### 2e. Handle the response

**On pick (number)**:
- Read 3–5 real instances to understand consistent patterns
- Write full `ARCHETYPE.md` replacing the stub, with `status: pinned`
- Confirm: "✓ Pinned `<archetype_id>` → `<epitome_file>`"
- Move to next archetype

**On skip (`s`)**:
- Leave `ARCHETYPE.md` unchanged (`status: discovered`)
- Confirm: "⟳ Skipped `<archetype_id>` — still discovered"
- Move to next archetype

**On reject (`r`)**:
- Ask for a one-line reason: "Why reject? (e.g. 'covered by service', 'not a real seam')"
- Update `ARCHETYPE.md` — change `status: discovered` to `status: rejected` and append the reason
- Confirm: "✗ Rejected `<archetype_id>`"
- Move to next archetype

The rejected ARCHETYPE.md keeps its `detect` patterns and `## Reasons` section so future
`epitome-init` runs know to skip it. Add a `## Rejection reason` section with the explanation.

---

#### 3. After all archetypes processed

Report a summary:

```
epitome-pin complete.

  Pinned:   15 archetypes
  Skipped:   3 archetypes (run epitome-pin again to continue)
  Rejected:  5 archetypes

Next step: run epitome-review on the pinned archetypes.
```

---

#### 4. Write full ARCHETYPE.md (for pinned archetypes)

Before writing, read **3–5 real instances** of the archetype to understand consistent patterns:
- Which annotations are always present vs sometimes present
- How dependencies are declared
- How errors are handled
- What the structural skeleton looks like

Then write `.epitome/<archetype_id>/ARCHETYPE.md` following the
[ARCHETYPE.md format spec](../spec/archetype-md-format.md) exactly.

```yaml
---
id: service_lookup
status: pinned
epitome_file: wms/wms-logic/src/main/java/com/picnic/wms/stock/StockLookupService.java
detect:
  grep:
    - "@Transactional(readOnly = true)"
  path: "wms/wms-logic/src/main/java/**/*Service.java"
related:
  - endpoint_get_single
  - repository_jooq
---
```

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

- `.epitome/<archetype_id>/ARCHETYPE.md` updated with `status: pinned` for each picked archetype
- `.epitome/<archetype_id>/ARCHETYPE.md` updated with `status: rejected` for each rejected archetype
- Skipped archetypes unchanged (`status: discovered`)

Next step: run `epitome-review` on the pinned archetypes.

## Output (Mode 2)

- New code committed
- `.epitome/<archetype_id>/ARCHETYPE.md` created with `status: reviewed`
- `MANIFESTO.md` updated
- Return to full feature implementation
