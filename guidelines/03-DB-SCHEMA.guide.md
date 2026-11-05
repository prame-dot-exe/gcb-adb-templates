# Guideline — ADB 03: Database Schema

Explains every section of `templates/03-DB-SCHEMA.md`. Section numbers match the template.

**Purpose of this book:** the single source of truth for the PostgreSQL schema. Each table appears three ways that must agree: SQL DDL (what runs), a column dictionary (what it means), and a TypeScript data model (what the code uses).

**Rules**
- **Group by module.** Tables belong to the module that owns them (BRD module codes). Shared tables (users, roles, audit) go in a `CORE` module.
- **SQL, table, and TS type must match** column for column. If you change one, change all three.
- **Every table is justified** by at least one BRD feature and used by at least one endpoint in ADB 04.
- **Schema changes go through migrations** and a new DB schema version, never by editing old migrations.

---

## 0. Document Info

| Field | Meaning | Example |
|---|---|---|
| Version | Version of this document | `0.4.0` |
| DB schema version | Version of the schema itself; matches the migration file prefix | `v-1-2-0` |
| Database | Engine + version | PostgreSQL 16 |
| Change Log → Migration file | The migration that implements each schema change | `migrations/v-1-2-0_add_budget.ts` |

**Schema versioning:** `v-<major>-<minor>-<patch>`. Major = breaking change (drop/rename), minor = new table/column, patch = index/comment/constraint tweak.

---

## 1. Conventions
The pre-filled conventions are recommended defaults. Confirm or adjust each row once per project, then never break them.

| Convention | Why |
|---|---|
| snake_case plural tables, snake_case columns | PostgreSQL folds unquoted identifiers to lowercase; avoids quoting everywhere |
| `uuid` PK with `gen_random_uuid()` | Safe to expose in URLs (`/project/:projectId/get`); no enumeration |
| `<singular>_id` FKs | Readable joins (`project_id`) |
| `timestamptz` in UTC | Correct across timezones; display in local time on the FE |
| `numeric` for money | Floats lose cents |
| `is_` / `has_` booleans | Reads as a question |
| Enum strategy | **PostgreSQL ENUM** for stable sets (statuses); **lookup table** when admins can add values; **CHECK** for tiny sets |
| JSONB only for flexible, non-queried data | Queried fields deserve real columns and indexes |
| Soft delete via `deleted_at` | Matches "delete" endpoints that must be recoverable; every query filters `deleted_at IS NULL` |

**DB ↔ API naming:** columns are snake_case (`created_at`); API JSON and TS types are camelCase (`createdAt`). Mapping happens in the service/DB layer (e.g. a query builder `wrapIdentifier`/`postProcessResponse` hook), never in handlers.

### 1.1 Standard Columns
Every business table includes these. Junction tables may omit `updated_*` and `deleted_at` if rows are only inserted and removed; say so in the table description.

---

## 2. Module Overview

| Field | Meaning | Example |
|---|---|---|
| Module code | BRD module code | `PRJ` |
| Module | Name | Project |
| Tables | All tables owned by the module | `projects, project_members` |
| Owner | Responsible developer | Name |

**2.1 Full ERD:** every table and relationship in one diagram. For large schemas show only table names and relationships here; columns go in the module ERDs.

Mermaid ERD cardinality: `||--||` one-to-one, `||--o{` one-to-many (optional), `||--|{` one-to-many (at least one), `}o--o{` many-to-many (use a junction table in reality).

---

## 3. Enumerations

| Field | Meaning | Example |
|---|---|---|
| Enum type | PostgreSQL type name, `<entity>_<attribute>` | `project_status` |
| Values | Lowercase snake_case | `draft, active, on_hold, closed` |
| Used by | `table.column` | `projects.status` |
| Description | Meaning of each value | |
| Matches state diagram | The ADB 02 §3.3 diagram that uses these values | Project lifecycle |

**Adding enum values** is safe (`ALTER TYPE … ADD VALUE`); removing or renaming is a major schema change. The FE status color map (ADB 05 §3.4) must list every value.

---

## 4. Modules
One §4.x per module; inside it a module ERD and one table block per table.

### Table block

