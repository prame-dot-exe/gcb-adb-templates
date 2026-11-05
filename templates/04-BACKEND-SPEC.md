# [Project Name] — ADB 04: Backend Specification

> **Application Design Book (ADB) — Book 4 of 5.** Every endpoint the backend exposes, with the rules that are specific to this project. The API shape, folder structure and code patterns are in `CONVENTIONS.md` §3–4.
> Guideline: `guidelines/04-BACKEND-SPEC.guide.md`. Section tags: **Core** = always fill · **Optional** = fill only if it applies, otherwise leave the section out · **Standard** = follows `CONVENTIONS.md`; leave it out unless the project differs (each difference needs an ADR in ADB 02 §13). Untagged subsections follow their parent's tag. List every section left out in Document Info → *Sections left out*; keep the numbers of the sections you write.
> Keep it short: tables, not prose. Use `TBD — see Q-NN` for unresolved items.

---

## 0. Document Info

> **Core**

| Field        | Value                                       |
| ------------ | ------------------------------------------- |
| Project name | [Name]                                      |
| Project code | [Code]                                      |
| Book         | ADB 04 — Backend Spec                       |
| Version      | [0.1.0]                                     |
| API version  | v1                                          |
| Status       | [Draft / In Review / Approved / Deprecated] |
| Author(s)    | [Name, role]                                |
| Last updated | [YYYY-MM-DD]                                |
| Sections left out | [§x Name (not used) · §y Name (standard, no changes)] |
| Repository   | [URL]                                       |

### Change Log

| Version | Date         | Author | Summary of changes |
| ------- | ------------ | ------ | ------------------ |
| 0.1.0   | [YYYY-MM-DD] | [Name] | Initial draft      |

---

## 1. Tech Summary

> **Core.** Fixed stack: `CONVENTIONS.md` §1. Fill only the open choices.

| Item              | Value                            |
| ----------------- | -------------------------------- |
| Node.js           | [version]                        |
| DB access         | [Knex / Drizzle / Prisma]        |
| Auth verification | [JWT from NextAuth / own issuer] |
| Package manager   | [pnpm / npm]                     |

---

## 2. API Conventions

> **Standard:** `CONVENTIONS.md` §3.1–3.2 (base paths, endpoint pattern, submodules, extra actions).

## 3. Request Body Structure

> **Standard:** `CONVENTIONS.md` §3.3. Search page size limit: [100].

## 4. Response Structure

> **Standard:** `CONVENTIONS.md` §3.4.

| Difference from §2–4 standards | Reason | ADR |
| ------------------------------ | ------ | --- |
| [None]                         |        |     |

---

## 5. Response Codes

> Format and global codes: **Standard**, `CONVENTIONS.md` §3.5.

### 5.1 Module Codes

> **Core.** Every module code, once. Endpoint details refer to these.

| Code                            | HTTP | When                      | User message                                  |
| ------------------------------- | ---- | ------------------------- | --------------------------------------------- |
| [PROJECT_CREATE_SUCCESS]        | 201  | [Created]                 | [Project created]                             |
| [PROJECT_NOT_FOUND]             | 404  | [ID not found or deleted] | [Project not found]                           |
| [PROJECT_CLOSE_HAS_OPEN_ISSUES] | 409  | [BR-PRJ-01]               | [Close all issues before closing the project] |

---

## 6. Authentication & Permissions

> **Core**

### 6.1 Auth Flow

[One or two lines: how the frontend gets the token and how the backend verifies it. See ADB 02 §6.1.]

### 6.2 Middleware

> **Standard:** `CONVENTIONS.md` §4.3. List only extra middleware.

| Middleware | Applies to | Purpose |
| ---------- | ---------- | ------- |
| [None]     |            |         |

### 6.3 Ownership Rules

> Who can call what comes from BRD §4.4 and the Roles column in §7. List here only the "own only" / conditional rules and how they are checked.

| Endpoint(s)                           | Rule                   | Check                                          |
| ------------------------------------- | ---------------------- | ---------------------------------------------- |
| [PUT /api/v1/project/:projectId/edit] | [Own only for ROLE-02] | [`created_by = req.user.id` or project member] |

---

## 7. Route Catalog

