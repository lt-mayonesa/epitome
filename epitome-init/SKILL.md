---
name: epitome-init
description: Step 1 of the epitome workflow. Scans a codebase to identify recurring code archetypes at the right level of granularity — differentiating service types (lookup vs command vs async), repository types (simple vs custom query), and endpoint types (GET single, GET filtered list, POST, etc.). Presents them for human approval and creates .epitome/tasks.json. Use when starting an epitome for a new project.
license: MIT
---

# epitome-init

Runs **step 1** of the epitome workflow: discover archetypes in the codebase and present
them for human approval before any files are generated.

Read the [directory structure spec](../spec/directory-structure.md) and
[tasks.json format](../spec/tasks-json-format.md) before proceeding.

---

## Goal

Produce a list of archetypes — recurring structural patterns worth defining as canonical
examples — and write them to `.epitome/tasks.json` as `pending` entries awaiting human approval.

**The key principle**: one archetype = one distinct structural pattern. Do not group different
patterns together just because they share a class suffix. A read-only lookup service and an
async command service are different archetypes even though both end in `Service`.

Do **not** create archetype directories yet. That is step 2 (`epitome-pin`).

---

## Process

### 1. Orient yourself

```bash
# High-level layout
find . -maxdepth 4 -type d | grep -v ".git\|build\|target\|node_modules\|__pycache__\|.epitome\|.gradle" | head -60

# Primary language(s)
find . -type f | grep -v ".git\|build\|target\|node_modules\|dist" | \
  sed 's|.*\.||' | sort | uniq -c | sort -rn | head -15
```

Identify: language, framework, module structure, test framework.

---

### 2. Find all candidate files by type

```bash
# Kotlin/Java
find . -type f \( -name "*.kt" -o -name "*.java" \) | \
  grep -v "build\|target\|node_modules" | sed 's|.*/||' | sort | uniq -c | sort -rn

# TypeScript
find . -type f \( -name "*.ts" -o -name "*.tsx" \) | \
  grep -v "node_modules\|dist" | sed 's|.*/||' | sort
```

---

### 3. Detect seams — what makes one pattern different from another

For each file type, look INSIDE files to detect structural differences. The goal is to
find the natural "seams" that separate one archetype from another.

#### Controllers / Routers

Find all controller files, then scan their handler methods:

```bash
# Find controller files
find . -type f -name "*Controller.kt" | grep -v "build\|test"

# For each controller file, show all handler method signatures
grep -n "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@PatchMapping\|@RequestParam\|@PathVariable\|@RequestBody\|@ResponseStatus" \
  <controller-file>
```

Separate archetypes to propose for controllers:

| Pattern | What it looks like | Archetype name |
|---|---|---|
| Class structure | `@RestController` + `@RequestMapping` + constructor injection | `controller_rest` |
| GET by path variable | `@GetMapping("/{id}")` + `@PathVariable` → single typed result | `endpoint_get_single` |
| GET list, no params | `@GetMapping` → `List<Response>` | `endpoint_get_list` |
| GET list with filter | `@GetMapping` + `@RequestParam` → filtered `List<Response>` | `endpoint_get_filtered_list` |
| POST create | `@PostMapping` + `@RequestBody` → `@ResponseStatus(CREATED)` | `endpoint_post_create` |
| POST async | `@PostMapping` + `@RequestBody` → `@ResponseStatus(ACCEPTED)` | `endpoint_post_async` |
| DELETE | `@DeleteMapping("/{id}")` + `@ResponseStatus(NO_CONTENT)` | `endpoint_delete` |
| PUT/PATCH | `@PutMapping`/`@PatchMapping` + `@RequestBody` | `endpoint_put` / `endpoint_patch` |

Propose a separate archetype for each distinct handler pattern found across the codebase,
even if all handlers live in the same controller file.

#### Services

Differentiate services by their transactional role and execution model:

```bash
# Find all service files
find . -type f -name "*Service.kt" | grep -v "build\|test"

# For each service, show its class-level annotations and any @Async
grep -n "@Service\|@Transactional\|@Async\|class " <service-file>
```

Seams to look for:

| Pattern | Markers | Archetype name |
|---|---|---|
| Read-only lookup | `@Transactional(readOnly = true)` on class | `service_lookup` |
| Command / orchestration | `@Service` only, or `@Transactional` on methods | `service_command` |
| Async processor | `@Async` on a method, `@Service` on class | `service_async_processor` |
| Scheduled / batch | `@Scheduled` on a method | `service_scheduled` |
| Event handler | `@EventListener` / `@TransactionalEventListener` | `service_event_handler` |

#### Repositories

Differentiate repositories by query complexity:

```bash
# Find all repository files
find . -type f -name "*Repository.kt" | grep -v "build\|test"

# For each, check if it has custom @Query
grep -n "@Query\|@Modifying\|@NativeQuery" <repository-file>
```

Seams to look for:

| Pattern | Markers | Archetype name |
|---|---|---|
| Simple CRUD | Extends `JpaRepository` / `CrudRepository`, no custom `@Query` | `repository_simple` |
| Custom query | Has `@Query` (JPQL or native SQL) | `repository_custom_query` |
| Projection | Returns interface or DTO projections | `repository_projection` |