**SQL** — the exact DDL that a migration would run. Include constraint names (they appear in error messages and make migrations predictable), `ON DELETE` behavior for every FK, partial indexes for soft-delete, and `COMMENT ON` statements.

**Columns**
| Field | Meaning | Example |
|---|---|---|
| Column | snake_case name | `planned_end_date` |
| Type | PostgreSQL type with size/precision | `date`, `varchar(200)`, `numeric(14,2)` |
| Nullable | Yes / No | No |
| Default | Default expression | `now()` |
| Key / constraint | PK, FK (→ table), UQ, CK | `FK → projects` |
| Description | Business meaning, units, rules | Planned completion date, Bangkok calendar day |
| Example | Realistic value | `2027-03-31` |

**Type tips:** `text` vs `varchar(n)`: use `varchar(n)` when the business has a real limit (the zod `max()` in ADB 04 uses the same number). Use `date` for calendar days with no time, `timestamptz` for moments.

**Indexes**
| Field | Meaning | Example |
|---|---|---|
| Index | Name per convention | `idx_projects_status_created_at` |
| Columns | In order (most selective / equality first) | `status, created_at` |
| Type | B-tree, GIN (JSONB, full-text, trigram), unique | B-tree |
| Partial condition | `WHERE` clause | `deleted_at IS NULL` |
| Supports query | The endpoint or search filter it serves | `POST /project/search` filter status, sort createdAt |

**Rule:** every filter and sort field listed for a search endpoint in ADB 04 should be covered by an index, or the row should say why not (tiny table). For `keyword` ILIKE search, use a `pg_trgm` GIN index.

**Relationships** — direction, cardinality, `ON DELETE` choice:
- `RESTRICT` — parent can't be deleted while children exist (default for business data)
- `CASCADE` — children go with the parent (junction rows, owned detail rows)
- `SET NULL` — optional link that survives (e.g. `assignee_id`)

With soft delete, cascades are usually done in the service (set `deleted_at` on children), not by the database.

**Business rules enforced in DB** — which BRD rules the database guarantees (UNIQUE, CHECK, trigger) and which are service-only. Prefer DB enforcement for invariants that must never break (uniqueness, non-negative amounts).

**Data model (TypeScript)** — the camelCase type in `src/types/db/<module>.types.ts`:
| SQL | TypeScript |
|---|---|
| uuid, text, varchar | `string` |
| integer, smallint | `number` |
| bigint | `string` (avoid precision loss) |
| numeric | `string` (convert with a decimal library when calculating) |
| boolean | `boolean` |
| date, timestamptz | `string` (ISO) at the API boundary |
| jsonb | a named type from §5 |
| enum | string union type |
| nullable column | `T \| null` |

**Endpoints that use this table** — reverse link to ADB 04, with the operation (read / write / soft delete).

---

## 5. JSONB Structures
For every JSONB column: purpose, whether it's queried (and how it's indexed), and the exact TS shape. Validate JSONB with a zod schema in the service before writing. If you find yourself filtering on a JSONB key often, promote it to a column.

## 6. Views, Functions & Triggers
Everything that lives in the database besides tables: `updated_at` trigger, reporting views, materialized views (with refresh schedule), functions. Include the SQL.

## 7. Data Retention & Privacy
Per table: PII yes/no, which columns, retention period (from BRD 3.5 / §7), and deletion method (soft delete, anonymize, hard delete by a job). Anonymizing keeps aggregate reports correct while removing identity.

## 8. Seed Data
| Field | Meaning | Example |
|---|---|---|
| Seed file | `seeds/v-x-y-z_<topic>.ts` | `seeds/v-1-0-0_roles.ts` |
| Tables | Tables filled | `roles, permissions` |
| Environments | Where it runs | All (master data) / local+dev only (demo data) |
| Purpose | Why | Required roles for login |

Master data seeds must be idempotent (safe to run twice).

## 9. Migration Plan
One row per migration file: version, description, reversible yes/no, backfill. Follow expand → migrate → contract for renames and type changes so old app versions keep working during deploy.

**9.1 Legacy Data Migration:** only when replacing an existing system: source, field mapping, transforms, validation (row counts, totals), and rollback. Otherwise `N/A — new system`.

## 10. Open Questions
Global numbering shared with all books.
