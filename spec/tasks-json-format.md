# tasks.json Format

`tasks.json` lives at `.epitome/tasks.json` and tracks the state of the epitome build.
It is created in step 1 and updated by each subsequent skill.

---

## Schema

```json
{
  "project": "<project name>",
  "steps": [
    {
      "id": 1,
      "title": "Identify archetypes",
      "status": "done | in-progress | todo",
      "subtasks": [
        { "id": "1.1", "title": "Explore codebase for patterns", "status": "done" },
        { "id": "1.2", "title": "Present archetypes for approval", "status": "done" }
      ]
    }
  ],
  "archetypes": [
    {
      "id": "<archetype_id>",
      "folder": ".epitome/<archetype_id>/",
      "status": "pending | approved | rejected | pinned | reviewed"
    }
  ]
}
```

---

## Status values

### Step status
| Value | Meaning |
|-------|---------|
| `todo` | Not started |
| `in-progress` | Currently being worked on |
| `done` | Completed |

### Archetype status
| Value | Meaning |
|-------|---------|
| `pending` | Proposed, awaiting human approval |
| `approved` | Human approved — proceed to step 2 (pin) |
| `rejected` | Human rejected — do not process |
| `pinned` | Epitome file chosen, ARCHETYPE.md written — ready for step 3 (review) |
| `reviewed` | Step 3 complete — archetype is considered final |

---

## Lifecycle

1. **`epitome-init`** creates `tasks.json` with steps and proposed archetypes (all `pending`)
2. Human approves/rejects archetypes — status moves to `approved` or `rejected`
3. **`epitome-pin`** reads `approved` archetypes, pins a real file for each, writes ARCHETYPE.md; marks each `pinned`, step 2 `done`
4. **`epitome-review`** compares pinned epitome against other instances; marks each `reviewed`, step 3 `done`
5. **`epitome-manifesto`** writes MANIFESTO.md; marks step 4 `done`
6. **`epitome-refactor`** runs periodically; does not change archetype status
