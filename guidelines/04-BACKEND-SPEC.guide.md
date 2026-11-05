# Guideline — ADB 04: Backend Specification

Explains every section of `templates/04-BACKEND-SPEC.md`. Section numbers match the template.

**Purpose of this book:** the contract between frontend and backend, and the map of the backend codebase. A developer should be able to implement any endpoint, and the frontend should be able to call it, from this book alone.

**Rules**
- **Sections 2–5 and 9 are fixed conventions.** Don't rewrite them per project; fill only the `[placeholders]`. Changing a convention requires an ADR in ADB 02.
- **Every endpoint appears three times, consistently:** route catalog (§7), endpoint detail (§8), and permission map (§6.3).
- **Every endpoint traces to a BRD feature** and lists the tables it touches (ADB 03).
- **Validation in zod mirrors the DB:** `max(200)` in zod ↔ `varchar(200)` in SQL.

---

## 0. Document Info
Standard fields plus **API version** (`v1`) and the **OpenAPI / Swagger URL** per environment (generated from `*.docs.ts`).

## 1. Tech Summary
Confirm the fixed stack (TypeScript strict, Express, Zod, PostgreSQL) and fill the open choices: Node version, DB access library, auth verification, package manager.

---

## 2. API Conventions

### Why these conventions
| Convention | Reason |
|---|---|
| Action suffix on every path (`/create`, `/search`, `/get`, `/edit`, `/delete`) | Paths read like intent; the same 5 actions exist in every module, matching the FE `feature/<module>/<action>` folders and `libs/api/<module>/<action><Entity>.ts` files |
| Only GET / POST / PUT / DELETE | Simple, consistent; no PATCH |
| `search` is always POST | Filters can be nested, long, and contain arrays; bodies avoid URL length limits and encoding issues |
| ID before the action (`/:projectId/edit`) | The path reads "this project → edit"; submodules nest naturally (`/:projectId/milestone/create`) |
| Singular camelCase module segments | Same name as the folder (`src/api/milestoneTask/` ↔ `/milestoneTask`) |

### 2.1 Endpoint pattern
| Action | Method | Path param naming |
|---|---|---|
| create | POST | — |
| search | POST | parent IDs only |
| get / edit / delete | GET / PUT / DELETE | `:<module>Id` — always named after the module (`:projectId`, never `:id`) so nested routes never collide |

**Extra actions:** when a change is a business transition rather than a field edit (close, approve, assign, restore), give it its own endpoint: `PUT /api/v1/project/:projectId/close`. Use POST for actions that create something or run a process (`POST /api/v1/project/export`). Name them verb or verbNoun in camelCase. Each needs a reason in the route catalog and usually a transition row in ADB 02 §3.3.

**Delete:** soft delete by default (sets `deleted_at`). A hard delete needs a reason in the endpoint detail.

---

## 3. Request Body Structure

| Action | Body | Notes |
|---|---|---|
| create | `{ data: {…} }` | All required fields |
| edit | `{ data: {…} }` | Only changed fields; omitted = unchanged. Never accept `id`, `createdAt`, `createdBy`, `deletedAt` in `data` |
| search | `{ filter, sort, pagination }` | All optional with defaults |
| get / delete | none | Use query params only if listed (e.g. `?include=`) |

**Why wrap in `data`:** leaves room for request-level options later (`{ data, options: { notify: false } }`) without mixing them with entity fields.

**Search filter naming:**
| Kind | Key pattern | Example |
|---|---|---|
| Free text | `keyword` | `"keyword": "alpha"` |
| Exact / multi-select | field name, value or array | `"status": ["active", "on_hold"]` |
| Range | `<field>From`, `<field>To` | `"plannedEndDateFrom": "2027-01-01"` |
| Boolean | field name | `"isArchived": false` |
| Relation | `<entity>Id` | `"ownerId": "…"` |

Unknown filter keys must fail validation (`z.object({...}).strict()`), so typos don't silently return everything.

**Pagination:** page-based. `page` starts at 1. `pageSize` max is set per project (default 100).

---

## 4. Response Structure

| Field | Type | Present | Meaning |
|---|---|---|---|
| `success` | boolean | Always | `true` for 2xx, `false` otherwise |
| `message` | string | Always | Human-readable, safe to show the user; no stack traces or SQL |
| `code` | string | Always | Stable machine code (§5); FE logic branches on this, never on `message` |
| `data` | object / array | When there is a payload | Object for get/create/edit/delete; **array of rows for search**; `{ errors: [...] }` for validation errors |
| `items` | object | Search only | `total` = count of all matches; `pagination.next` / `prev` = page numbers or `null` |

**Search response note:** the rows go in `data` (an array) and the paging metadata in `items`. This keeps `data` meaning "the payload" for every action. Calculation: `next = page * pageSize < total ? page + 1 : null`, `prev = page > 1 ? page - 1 : null`.

