# Guideline — ADB 03: Database Schema

Explains every section of `templates/03-DB-SCHEMA.md`. Section numbers match the template.

**Purpose of this book:** the PostgreSQL schema. One ERD shows the whole database; each module then lists its tables as SQL plus a data dictionary. Naming, types, standard columns, soft delete, index and migration rules are in `CONVENTIONS.md` §2.

**Rules**

- **One ERD only** (§2.1), for the whole database. Modules don't get their own diagrams.
- **Each table = SQL + data dictionary.** Foreign keys, constraints and indexes are written in the SQL, with a short comment naming the business rule or search they serve. No separate index or relationship tables.
- **Don't repeat standard columns** (`id`, `created_*`, `updated_*`, `deleted_at`) in the data dictionary.
- **Group by module.** Tables belong to the BRD module that owns them; shared tables (users, roles, audit) go in `CORE`.
- **Every table is used** by at least one endpoint in ADB 04 (its Tables column).
- **Optional sections** that don't apply are left out and listed in Document Info → *Sections left out*.

---

## 0. Document Info · Core

| Field             | Meaning                                                  | Example       |
| ----------------- | -------------------------------------------------------- | ------------- |
| DB schema version | Version of the schema, matches the migration file prefix | `v-1-2-0`     |
| Database          | Engine + version                                         | PostgreSQL 16 |

---

## 1. Conventions · Standard

`CONVENTIONS.md` §2. List only differences (e.g. a separate schema per module), each with an ADR.

## 2. Module Overview · Core

One row per module with its tables, then **one ERD** with every table and relationship. For large schemas show table names and relationships only; columns are in §4.

Mermaid cardinality: `||--||` one-to-one, `||--o{` one-to-many, `||--|{` one-to-many (at least one).

---

## 3. Enumerations · Optional

One row per enum type, named `<entity>_<attribute>` (`project_status`), values in lowercase snake_case. The values come from BRD §4.2 statuses and must match the ADB 02 state diagram and the ADB 05 status colors. Adding a value is safe; removing or renaming is a major schema change.

---

## 4. Modules · Core

One §4.x per module, one table block per table. Each block has two parts.

**SQL** — the `CREATE TABLE` (standard columns as a comment) plus its `CREATE INDEX` statements:

- Every foreign key with its `ON DELETE` choice (defaults in `CONVENTIONS.md` §2.3).
- `UNIQUE` / `CHECK` for business rules that must never break (uniqueness, non-negative amounts), with a `-- BR-…` comment.
- An index for every filter and sort of a search endpoint (ADB 04 §8) unless the table is tiny, with a `-- search: …` comment. Equality column first; `keyword` search uses a `pg_trgm` GIN index.

**Data dictionary** — every non-standard column: type, required, **set by** (User = the form sends it; System = computed or generated; Action: … = changed only by an extra action such as *close*), and the business meaning with limits and an example ("Planned completion date (Bangkok day), e.g. 2027-03-31"). Use `varchar(n)` when the business has a real limit; the same `n` is the validation limit in the backend.

---

## 5. JSONB Structures · Optional

For each JSONB column: purpose and the TypeScript shape. If you often filter on a JSONB key, make it a column.

## 6. Views, Functions & Triggers · Optional

Reporting views, materialized views (with refresh schedule), functions. The `updated_at` trigger is standard.

## 7. Data Retention & Privacy · Core when there is personal data

Per table with personal data: which columns, how long to keep (from BRD 3.4 / §7), and what happens after (anonymize keeps reports correct while removing identity; or delete by a job).

## 8. Seed & Migration · Optional

Master data needed at launch (roles, settings) and any import from an old system: source, field mapping, cleanup rules, and how to check counts after import.

## 9. Open Questions · Core

Global numbering shared with all books.
