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
- **`MANIFESTO.md`** — a prompt-style document declaring policies and principles that
  code examples alone cannot express
- **`tasks.json`** — tracks progress as the epitome is built

```
.epitome/
├── MANIFESTO.md
├── tasks.json
├── controller_rest/
│   └── ARCHETYPE.md          ← epitome_file: src/.../OrderController.kt
├── service_domain/
│   └── ARCHETYPE.md          ← epitome_file: src/.../OrderService.kt
└── ...
```

No code is copied into `.epitome/`. Each archetype points to a real, maintained file in
your codebase. When that file evolves, the archetype evolves with it.

See [Directory Structure](spec/directory-structure.md) for the full spec.

---

## Workflow

| Step | Skill | Description |
|------|-------|-------------|
| 1 | `epitome-init` | Scan the codebase, identify archetypes, present for approval |
| 2 | `epitome-pin` | For each archetype, pick a real file as the canonical example |
| 3 | `epitome-review` | Compare the epitome against other instances, refine rules |
| 4 | `epitome-manifesto` | Write `MANIFESTO.md` from what the archetypes reveal |
| 5 | `epitome-refactor` | Periodic: find drift from archetypes and fix it |

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
```

Pick a canonical file for each archetype, then review and refine:

```
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
- [tasks.json Format](spec/tasks-json-format.md) — progress tracking schema

---

## Principles

- **Real code, not examples** — archetypes point to real files in your codebase; no synthetic copies
- **Single source of truth** — the `epitome_file` is the pattern; ARCHETYPE.md is the rules around it
- **Detect, don't just declare** — every archetype ships with bash commands to find instances and deviations
- **Language-agnostic** — the framework works for any language; your archetypes are specific to yours
- **Human in the loop** — steps 2 and 3 are human-guided; agents propose, humans decide what ideal means

---

## License

MIT
