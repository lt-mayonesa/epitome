# ARCHETYPE.md Format

Every archetype directory contains an `ARCHETYPE.md` file. It is the machine- and
human-readable contract for that pattern. It will be fed to an agent to guide both
code generation (implementing a new feature) and code review (finding legacy deviations).

---

## Structure

````markdown
---
id: <archetype_id>
status: pinned | reviewed
epitome_file: <relative path to the canonical real file in the codebase>
detect:
  grep:
    - "pattern1"
    - "pattern2"
  path: "glob/pattern/**/*ClassName.ext"
related:
  - other_archetype_id
---

## What it is

One paragraph. What this pattern is, why it exists, and what problem it solves.
Written for a developer encountering it for the first time.

## Epitome

[`<filename>`](<relative path to epitome_file from repo root>)

One sentence explaining why this specific file was chosen as the canonical example.

## How to detect it

```bash
# Find all instances of this archetype
<detection command>

# Find instances that may violate the rules (smell detection)
<smell detection command>
```

## Rules / checklist

- [ ] Rule 1
- [ ] Rule 2
- [ ] Rule 3

## Anti-patterns

```
// ❌ What the wrong version looks like, with a comment explaining why
...

// ✅ What the right version looks like
...
```

## Related archetypes

- [`other_archetype`](../other_archetype/ARCHETYPE.md) — one-line explanation of the relationship
````

---

## Frontmatter fields

| Field | Required | Description |
|-------|----------|-------------|
| `id` | Yes | Matches the directory name exactly |
| `status` | Yes | Workflow status: `pinned` (file chosen, rules written) or `reviewed` (compared against real instances) |
| `epitome_file` | Yes | Relative path (from repo root) to the canonical real file in the codebase |
| `detect.grep` | Yes | One or more grep patterns that identify instances of this archetype |
| `detect.path` | Yes | Glob pattern for where instances live in the codebase |
| `related` | No | List of archetype IDs this one interacts with |

### `status` lifecycle

| Value | Set by | Meaning |
|-------|--------|---------|
| `pinned` | `epitome-pin` | Epitome file chosen, ARCHETYPE.md written — ready for review |
| `reviewed` | `epitome-review` | Compared against all real instances, rules and anti-patterns finalised |

When an archetype is registered mid-development (during feature implementation, not as part
of the step-2 workflow), set `status: reviewed` directly — it was written from a real
implementation and does not need a separate review pass.

---

## Writing guidelines

**Epitome section** — name the file and give one sentence explaining the choice.
Good choices: the most recently written instance, the cleanest implementation, the one
a new team member should read first.

**What it is** — avoid restating the archetype name. Explain the *why*, not just the *what*.

**Detection commands** — write two kinds:
1. Commands that *find* all instances (for inventory)
2. Commands that *find violations* (for drift detection) — these are the most valuable

Test every detection command before writing it in.

**Rules / checklist** — write them so a reviewer can mechanically verify each one.
Each rule should be falsifiable: avoid "code should be clean", prefer "no `@Component` annotation".
Aim for 5–10 rules.

**Anti-patterns** — always show ❌ and ✅ side by side. The ❌ should look like real
code your codebase might actually contain, not a strawman.

**Related archetypes** — explain the *direction* of the relationship (calls, tested by,
published to, registered by).