#### Tests

Differentiate test files by framework usage:

```bash
# Unit tests (Mockito)
grep -rl "@ExtendWith(MockitoExtension\|MockitoJUnitRunner" . --include="*.kt" | grep -v "build"

# Integration tests (Spring context)
grep -rl "@SpringBootTest\|@DataJpaTest\|@WebMvcTest\|@ApplicationModuleTest" . --include="*.kt" | grep -v "build"

# Test data factories
find . -type f -name "*TestData.kt" | grep -v "build"
```

---

### 4. Count instances and apply thresholds

For a pattern to become an archetype it must be:
- **Recurring** — appears in 2+ places (or is clearly the intended pattern for the project even if only 1 instance today)
- **Structural** — has a consistent skeleton
- **Meaningful** — knowing the pattern changes how a developer writes code

Threshold: **2+ instances** for most patterns. Relaxed to **1 instance** if the pattern is central
to the architecture (e.g., the main controller pattern in a small project).

For patterns with **0 instances** (e.g., no filtered-list endpoints yet): do NOT propose an archetype.
Their absence is a meaningful signal — when the agent hits a gap later, it stops and asks.

---

### 5. Present for approval

Show proposed archetypes grouped by category (controller/endpoint, service, repository, test).
For each:
- One-line description
- Count of instances (or "1 instance — central pattern")
- A representative file path

Example:

```
Technical archetypes found:

CONTROLLER/ENDPOINT
  [1] controller_rest — REST controller class structure
      1 instance: WordCountController.kt
  [2] endpoint_get_single — GET /{pathVar} returning typed single resource
      2 instances: WordCountController.getJob, WordCountController.getWordCount
  [3] endpoint_post_async — POST → 202 ACCEPTED for async job submission
      1 instance: WordCountController.count

SERVICE
  [4] service_lookup — @Transactional(readOnly=true) read-only service
      2 instances: WordStatisticsService, WordCountingJobLookupService
  [5] service_command — command/orchestration service, no class-level @Transactional
      1 instance: WordCountingService
  [6] service_async_processor — @Async processor with try/catch error state
      1 instance: WordCountingProcessor

REPOSITORY
  [7] repository_simple — plain JpaRepository, no custom @Query
      1 instance: WordCountingJobRepository
  [8] repository_custom_query — JpaRepository + @Modifying @Query native SQL
      1 instance: WordCountRepository

TEST
  [9]  test_unit — @ExtendWith(MockitoExtension) unit test
       4 instances
  [10] test_integration — @SpringBootTest + MockMvcTester
       1 instance: WordCountControllerIntegrationTest
  [11] test_data — TestData object with factory functions
       1 instance: WordCountsTestData

Approve all? Or specify changes:
```

Wait for explicit human approval before proceeding.

---

### 6. Write tasks.json

Once approved, create `.epitome/tasks.json`. Follow the [tasks.json format spec](../spec/tasks-json-format.md).

```json
{
  "project": "<project name>",
  "steps": [
    { "id": 1, "title": "Identify archetypes", "status": "done", "subtasks": [
      { "id": "1.1", "title": "Explore codebase for patterns", "status": "done" },
      { "id": "1.2", "title": "Present archetypes for approval", "status": "done" }
    ]},
    { "id": 2, "title": "Pin epitome file for each archetype", "status": "in-progress", "subtasks": [] },
    { "id": 3, "title": "Review each archetype against real instances", "status": "todo", "subtasks": [] },
    { "id": 4, "title": "Write MANIFESTO.md", "status": "todo", "subtasks": [] },
    { "id": 5, "title": "Refactor & iterate", "status": "todo", "subtasks": [] }
  ],
  "archetypes": [
    { "id": "controller_rest",          "folder": ".epitome/controller_rest/",          "status": "approved" },
    { "id": "endpoint_get_single",      "folder": ".epitome/endpoint_get_single/",      "status": "approved" },
    { "id": "endpoint_post_async",      "folder": ".epitome/endpoint_post_async/",      "status": "approved" },
    { "id": "service_lookup",           "folder": ".epitome/service_lookup/",           "status": "approved" },
    { "id": "service_command",          "folder": ".epitome/service_command/",          "status": "approved" },
    { "id": "service_async_processor",  "folder": ".epitome/service_async_processor/",  "status": "approved" },
    { "id": "repository_simple",        "folder": ".epitome/repository_simple/",        "status": "approved" },
    { "id": "repository_custom_query",  "folder": ".epitome/repository_custom_query/",  "status": "approved" },
    { "id": "test_unit",                "folder": ".epitome/test_unit/",                "status": "approved" },
    { "id": "test_integration",         "folder": ".epitome/test_integration/",         "status": "approved" },
    { "id": "test_data",                "folder": ".epitome/test_data/",                "status": "approved" }
  ]
}
```

---

## Output

- `.epitome/tasks.json` created
- Human approved the archetype list
- No archetype directories created yet

Next step: run `epitome-pin`.
