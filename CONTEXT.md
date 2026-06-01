# Epitome — Project Context

This document gives a full picture of what epitome is, what has been built, the decisions
made along the way, and what remains to be done. It is written to be given to an agent
picking up work on this project.

---

## What it is

**Epitome** is a framework for defining and enforcing code standards through concrete examples
and agent-executable workflows.

The core insight: static style guides don't work because they're prose, not code, and nobody
reads them. Instead, you define *archetypes* — real, runnable code examples that show what
ideal code looks like for every recurring pattern in your codebase — and then let agents
periodically scan for deviations and fix them.

There are two artifacts:

1. **This OSS project** (`/Users/joaco/src/oss/epitome/`) — the framework: agent skills,
   format specifications, and README. Language-agnostic. No codebase-specific content.

2. **A `.epitome/` directory** inside a specific codebase — the instance: the actual archetypes,
   example files, ARCHETYPE.md files, MANIFESTO.md, and tasks.json for that project.
   This is codebase-specific and is NOT part of the OSS project.

The reference instance lives at:
`/Users/joaco/src/picnic/picnic-java-all/picnic-wms/.epitome/`

---

## The five-step workflow

| Step | Skill | Status (OSS) | Status (picnic-wms instance) |
|------|-------|--------------|-------------------------------|
| 1 | `epitome-init` | ✅ Skill written | ✅ Done — 16 archetypes approved |
| 2 | `epitome-generate` | ✅ Skill written | ✅ Done — all 16 archetypes generated |
| 3 | `epitome-review` | ✅ Skill written | 🔄 In progress — human review pending |
| 4 | `epitome-manifesto` | ❌ Not written | 🔄 In progress — MANIFESTO.md stub exists |
| 5 | `epitome-refactor` | ❌ Not written | ⬜ Todo |

---

## OSS project structure

```
epitome/                          <- /Users/joaco/src/oss/epitome/
├── README.md                     <- Project overview, installation, usage
├── CONTEXT.md                    <- This file
├── TODO.md                       <- Original brief and how-to notes
├── LICENSE                       <- MIT
├── spec/
│   ├── directory-structure.md    <- The .epitome/ format spec
│   ├── archetype-md-format.md    <- How to write ARCHETYPE.md files
│   └── tasks-json-format.md      <- tasks.json schema
├── epitome-init/
│   └── SKILL.md                  <- Step 1: scan codebase, propose archetypes
├── epitome-generate/
│   └── SKILL.md                  <- Step 2: create example files + ARCHETYPE.md
└── epitome-review/
    └── SKILL.md                  <- Step 3: compare archetype vs real instances, update
```

Steps 4 (`epitome-manifesto`) and 5 (`epitome-refactor`) are **not yet written**.

---

## Key design decisions

**Archetypes are generic, not project-specific.**
Example files use placeholder names like `<Entity>`, `Fc<Entity>`, `Dc<Entity>` in their
*filenames* to signal the naming pattern. The Java code inside uses realistic domain names
(e.g. `Shipment`) to make examples readable, but with enough comments that the pattern is clear.

**Archetype naming: `<technical_aspect>_<domain_specification>`.**
Technical aspect first. Domain qualifier only when the pattern has a domain-specific meaning
(e.g. `service_warehouse_variant` not just `strategy`). Purely technical archetypes use the
technical name alone (e.g. `config_module`, `event_internal`).

**ARCHETYPE.md has YAML frontmatter for machine use.**
Each ARCHETYPE.md starts with frontmatter containing `id`, `detect.grep`, `detect.path`,
and `related`. This allows the future `epitome-refactor` skill to parse and run detection
commands without reading the prose.

**Detection commands are the most valuable part.**
Every ARCHETYPE.md contains two kinds of bash detection commands:
1. Find all instances (inventory)
2. Find instances that likely violate the rules (smell detection)

The smell-detection commands are what make the refactoring step possible.

**Step 3 is intentionally manual.**
The `epitome-review` skill guides a human+agent review where the agent surfaces deviations
and proposes categorised changes, but the human makes the final call on what "ideal" means.
No automated merge of review results — explicit approval per deviation.

**Skills reference the spec via relative paths.**
`epitome-init/SKILL.md` links to `../spec/directory-structure.md`. This keeps the spec
as the single source of truth and means skills stay in sync with the spec when it changes.

---

## The 16 archetypes in the picnic-wms instance

### Technical archetypes (12)

