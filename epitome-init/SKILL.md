---
name: epitome-init
description: Step 1 of the epitome workflow. Scans a codebase to identify recurring code archetypes at the right level of granularity — differentiating service types (lookup, command, deletion, async), repository types (simple, custom query), endpoint types (GET single, GET filtered list, POST, DELETE), migration files, entities, domain errors, and more. Presents them for human approval. Use when starting an epitome for a new project.
license: MIT
---

# epitome-init

Runs **step 1** of the epitome workflow: discover archetypes in the codebase and present
them for human approval.

Read the [directory structure spec](../spec/directory-structure.md) and
[ARCHETYPE.md format](../spec/archetype-md-format.md) before proceeding.

---

## Goal

Produce a list of archetypes — recurring structural patterns worth defining as canonical
examples — and present them to the human for approval.

**The key principle**: one archetype = one distinct structural pattern. Do not group different
patterns together just because they share a class suffix or file type. A read-only lookup
service and a deletion service are different archetypes even though both end in `Service`.
A schema migration and a data migration are different archetypes even though both are SQL files.

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

Identify: language, framework, module structure, test framework, migration tool.

---

### 2. Detect seams within each file type

For each category below, look INSIDE files to find the structural markers that separate
one archetype from another. Run the detection commands, read 2–3 representative files per
candidate pattern.

---

#### Controllers / Routers — split by HTTP handler pattern

```bash
# Find all controller files
find . -type f -name "*Controller.kt" | grep -v "build\|test"

# For each, show handler signatures and annotations
grep -n "@GetMapping\|@PostMapping\|@PutMapping\|@DeleteMapping\|@PatchMapping\|@RequestParam\|@PathVariable\|@RequestBody\|@ResponseStatus" \
  <controller-file>
```

| Pattern | Structural markers | Archetype |
|---|---|---|
| Controller class | `@RestController` + `@RequestMapping` + constructor injection | `controller_rest` |
| GET by path variable | `@GetMapping("/{var}")` + `@PathVariable` → single typed result | `endpoint_get_single` |
| GET filtered list | `@GetMapping` + `@RequestParam` → `List<Response>` | `endpoint_get_filtered_list` |
| GET unfiltered list | `@GetMapping` (no path var, no params) → `List<Response>` | `endpoint_get_list` |
| GET sub-resource list | `@GetMapping("/{var}/subresource")` + `@PathVariable` → `List<Response>` | `endpoint_get_subresource_list` |
| POST async | `@PostMapping` + `@ResponseStatus(ACCEPTED)` + `@RequestBody` | `endpoint_post_async` |
| POST create | `@PostMapping` + `@ResponseStatus(CREATED)` + `@RequestBody` | `endpoint_post_create` |
| DELETE | `@DeleteMapping("/{var}")` + `@ResponseStatus(NO_CONTENT)` → `Unit` | `endpoint_delete` |
| PUT / PATCH | `@PutMapping` / `@PatchMapping` + `@RequestBody` | `endpoint_put` / `endpoint_patch` |

Propose a separate archetype for each distinct handler pattern found, even if all live
in the same controller file.

---

#### Services — split by transactional role and behaviour

```bash
find . -type f -name "*Service.kt" | grep -v "build\|test"

# For each service, show annotations + any delete/async calls
grep -n "@Service\|@Transactional\|@Async\|@Scheduled\|\.delete(\|\.deleteById(" <service-file>
```

| Pattern | Structural markers | Archetype |
|---|---|---|
| Read-only lookup | `@Transactional(readOnly = true)` at class level | `service_lookup` |
| Command / orchestration | `@Service` only; validates input, coordinates collaborators | `service_command` |
| Deletion | `@Service`; find-or-throw via lookup + `repository.delete()` | `service_deletion` |
| Async processor | `@Async("executor")` on processing method; try/catch → success/fail states | `service_async_processor` |
| Scheduled / batch | `@Scheduled` on a method | `service_scheduled` |
| Event handler | `@EventListener` / `@TransactionalEventListener` | `service_event_handler` |

```bash
# Identify deletion services specifically
grep -rl "@Service" . --include="*.kt" | grep -v "build\|test" | \
  xargs grep -l "\.delete(\|\.deleteById("

# Identify async processors
grep -rl "@Async" . --include="*.kt" | grep -v "build\|test"
```

---

#### Repositories — split by query complexity

```bash
find . -type f -name "*Repository.kt" | grep -v "build\|test"

grep -n "@Query\|@Modifying\|@NativeQuery" <repository-file>
```

| Pattern | Structural markers | Archetype |
|---|---|---|
| Simple CRUD | Extends `JpaRepository` / `CrudRepository`; no custom `@Query` | `repository_simple` |
| Custom query | Has `@Query` (JPQL or native SQL) | `repository_custom_query` |
| Projection | Returns interface or DTO projections | `repository_projection` |

---

#### Migrations — split by migration type

