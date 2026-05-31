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
      "status": "approved | pending | rejected"
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
| `approved` | Human approved — proceed to generate |
| `rejected` | Human rejected — do not generate |

---

## Lifecycle

1. **`epitome-init`** creates `tasks.json` with steps and a list of proposed archetypes (all `pending`)
2. Human approves/rejects archetypes (edits `status` fields, or tells the agent)
3. **`epitome-generate`** reads `approved` archetypes and creates their directories; marks step 2 `done`
4. Future skills update steps 4 and 5
