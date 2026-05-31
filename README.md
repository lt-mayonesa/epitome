# epitome

> Keep your codebase honest. Define the ideal, detect the drift, fix it.

**epitome** is a framework for defining and enforcing code standards through concrete examples
and agent-executable workflows. Instead of writing static style guides that nobody reads,
you write *archetypes* — living code examples paired with machine-readable rules — and let
agents find and fix deviations automatically.

---

## How it works

An **epitome** for your project lives in a `.epitome/` directory at the root of your repository.
It holds:

- **Archetypes** — one directory per recurring code pattern, each with example code files and
  an `ARCHETYPE.md` describing rules, detection commands, and anti-patterns
- **`MANIFESTO.md`** — a prompt-style document declaring the policies and principles that cannot
  be expressed in code examples alone
- **`tasks.json`** — tracks the state of the epitome as it is built

```
.epitome/
├── MANIFESTO.md
├── tasks.json
├── controller_rest/
│   ├── ShipmentController.java
│   └── ARCHETYPE.md
├── service_domain/
│   ├── ShipmentService.java
│   └── ARCHETYPE.md
└── ...
```

See [Directory Structure](spec/directory-structure.md) for the full spec.

---

## Workflow

| Step | Skill | Description |
|------|-------|-------------|
| 1 | `epitome-init` | Scan the codebase, identify archetypes, present for approval |
| 2 | `epitome-generate` | Create example files and `ARCHETYPE.md` for each archetype |
| 3 | *(manual)* | Review and refine each example until it is truly ideal |
| 4 | *(coming soon)* | Write `MANIFESTO.md` |
| 5 | *(coming soon)* | Periodic refactoring agent that finds and fixes deviations |

---

## Installation

Add the skills to your Pi configuration or point your agent harness at this repository.

### Global (Pi)
```bash
# Clone the repo
git clone https://github.com/yourname/epitome ~/.pi/agent/skills/epitome-src

# Symlink the skills into your skills directory
ln -s ~/.pi/agent/skills/epitome-src/epitome-init ~/.pi/agent/skills/epitome-init
ln -s ~/.pi/agent/skills/epitome-src/epitome-generate ~/.pi/agent/skills/epitome-generate
```

### Project-level
```bash
mkdir -p .pi/skills
ln -s /path/to/epitome/epitome-init .pi/skills/epitome-init
ln -s /path/to/epitome/epitome-generate .pi/skills/epitome-generate
```

---

## Usage

Open your agent and run:

```
/skill:epitome-init
```

Follow the approval step, then:

```
/skill:epitome-generate
```

Then manually review and improve each archetype (step 3).

---

## Specification

- [Directory Structure](spec/directory-structure.md) — the `.epitome/` format
- [ARCHETYPE.md Format](spec/archetype-md-format.md) — rules for writing archetypes
- [tasks.json Format](spec/tasks-json-format.md) — progress tracking schema

---

## Principles

- **Code over prose** — archetypes are real, runnable code examples, not bullet points
- **Detect, don't just declare** — every archetype ships with bash commands to find instances and deviations
- **Generic structure, specific content** — the framework is language-agnostic; your archetypes are not
- **Human in the loop** — step 3 (review) is intentionally manual; machines propose, humans decide what ideal means

---

## License

MIT
