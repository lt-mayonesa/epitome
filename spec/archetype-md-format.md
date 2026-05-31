# ARCHETYPE.md Format

Every archetype directory contains an `ARCHETYPE.md` file. It is the machine- and
human-readable contract for that pattern. It will be fed to an agent to guide both
code generation (implementing a new feature) and code review (finding legacy deviations).

---

## Structure

````markdown
---
id: <archetype_id>
detect:
  grep:
    - "pattern1"
    - "pattern2"
  path: "glob/pattern/**/*ClassName.java"
related:
  - other_archetype_id
  - another_archetype_id
---

## What it is

One paragraph. What this pattern is, why it exists, and what problem it solves.
Written for a developer encountering it for the first time.

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
| `detect.grep` | Yes | One or more grep patterns that identify instances of this archetype |
| `detect.path` | Yes | Glob pattern for where instances live in the codebase |
| `related` | No | List of archetype IDs this one interacts with |

---

## Writing guidelines

**What it is** — avoid restating the archetype name. Explain the *why*, not just the *what*.

**Detection commands** — write two kinds:
1. Commands that *find* all instances (for inventory)
2. Commands that *find violations* (for drift detection) — these are the most valuable

**Rules / checklist** — write them so a reviewer can mechanically verify each one.
Each rule should be falsifiable: avoid "code should be clean", prefer "no `@Component` annotation".

**Anti-patterns** — always show the ❌ and ✅ side by side. The ❌ should look like real
legacy code your codebase might actually contain, not a strawman.

**Related archetypes** — explain the *direction* of the relationship (calls, is registered by,
publishes to, is tested by).
