# .epitome/ Directory Structure

The `.epitome/` directory lives at the root of the repository it describes.

```
.epitome/
├── MANIFESTO.md              # Global policies (written in step 4)
├── tasks.json                # Workflow progress tracker
└── <archetype_id>/           # One directory per archetype
    └── ARCHETYPE.md          # Rules, detection, anti-patterns, pointer to epitome file
```

**No code files are copied into `.epitome/`.** Each archetype's `ARCHETYPE.md` contains an
`epitome_file` field pointing to a real file in the codebase. That file *is* the canonical
example — there is only one source of truth.

---

## Archetype directory naming

```
<technical_aspect>_<domain_specification>
```

Lowercase with underscores. Technical aspect first; domain qualifier only when the pattern
has a domain-specific meaning within that codebase.

Examples:
```
controller_rest/
service_domain/
repository_spring_data/
test_unit/
test_integration/
test_data/
config_module/
```

When purely technical, the technical aspect alone is enough:
```
event_domain/
middleware_auth/
```

---

## tasks.json

Tracks the state of the epitome build process. See [tasks-json-format.md](tasks-json-format.md).

---

## MANIFESTO.md

A prompt-style document written for an AI agent. Contains:
- Policies that cannot be expressed in code (naming conventions, commit hygiene)
- Architectural decisions and their rationale
- Cross-cutting rules that apply to all archetypes

Written in step 4. See [`../epitome-manifesto/SKILL.md`](../epitome-manifesto/SKILL.md).