**Create** returns 201 with the created entity (or at least `{ id }`). **Delete** returns 200 with `{ id }` (not 204, because 204 can't carry the envelope).

**Validation errors:** `data.errors[]` with `path` (dot path into the request, e.g. `data.name`, `filter.status.0`) and `message`. The FE maps `path` to form fields.

---

## 5. Response Codes

| Rule | Example |
|---|---|
| Format `<MODULE>_<ACTION>_<RESULT>` | `PROJECT_EDIT_SUCCESS` |
| `<MODULE>` = module name in UPPER_SNAKE | `MILESTONE_TASK_…` |
| Success code for every endpoint | `<MODULE>_<ACTION>_SUCCESS` |
| Not found per module | `<MODULE>_NOT_FOUND` |
| Business rule failures name the rule | `PROJECT_CLOSE_HAS_OPEN_ISSUES` |
| Codes are permanent | Never reuse or rename a published code |
| Declared in constants | `src/constants/<module>.constants.ts` exports the codes as a const object |

**HTTP status guide:** 400 validation, 401 not logged in, 403 not allowed, 404 not found, 409 business rule / state conflict / duplicate, 422 not used (use 400 or 409), 429 rate limit, 500 unexpected.

§5.3 lists every module code with its HTTP status, endpoints, when it happens, and the user message (the FE can show it as-is).

---

## 6. Authentication & Middleware

| Section | What to write |
|---|---|
| 6.1 Auth Flow | How the FE gets the token (NextAuth) and how the BE verifies it; reference ADB 02 §6.1 |
| 6.2 Middleware Chain | Order matters; the pre-filled order is the default. Add project-specific middleware with its file and scope |
| 6.3 Permission Map | Endpoint × role, plus ownership rule. Must cover **every** endpoint in §7 and agree with BRD §4.4 |

**Permission keys:** `<module>.<action>` (`project.edit`, `milestone.create`) used by `permit()` in the route file. Store the role → keys mapping in `src/constants/permission.constants.ts` (or the DB if roles are configurable).

---

## 7. Route Catalog
All endpoints, grouped by module then submodule, in folder order.

| Field | Meaning | Example |
|---|---|---|
| Method / Path | Full path including `/api/v1` | `PUT /api/v1/project/:projectId/close` |
| Action | create / search / get / edit / delete / extra action name | `close` |
| Description | One line | Close a project |
| Feature (BRD) | Feature ID | F-PRJ-03 |
| Roles | Allowed roles | ROLE-01, ROLE-02 |
| Tables | Tables touched | projects, milestones, issues (read) |

**§7.2 External API:** endpoints under `/api-ext/v1` for machines (partners, integrations, webhooks). Separate router, separate auth (API key / HMAC signature), defined in `<module>.extRoute.ts`.

---

## 8. Endpoint Details
One block per endpoint, ID `EP-<CODE>-NN`.

| Part | What to write |
|---|---|
| Header table | Description, feature, workflow, roles, ownership rule, files, tables, idempotent |
| Path parameters | Each `:param`, type, rule |
| Body — `data` fields | Create/edit: type, required on create, editable on edit, rule, example |
| Search — filter & sort | Each filter key → column and operator; sortable fields and default |
| Zod schema | The actual schema; validate `params`, `query`, `body` in one object |
| Success response | TS type of `data` |
| Response codes | Every code this endpoint can return besides the global ones |
| Business rules | Which BR- rules the service enforces and the code returned when violated |
| Side effects | Audit log, notifications, jobs queued, cache invalidated |

**Idempotent:** GET, PUT edit, and DELETE should be safe to retry. POST create is not; add an `Idempotency-Key` header only if the BRD needs it (payments, imports).

---

## 9. Folder Structure

### 9.1 Next.js-style folders
The folder path mirrors the URL path, without `/api/v1`, path params, and the action:

| URL | Folder |
|---|---|
| `/api/v1/project/create` | `src/api/project/` |
| `/api/v1/project/:projectId/milestone/search` | `src/api/project/milestone/` |
| `/api/v1/project/:projectId/milestone/:milestoneId/task/get` | `src/api/project/milestone/task/` |

Each module folder holds **one set of 5 files** covering all its actions. A submodule folder holds its own 5 files and is mounted by the parent's route file with `Router({ mergeParams: true })` so it can read the parent ID.

### 9.2 File responsibilities

| File | Contains | Naming inside |
|---|---|---|
| `<module>.route.ts` | Express Router: path, method, `permit()`, `validate()`, handler; mounts submodule routers | export `<module>Router` |
| `<module>.handler.ts` | One function per action; reads validated input, calls service, returns envelope with `utils/response` helpers | `createProject`, `searchProject`, `getProject`, `editProject`, `deleteProject` |
| `<module>.validation.ts` | One zod schema per action + inferred types | `createProjectSchema`, `type CreateProjectInput = z.infer<…>` |
| `<module>.service.ts` | One function per action + private helpers; DB queries, transactions, business rules, throws `AppError` | same names as handlers |
| `<module>.docs.ts` | OpenAPI registration for each route, built from the zod schemas | `registerProjectDocs(registry)` |
| `<module>.extRoute.ts` *(optional)* | External endpoints mounted on `apiExt.routes.ts` | export `<module>ExtRouter` |
| `_<module>.utils.ts` *(optional)* | Private helpers used only inside this folder | — |

**Layer rule:** route → handler → service → DB. A handler never queries the DB; a service never touches `req`/`res`. Services may call other modules' services (not their handlers).

**Name collisions:** if two modules have a submodule with the same name (`project/member`, `team/member`), the folders are fine but exported symbols collide; prefix the export (`projectMemberRouter`, `teamMemberRouter`).

### 9.4 Folder Index
One row per folder in `src/api/`: module code, URL prefix, number of endpoints, owner. This is the quick map for new developers.

### 9.5 Reference skeletons
Show the real pattern for one module. Keep them in the book so AI agents and new developers copy the right structure.

---

## 10. Configuration
Every env variable: required, example (never a real secret), description. Validated at startup in `src/config/` with zod so a missing variable fails fast.

## 11. Background Jobs
Maps each ADB 02 `JOB-NN` to its file, schedule/queue, and the service function it calls. Jobs call services; they don't contain business logic.

## 12. Error Handling & Logging Rules
Services throw `AppError(code, status, message)`. The error middleware converts it to the envelope. Unknown errors become `INTERNAL_ERROR` and are logged with `requestId`. List anything that must never be logged.

## 13. Open Questions
Global numbering shared with all books.
