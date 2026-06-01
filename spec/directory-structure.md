# .epitome/ Directory Structure

The `.epitome/` directory lives at the root of the repository it describes.

```
.epitome/
├── MANIFESTO.md              # Global policies and archetype-first protocol (written in step 4)
└── <archetype_id>/           # One directory per archetype
    └── ARCHETYPE.md          # Rules, detection, anti-patterns, pointer to epitome file
```

**No code files are copied into `.epitome/`.** Each archetype's `ARCHETYPE.md` contains an
`epitome_file` field pointing to a real file in the codebase. That file *is* the canonical
example — there is only one source of truth.

The directory itself is the state tracker: if `.epitome/<id>/ARCHETYPE.md` exists, the
archetype exists. The `status` field in each frontmatter tracks workflow progress.

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
endpoint_get_single/
endpoint_get_filtered_list/
endpoint_post_async/
endpoint_delete/
service_lookup/
service_command/
service_deletion/
service_async_processor/
repository_simple/
repository_custom_query/
migration_flyway_schema/
test_unit/
test_integration/
test_data/
entity_jpa/
domain_error/
config_module/
```

---

## MANIFESTO.md

A prompt-style document written for an AI agent. Contains:
- The archetype-first development protocol (what to do when a pattern has no archetype)
- A table of all archetypes with one-liner summaries
- A "Patterns NOT yet defined" list (convenience hint — the directory is authoritative)
- Pointer to `AGENTS.md` / `CLAUDE.md` for all other project guidelines

Written in step 4. See [`../epitome-manifesto/SKILL.md`](../epitome-manifesto/SKILL.md).
