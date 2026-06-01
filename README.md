# epitome

> Code once, steer later — or steer along the way.

**epitome** is a framework for defining and enforcing code standards through concrete, living
examples and agent-executable workflows. Instead of writing static style guides that nobody
reads, you designate *archetypes* — real files from your own codebase that show what ideal
code looks like — and let agents periodically find and fix deviations.

---

## How it works

An **epitome** for your project lives in a `.epitome/` directory at the root of your
repository. It holds:

- **Archetypes** — one directory per recurring pattern, each with an `ARCHETYPE.md`
  that points to a real canonical file in your codebase plus detection commands, rules,
  and anti-patterns
- **`MANIFESTO.md`** — the archetype-first development protocol: what agents do when a
  pattern has no archetype yet, plus a table of all defined archetypes

```
.epitome/
├── MANIFESTO.md
├── controller_rest/
│   └── ARCHETYPE.md    ← status: reviewed  |  epitome_file: src/.../OrderController.kt
├── endpoint_post_async/
│   └── ARCHETYPE.md    ← status: reviewed  |  epitome_file: src/.../OrderController.kt
├── service_lookup/
│   └── ARCHETYPE.md    ← status: reviewed  |  epitome_file: src/.../OrderLookupService.kt
└── ...
```

No code is copied into `.epitome/`. Each archetype points to a real, maintained file.
When that file evolves, the archetype evolves with it.

See [Directory Structure](spec/directory-structure.md) for the full spec.

---

## Workflow

| Step | Skill | Description |
|------|-------|-------------|
| 1 | `epitome-init` | Scan the codebase, detect archetype seams, present for approval |
| 2 | `epitome-pin` | For each archetype, pick a real file as the canonical example |
| 3 | `epitome-review` | Compare each epitome against other instances, refine rules |
| 4 | `epitome-manifesto` | Write `MANIFESTO.md` with the archetype-first gap protocol |
| 5 | `epitome-refactor` | Periodic: find drift from archetypes and fix it |

**Mid-development**: `epitome-pin` also handles the case where an agent hits a code pattern
with no archetype during feature work — it stops, defines the new archetype, then resumes.

---

## Installation

### Global (Pi)

```bash
git clone https://github.com/yourusername/epitome ~/.pi/agent/skills/epitome-src

ln -s ~/.pi/agent/skills/epitome-src/epitome-init      ~/.pi/agent/skills/epitome-init
ln -s ~/.pi/agent/skills/epitome-src/epitome-pin       ~/.pi/agent/skills/epitome-pin
ln -s ~/.pi/agent/skills/epitome-src/epitome-review    ~/.pi/agent/skills/epitome-review
ln -s ~/.pi/agent/skills/epitome-src/epitome-manifesto ~/.pi/agent/skills/epitome-manifesto
ln -s ~/.pi/agent/skills/epitome-src/epitome-refactor  ~/.pi/agent/skills/epitome-refactor
```

### Project-level

```bash
mkdir -p .pi/skills
ln -s /path/to/epitome/epitome-init      .pi/skills/epitome-init
ln -s /path/to/epitome/epitome-pin       .pi/skills/epitome-pin
ln -s /path/to/epitome/epitome-review    .pi/skills/epitome-review
ln -s /path/to/epitome/epitome-manifesto .pi/skills/epitome-manifesto
ln -s /path/to/epitome/epitome-refactor  .pi/skills/epitome-refactor
```

---

## Usage

```
/skill:epitome-init
```

Follow the approval step, then:

```
/skill:epitome-pin
/skill:epitome-review
/skill:epitome-manifesto
```

Run periodically to catch drift:

```
/skill:epitome-refactor
```

---

## Specification

- [Directory Structure](spec/directory-structure.md) — the `.epitome/` format
- [ARCHETYPE.md Format](spec/archetype-md-format.md) — how to write archetypes

---

## Principles

- **Real code, not examples** — archetypes point to real files; no synthetic copies
- **Single source of truth** — the `epitome_file` is the pattern; ARCHETYPE.md is the rules
- **Granular seams** — one archetype per distinct structural pattern; `service_deletion` and `service_command` are different archetypes
- **Detect, don't just declare** — every archetype ships with bash commands to find instances and deviations
- **Language-agnostic** — the framework works for any language; your archetypes are specific to yours
- **Human in the loop** — steps 2 and 3 are human-guided; agents propose, humans decide what ideal means
- **Gap protocol** — when an agent hits a missing archetype, it stops and defines before implementing

---

## License

MIT
