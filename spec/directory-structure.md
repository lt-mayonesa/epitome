# .epitome/ Directory Structure

The `.epitome/` directory lives at the root of the repository it describes.

```
.epitome/
├── MANIFESTO.md              # Global policies (written in step 4)
├── tasks.json                # Workflow progress tracker
└── <archetype_id>/           # One directory per archetype
    ├── ARCHETYPE.md          # Rules, detection, anti-patterns
    └── <ExampleFile>.<ext>   # One or more example files
```

---

## Archetype directory naming

```
<technical_aspect>_<domain_specification>
```

The name should be lowercase with underscores. The technical aspect comes first; the domain
specification narrows it when a technical pattern has a domain-specific meaning.

Examples:
```
controller_rest/
service_domain/
service_warehouse_variant/
repository_jooq/
configuration_wms_db_driven/
test_controller/
```

When the pattern is purely technical with no domain qualifier, the technical aspect alone is enough:
```
config_module/
event_internal/
query_jooq_cte/
```

---

## Example file naming

Example files inside an archetype directory use realistic, domain-relevant names — not `example.*`.
The filename acts as the naming template for that archetype in the real codebase.

Use `<Entity>` as a placeholder where a concrete domain entity name would go:

```
controller_rest/
└── <Entity>Controller.java       # e.g. ShipmentController, OrderController

service_warehouse_variant/
├── <Entity>Strategy.java         # shared interface
├── Fc<Entity>Strategy.java       # FC implementation
└── Dc<Entity>Strategy.java       # DC implementation
```

When an archetype requires multiple files to illustrate the full pattern, include all of them.

---

## tasks.json

Tracks the state of the epitome build process. See [tasks-json-format.md](tasks-json-format.md).

---

## MANIFESTO.md

A prompt-style document written for an AI agent. Contains:
- Policies that cannot be expressed in code (e.g. naming conventions, commit hygiene)
- Architectural decisions and their rationale
- Cross-cutting rules that apply to all archetypes

Written in step 4. See the step 4 skill (coming soon) for guidance.