```bash
# Flyway (SQL)
find . -path "*/db/migration/*.sql" | grep -v build | sort

# Liquibase
find . \( -name "*.xml" -o -name "*.yaml" -o -name "*.yml" \) \
  -path "*/changelog*" | grep -v build

# Alembic (Python)
find . -name "*.py" -path "*/migrations/*" | grep -v build

# Django migrations
find . -name "*.py" -path "*/migrations/[0-9]*" | grep -v build

# Node / TypeORM
find . -name "*Migration*.ts" | grep -v build
```

Read several migration files to detect seams:

| Pattern | What to look for | Archetype |
|---|---|---|
| Schema migration | DDL: `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX` | `migration_flyway_schema` (or `migration_schema`) |
| Data migration | DML with business logic: `INSERT`, `UPDATE` across rows | `migration_flyway_data` (or `migration_data`) |
| Seed migration | Initial reference data insert | `migration_seed` |

Migrations are archetypes because they have a consistent structure (naming convention,
reversibility strategy, transaction handling) that new team members must learn.

---

#### Tests — split by framework

```bash
# Unit tests (Mockito / Jest / pytest-mock)
grep -rl "@ExtendWith(MockitoExtension\|MockitoJUnitRunner\|jest.mock\|unittest.mock" \
  . --include="*.kt" --include="*.ts" --include="*.py" | grep -v "build"

# Integration / slice tests
grep -rl "@SpringBootTest\|@DataJpaTest\|@WebMvcTest\|@ApplicationModuleTest" \
  . --include="*.kt" | grep -v "build"

# Test data factories
find . \( -name "*TestData.kt" -o -name "*factory.ts" -o -name "*factories.py" \
  -o -name "conftest.py" \) | grep -v "build"
```

---

#### Supporting patterns

```bash
# JPA entities
grep -rl "@Entity" . --include="*.kt" --include="*.java" | grep -v "build\|test"

# Domain errors / typed exceptions
grep -rl "class.*: NotFound\|class.*: Conflict\|class.*: BadRequest\|class.*Exception\|class.*Error" \
  . --include="*.kt" | grep -v "build\|test"

# Configuration modules
grep -rl "@Configuration" . --include="*.kt" --include="*.java" | grep -v "build\|test"

# Request / response DTOs
find . \( -name "*Request.kt" -o -name "*Response.kt" \) | grep -v "build\|test"
```

---

### 3. Apply thresholds

An archetype must be:
- **Recurring** — 2+ instances (relaxed to 1 for central/important patterns in small projects)
- **Structural** — consistent skeleton that can be shown by pointing at one file
- **Meaningful** — knowing the pattern changes how a developer writes code

Patterns with **0 instances** in the codebase: do NOT propose. Their absence is the signal
that stops the agent when implementing a feature that would need them for the first time.

---

### 4. Present for approval

Group by category. For each, show:
- One-line description
- Count of instances
- Representative file path (would become the epitome candidate)

```
Technical archetypes found:

CONTROLLER / ENDPOINT
  [1] controller_rest         — REST controller class structure
      1 instance: WordCountController.kt
  [2] endpoint_get_single     — GET /{pathVar} → single typed resource
      2 instances: getJob, getWordCount in WordCountController
  [3] endpoint_post_async     — POST → 202 ACCEPTED
      1 instance: WordCountController.count
  [4] endpoint_delete         — DELETE /{pathVar} → 204 NO CONTENT
      1 instance: WordCountController.deleteJob

SERVICE
  [5] service_lookup          — @Transactional(readOnly=true) class-level
      2 instances: WordStatisticsService, WordCountingJobLookupService
  [6] service_command         — orchestration, input validation, domain exceptions
      1 instance: WordCountingService
  [7] service_deletion        — find-or-throw + repository.delete()
      1 instance: WordCountingJobDeletionService
  [8] service_async_processor — @Async + try/catch state machine
      1 instance: WordCountingProcessor

REPOSITORY
  [9]  repository_simple       — plain JpaRepository, no @Query
       1 instance: WordCountingJobRepository
  [10] repository_custom_query — @Modifying @Query native SQL
       1 instance: WordCountRepository

MIGRATION
  [11] migration_flyway_schema — DDL migration (CREATE TABLE, index)
       2 instances: V1__, V2__

TEST
  [12] test_unit               — @ExtendWith(MockitoExtension) unit test
       4 instances
  [13] test_integration        — @SpringBootTest + MockMvcTester
       1 instance: WordCountControllerIntegrationTest
  [14] test_data               — object TestData with a_<entity>() factories
       1 instance: WordCountsTestData

Approve all? Or specify changes:
```

Wait for explicit human approval before proceeding.

---

### 5. After approval

Tell the human: "Run `epitome-pin` to create the ARCHETYPE.md for each approved archetype."

Do **not** create any files yourself at this step. `epitome-init` discovers and presents.
`epitome-pin` creates.

---

## Output

- Human has approved the archetype list
- No files created

Next step: run `epitome-pin`.