> **Core.** Every endpoint, grouped by module in folder order. This is the single list of endpoints.

### 7.1 Module [CODE] — [Module name]

| EP ID        | Method | Path                                  | Description             | Feature (BRD) | Roles     | Tables  |
| ------------ | ------ | ------------------------------------- | ----------------------- | ------------- | --------- | ------- |
| EP-[CODE]-01 | POST   | /api/v1/[module]/create               | [Create a …]            | F-[CODE]-xx   | [ROLE-xx] | [table] |
| EP-[CODE]-02 | POST   | /api/v1/[module]/search               | [Search …]              |               |           |         |
| EP-[CODE]-03 | GET    | /api/v1/[module]/:[module]Id/get      | [Get one …]             |               |           |         |
| EP-[CODE]-04 | PUT    | /api/v1/[module]/:[module]Id/edit     | [Edit …]                |               |           |         |
| EP-[CODE]-05 | DELETE | /api/v1/[module]/:[module]Id/delete   | [Soft delete …]         |               |           |         |
| EP-[CODE]-06 | PUT    | /api/v1/[module]/:[module]Id/[action] | [Extra action — reason] |               |           |         |

### 7.2 External API (`/api-ext/v1`)

> **Optional:** when partners or other systems call the app.

| EP ID          | Method | Path            | Consumer         | Auth      | Description   |
| -------------- | ------ | --------------- | ---------------- | --------- | ------------- |
| EP-[CODE]-[NN] | [POST] | [/api-ext/v1/…] | [Partner system] | [API key] | [Description] |

---

## 8. Endpoint Details

> **Core**, but only for endpoints that need more than the catalog row: every **search**, every **extra action**, every **create/edit with business rules**, and every **get that returns related or computed data**. Plain get/delete and simple create/edit need no block: create/edit accept the table's *Set by: User* columns (ADB 03) and get returns the table row.

### EP-[CODE]-[NN]: `[METHOD] [path]`

[One line: what it does.]

**Body — `data` fields** _(create / edit / extra action)_

| Field  | Type   | Required | Rule                   |
| ------ | ------ | -------- | ---------------------- |
| [name] | string | Yes      | [1–200 chars, trimmed] |

**Search — filter & sort** _(search)_

| Filter key | Matches column | Operator |
| ---------- | -------------- | -------- |
| keyword    | [name, code]   | ILIKE    |
| [status]   | [status]       | IN       |

Sortable: [createdAt, name] · Default: createdAt desc

**Returns** _(get / search, only when it includes more than the table row)_

[Project row + `members[]` (id, name, role) + `progressPercent` (done tasks ÷ all tasks × 100)]

**Business rules & side effects**

| Item                         | Behavior                             |
| ---------------------------- | ------------------------------------ |
| [BR-CODE-xx]                 | [How it's enforced → code from §5.1] |
| [Notification / audit / job] | [Detail]                             |

---

## 9. Folder Structure

> **Standard:** `CONVENTIONS.md` §4.1–4.2. Folders follow the module tree in §7.

| Difference from the standard | Reason | ADR |
| ---------------------------- | ------ | --- |
| [None]                       |        |     |

---

## 10. Configuration

> **Core.** Project-specific environment variables (standard ones: `NODE_ENV`, `PORT`, `DATABASE_URL`, `CORS_ORIGINS`, auth secret).

| Env variable         | Required | Example | Description   |
| -------------------- | -------- | ------- | ------------- |
| [LINE_CHANNEL_TOKEN] | [Yes]    | `…`     | [Description] |

---

## 11. Background Jobs

> **Optional:** one row per ADB 02 `JOB-NN`.

| Job      | File                     | Service called     |
| -------- | ------------------------ | ------------------ |
| [JOB-01] | `src/jobs/[name].job.ts` | [service function] |

---

## 12. Error Handling & Logging

> **Standard:** `CONVENTIONS.md` §4.4. Add any extra data that must never be logged.

| Never log (in addition to the standard) |
| --------------------------------------- |
| [None]                                  |

---

## 13. Open Questions

> **Core**

| ID   | Question   | Owner  | Due          | Answer    |
| ---- | ---------- | ------ | ------------ | --------- |
| Q-01 | [Question] | [Name] | [YYYY-MM-DD] | [Pending] |