| ID | What it covers |
|----|---------------|
| `controller_rest` | Spring `@RestController` in wms-webapp |
| `service_domain` | Business logic service in wms-logic |
| `repository_jooq` | jOOQ data access via `DSLContext` + Jolo |
| `config_module` | `@Configuration` + `@Import` bean registration |
| `handler_rabbitmq_message` | `MessageContentHandler<T>` + `@RunWithPrivileges` |
| `handler_event_internal` | `@TransactionalEventListener` + `@Async` + `@RunWithPrivileges` |
| `event_internal` | `sealed interface` with `record` variants |
| `test_controller` | `MockMvcBuilders.standaloneSetup` unit tests |
| `test_service` | Pure Mockito unit tests for services |
| `test_repository` | Testcontainer Postgres integration tests extending `AbstractPostgresTest` |
| `test_handler` | Pure delegation-verification unit tests for handlers |
| `builder_test_entity` | Fluent test data builder with `but()` copy pattern |

### Domain archetypes (4)

| ID | What it covers |
|----|---------------|
| `service_warehouse_variant` | FC/DC strategy: shared interface + `@ConditionalOnWarehouseType` impls |
| `configuration_wms_db_driven` | `@Value.Immutable` config interface + typed config service wrapping `WmsConfigurationService` |
| `query_jooq_cte` | `AbstractCte` subclass promoting a reusable SQL fragment to a named Java object |
| `log_operational` | `@Logged(action = ...)` + `LogEventHolder` audit trail on service methods |

---

## What still needs to be done

### In the OSS project

**Step 4 — `epitome-manifesto/SKILL.md`**
Guide the agent to write `MANIFESTO.md` based on the approved archetypes. The manifesto
covers rules that cannot be expressed in code examples: naming conventions, commit hygiene,
architectural decisions, cross-cutting policies. It is written in a prompt-like style —
it will be fed directly to an agent to guide feature implementation or code review.

**Step 5 — `epitome-refactor/SKILL.md`**
The periodic refactoring agent. It should:
1. Read all `ARCHETYPE.md` frontmatter to collect detection patterns
2. Run smell-detection commands across the codebase
3. For each violation found, show the offending file and the relevant rule
4. Propose a concrete fix, get approval, apply it
5. Report a summary of what was changed

Consider: should the refactor skill process one archetype at a time or scan all at once?
Should it prioritise by recency, frequency, or archetype importance?

**README improvements**
The current README covers installation and usage but lacks:
- A concrete worked example (before/after showing a real deviation being fixed)
- A "When NOT to use epitome" section (small codebases, early-stage projects)
- Badges (license, agentskills.io compatible)

### In the picnic-wms instance

**Step 3 — Human review of all 16 archetypes**
The human needs to read each `ARCHETYPE.md` and its example files and decide if they
represent truly ideal code for the picnic-wms codebase. The `epitome-review` skill
can assist: run it per archetype to compare the example against the 5 most recent real
instances and surface deviations.

**Step 4 — Complete MANIFESTO.md**
A stub exists at `.epitome/MANIFESTO.md`. It needs the full prompt-style content.
Use `epitome-manifesto` once that skill exists.

---

## Conventions to maintain

- Skills cross-reference each other and the spec via **relative paths** — never absolute
- Skill descriptions (the frontmatter `description` field) must precisely describe *when*
  to invoke the skill — this is what the agent's routing uses
- Every new skill must update the workflow table in `README.md`
- `tasks.json` schema changes go in `spec/tasks-json-format.md` first, then in the skills

---

## File locations quick reference

| Artifact | Path |
|----------|------|
| OSS project root | `/Users/joaco/src/oss/epitome/` |
| picnic-wms epitome instance | `/Users/joaco/src/picnic/picnic-java-all/picnic-wms/.epitome/` |
| Spec: directory structure | `spec/directory-structure.md` |
| Spec: ARCHETYPE.md format | `spec/archetype-md-format.md` |
| Spec: tasks.json format | `spec/tasks-json-format.md` |
| Step 1 skill | `epitome-init/SKILL.md` |
| Step 2 skill | `epitome-generate/SKILL.md` |
| Step 3 skill | `epitome-review/SKILL.md` |
| Step 4 skill | `epitome-manifesto/SKILL.md` ← **does not exist yet** |
| Step 5 skill | `epitome-refactor/SKILL.md` ← **does not exist yet** |
