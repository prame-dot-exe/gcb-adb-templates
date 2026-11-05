# ADB Conventions (shared standards)

The standard rules every project follows. The five books **link here instead of copying these rules**. A book section marked **Standard** only lists what the project does differently, and every difference needs an ADR in ADB 02 §13.

| §   | Covers                | Replaces in the books                                       |
| --- | --------------------- | ----------------------------------------------------------- |
| 1   | Target stack          | Fixed rows of SRS §2.4, BE §1, FE §1                        |
| 2   | Database              | DB §1                                                       |
| 3   | API contract          | BE §2–4, response code format and global codes              |
| 4   | Backend code          | BE §6.2, §9, §12                                            |
| 5   | Frontend code         | FE §2, token rules in §3, §6, §8–10, accessibility baseline |
| 6   | Delivery & operations | SRS §6.2, §8, §9, §10.2, §11                                |

---

## 1. Target Stack

| Layer            | Fixed choice                                                                   |
| ---------------- | ------------------------------------------------------------------------------ |
| Frontend         | Next.js (App Router) · shadcn/ui · Tailwind CSS · ESLint · TypeScript `strict` |
| Backend          | Node.js · Express.js · TypeScript `strict` · Zod                               |
| Database         | PostgreSQL                                                                     |
| Auth             | NextAuth on the frontend; the backend verifies the JWT                         |
| API docs         | OpenAPI generated from `*.docs.ts`                                             |
| Manual API tests | Bruno (`bruno-API/`)                                                           |

Open choices each project records in SRS §2.4: query/migration tool, cache/queue, file storage, hosting, CI/CD, monitoring, server-state library, i18n, package manager.

---

## 2. Database

### 2.1 Naming and types

| Item                       | Rule                                                                                                 | Why                                          |
| -------------------------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| Table names                | snake_case, plural (`projects`, `project_members`)                                                   | PostgreSQL folds unquoted names to lowercase |
| Column names               | snake_case (`created_at`)                                                                            | Same                                         |
| Primary key                | `id uuid DEFAULT gen_random_uuid()`                                                                  | Safe in URLs, no enumeration                 |
| Foreign key                | `<singular_table>_id` (`project_id`)                                                                 | Readable joins                               |
| Timestamps                 | `timestamptz`, stored in UTC                                                                         | Correct across timezones                     |
| Calendar days              | `date`                                                                                               | No fake time component                       |
| Money                      | `numeric(14,2)`; never float                                                                         | Floats lose cents                            |
| Text with a business limit | `varchar(n)`, same `n` as the zod `max()`                                                            | One limit everywhere                         |
| Booleans                   | `is_` / `has_` prefix                                                                                | Reads as a question                          |
| Enums                      | PostgreSQL ENUM for stable sets (statuses); lookup table when admins add values; CHECK for tiny sets |                                              |
| Enum values                | lowercase snake_case                                                                                 | Matches state diagrams and FE status map     |
| JSONB                      | Only for flexible, non-queried data; shape documented in DB §5                                       | Queried fields deserve columns and indexes   |
| Soft delete                | `deleted_at timestamptz NULL` on business tables; every query filters `deleted_at IS NULL`           | "Delete" is recoverable                      |
| Index names                | `idx_<table>_<columns>`; unique `uq_<table>_<columns>`                                               | Predictable migrations                       |
| Constraint names           | `fk_<table>_<ref_table>`, `ck_<table>_<rule>`                                                        | Readable errors                              |
| Migration files            | `migrations/v-<x>-<y>-<z>_<topic>.ts`                                                                |                                              |
| Seed files                 | `seeds/v-<x>-<y>-<z>_<topic>.ts`; master data seeds are idempotent                                   |                                              |
| Schema version             | `v-<major>-<minor>-<patch>`: major = drop/rename, minor = new table/column, patch = index/comment    |                                              |
| DB ↔ API naming            | snake_case in SQL, camelCase in JSON and TS; mapped in the DB layer, never in handlers               |                                              |

### 2.2 Standard Columns

Every business table has these. The DB book does not repeat them in each table.

```sql
id          uuid        PRIMARY KEY DEFAULT gen_random_uuid(),
created_at  timestamptz NOT NULL DEFAULT now(),
created_by  uuid        NULL REFERENCES users(id),
updated_at  timestamptz NOT NULL DEFAULT now(),
updated_by  uuid        NULL REFERENCES users(id),
deleted_at  timestamptz NULL
```

Junction tables may skip `updated_*` and `deleted_at` if rows are only inserted and removed.

### 2.3 Relationships and indexes

- `ON DELETE RESTRICT` by default for business data; `CASCADE` for junction and owned detail rows; `SET NULL` for optional links (`assignee_id`). With soft delete, the service cascades `deleted_at` to children.
- Indexes on business tables are partial: `WHERE deleted_at IS NULL`.
- Every filter and sort field of a search endpoint is covered by an index, unless the table is tiny. `keyword` ILIKE search uses a `pg_trgm` GIN index.
- Migrations follow expand → migrate → contract so the previous app version keeps working during deploy. Never edit an applied migration.

### 2.4 SQL → TypeScript

| SQL                 | TypeScript                                    |
| ------------------- | --------------------------------------------- |
| uuid, text, varchar | `string`                                      |
| integer, smallint   | `number`                                      |
| bigint, numeric     | `string` (use a decimal library to calculate) |
| boolean             | `boolean`                                     |
| date, timestamptz   | `string` (ISO)                                |
| jsonb               | named type from DB §5                         |
| enum                | string union                                  |
| nullable            | `T \| null`                                   |

Types live in `src/types/db/<module>.types.ts`, camelCase, one type per table.

---

## 3. API Contract

### 3.1 General

| Item               | Convention                                                                                           |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| Internal base path | `/api/v1`                                                                                            |
| External base path | `/api-ext/v1` (partners, webhooks; API key or HMAC auth)                                             |
| Versioning         | Major version in the path                                                                            |
| Path segments      | camelCase, singular module names (`/project`, `/milestoneTask`)                                      |
| JSON keys          | camelCase                                                                                            |
| IDs                | uuid strings                                                                                         |
| Dates              | ISO 8601 in UTC (`2026-11-05T06:00:00.000Z`)                                                         |
| Auth header        | `Authorization: Bearer <token>`                                                                      |
| Content type       | `application/json` (`multipart/form-data` only for uploads)                                          |
| Methods            | `GET`, `POST`, `PUT`, `DELETE` only                                                                  |
| Action suffix      | Every path ends with an action: `/create`, `/search`, `/get`, `/edit`, `/delete`, or an extra action |

### 3.2 Endpoint pattern

| Action | Method   | Path                                  | Body                           | Success |
| ------ | -------- | ------------------------------------- | ------------------------------ | ------- |
| create | `POST`   | `/api/v1/<module>/create`             | `{ data }`                     | 201     |
| search | `POST`   | `/api/v1/<module>/search`             | `{ filter, sort, pagination }` | 200     |
| get    | `GET`    | `/api/v1/<module>/:<module>Id/get`    | —                              | 200     |
| edit   | `PUT`    | `/api/v1/<module>/:<module>Id/edit`   | `{ data }`                     | 200     |
| delete | `DELETE` | `/api/v1/<module>/:<module>Id/delete` | —                              | 200     |

- **Submodules** nest under the parent record: `/api/v1/project/:projectId/milestone/create`, `/api/v1/project/:projectId/milestone/:milestoneId/edit`.
- **Path params** are always named after the module (`:projectId`, never `:id`).
- **Extra actions** (state changes, exports, bulk operations): `PUT …/:<module>Id/<verb>` for transitions (`PUT /api/v1/project/:projectId/close`), `POST …/<verbNoun>` for processes (`POST /api/v1/project/export`). Each one is listed in the route catalog with a reason.
- **Delete** is a soft delete. A hard delete needs a reason in the endpoint detail.
- **search is always POST** so filters can be nested and long.

### 3.3 Request bodies

| Action       | Body                                                | Rule                                                                           |
| ------------ | --------------------------------------------------- | ------------------------------------------------------------------------------ |
| create       | `{ "data": { … } }`                                 | All required fields                                                            |
| edit         | `{ "data": { … } }`                                 | Only changed fields. Never accepts `id`, `createdAt`, `createdBy`, `deletedAt` |
| search       | `{ "filter": {…}, "sort": {…}, "pagination": {…} }` | All optional                                                                   |
| get / delete | none                                                | Query params only where listed (e.g. `?include=members`)                       |

```json
{
  "filter": {
    "keyword": "alpha",
    "status": ["active"],
    "createdAtFrom": "2026-01-01T00:00:00.000Z",
    "createdAtTo": "2026-12-31T23:59:59.999Z"
  },
  "sort": { "field": "createdAt", "order": "desc" },
  "pagination": { "page": 1, "pageSize": 20 }
}
```

| Key                         | Default     | Rule                                                                          |
| --------------------------- | ----------- | ----------------------------------------------------------------------------- |
| `filter`                    | `{}`        | Only keys listed for the endpoint; unknown keys fail validation (`.strict()`) |
| `filter.keyword`            | —           | Free text over the endpoint's keyword columns                                 |
| `filter.<field>`            | —           | Value or array of values (IN)                                                 |
| `filter.<field>From` / `To` | —           | Range                                                                         |
| `filter.<entity>Id`         | —           | Relation                                                                      |
| `sort.field`                | `createdAt` | Only fields listed as sortable                                                |
| `sort.order`                | `desc`      | `asc` / `desc`                                                                |
| `pagination.page`           | `1`         | Integer ≥ 1                                                                   |
| `pagination.pageSize`       | `20`        | Integer 1–100                                                                 |

```ts
// src/types/api.types.ts
export type CreateBody<T> = { data: T };
export type EditBody<T> = { data: Partial<T> };
export type SearchBody<F> = {
  filter?: F & { keyword?: string };
  sort?: { field: string; order: "asc" | "desc" };
  pagination?: { page: number; pageSize: number };
};
```

### 3.4 Response envelope

```ts
// src/types/api.types.ts
export type ApiResponse<T = unknown> = {
  success: boolean;
  message: string; // Human-readable, safe to show the user
  code: string; // Stable machine code; the FE branches on this, never on message
  data?: T; // Object for get/create/edit/delete, array of rows for search
  items?: {
    // Search only
    total: number;
    pagination: { next: number | null; prev: number | null };
  };
};
```

- `next = page * pageSize < total ? page + 1 : null`, `prev = page > 1 ? page - 1 : null`.
- Create returns 201 with the created entity. Delete returns 200 with `{ id }`.
- Validation errors return `data.errors[]` with `path` (e.g. `data.name`) and `message`.

```json
{
  "success": true,
  "message": "Projects retrieved",
  "code": "PROJECT_SEARCH_SUCCESS",
  "data": [{ "id": "3f2a…", "name": "Alpha" }],
  "items": { "total": 125, "pagination": { "next": 3, "prev": 1 } }
}
```

```json
{
  "success": false,
  "message": "Invalid input",
  "code": "VALIDATION_ERROR",
  "data": { "errors": [{ "path": "data.name", "message": "Required" }] }
}
```

### 3.5 Response codes

- Format `<MODULE>_<ACTION>_<RESULT>` in UPPER_SNAKE_CASE: `PROJECT_CREATE_SUCCESS`, `PROJECT_NOT_FOUND`, `PROJECT_CLOSE_HAS_OPEN_ISSUES`.
- Every endpoint has a success code; every module has `<MODULE>_NOT_FOUND`; business-rule failures name the rule.
- Codes are permanent. Declared in `src/constants/<module>.constants.ts`.
- HTTP: 400 validation, 401 not logged in, 403 not allowed, 404 not found, 409 business rule / conflict / duplicate, 429 rate limit, 500 unexpected. 422 is not used.

**Global codes**

| Code                | HTTP | When                                      |
| ------------------- | ---- | ----------------------------------------- |
| VALIDATION_ERROR    | 400  | Zod validation failed                     |
| UNAUTHENTICATED     | 401  | Missing / invalid / expired token         |
| FORBIDDEN           | 403  | Authenticated but not allowed             |
| NOT_FOUND           | 404  | Route does not exist                      |
| CONFLICT            | 409  | Generic state conflict                    |
| PAYLOAD_TOO_LARGE   | 413  | Body or file too large                    |
| RATE_LIMITED        | 429  | Too many requests                         |
| INTERNAL_ERROR      | 500  | Unexpected failure (details only in logs) |
| SERVICE_UNAVAILABLE | 503  | Dependency down / maintenance             |

---

## 4. Backend Code

### 4.1 Folders mirror the URL

The folder path is the URL without `/api/v1`, path params, and the action:

| URL                                           | Folder                       |
| --------------------------------------------- | ---------------------------- |
| `/api/v1/project/create`                      | `src/api/project/`           |
| `/api/v1/project/:projectId/milestone/search` | `src/api/project/milestone/` |

```text
[project]-BE/
├── AGENTS.md                         # Agent single source of truth
├── .github/workflows/                # CI / deploy
├── bruno-API/                        # Manual API tests, one folder per module
├── migrations/ · seeds/
├── dev/docs/ · dev/plan/ · dev/report/ · dev/db-design/ · dev/set-up-environment/
└── src/
    ├── server.ts · app.ts            # HTTP server; Express app, global middleware, version mounts
    ├── config/                       # Typed env, validated with zod at startup
    ├── constants/                    # <domain>.constants.ts (incl. response codes, permissions)
    ├── types/                        # api.types.ts, db/*.types.ts, express.d.ts
    ├── middleware/                   # <concern>.middleware.ts
    ├── jobs/                         # Background jobs (call services, no logic)
    ├── utils/                        # db, logger, response.ts (ok, created, fail, searchResult), <domain>/
    └── api/
        ├── api.routes.ts             # /api/v1
        ├── apiExt.routes.ts          # /api-ext/v1
        ├── api.docs.ts
        └── <module>/
            ├── <module>.route.ts · .handler.ts · .validation.ts · .service.ts · .docs.ts
            ├── <module>.extRoute.ts  # optional, external endpoints
            ├── _<module>.utils.ts    # optional, private helpers
            └── <submodule>/          # same 5 files; mounted with Router({ mergeParams: true })
```

### 4.2 File responsibilities

| File             | Does                                                                                                 | Must not            |
| ---------------- | ---------------------------------------------------------------------------------------------------- | ------------------- |
| `.route.ts`      | Paths, methods, `permit('<module>.<action>')`, `validate(schema)`, handler; mounts submodule routers | Contain logic       |
| `.handler.ts`    | Read validated input, call service, return the envelope via `utils/response`                         | Query the DB        |
| `.validation.ts` | One zod schema per action (`params`, `query`, `body`) + inferred types                               | Import services     |
| `.service.ts`    | Business rules, DB access, transactions; throws `AppError(code, status, message)`                    | Touch `req` / `res` |
| `.docs.ts`       | OpenAPI registration built from the zod schemas                                                      | Contain logic       |

Function names per action: `createProject`, `searchProject`, `getProject`, `editProject`, `deleteProject`. Same-named submodules in different parents prefix their exports (`projectMemberRouter`).

```ts
// src/api/project/project.route.ts
export const projectRouter = Router();
projectRouter.post(
  "/create",
  permit("project.create"),
  validate(v.createProjectSchema),
  h.createProject,
);
projectRouter.post(
  "/search",
  permit("project.search"),
  validate(v.searchProjectSchema),
  h.searchProject,
);
projectRouter.get(
  "/:projectId/get",
  permit("project.get"),
  validate(v.getProjectSchema),
  h.getProject,
);
projectRouter.put(
  "/:projectId/edit",
  permit("project.edit"),
  validate(v.editProjectSchema),
  h.editProject,
);
projectRouter.delete(
  "/:projectId/delete",
  permit("project.delete"),
  validate(v.deleteProjectSchema),
  h.deleteProject,
);
projectRouter.use("/:projectId/milestone", milestoneRouter);
```

### 4.3 Middleware chain

| Order | Middleware   | Applies to                | Purpose                          |
| ----- | ------------ | ------------------------- | -------------------------------- |
| 1     | `requestId`  | All                       | Adds `x-request-id`              |
| 2     | `logger`     | All                       | Structured access log            |
| 3     | `security`   | All                       | helmet, CORS                     |
| 4     | `rateLimit`  | All; stricter on login    | Limits                           |
| 5     | `auth`       | `/api/v1/*` except public | Verify token, set `req.user`     |
| 6     | `permission` | Per route                 | Permission key + ownership check |
| 7     | `validate`   | Per route                 | Run zod schema                   |
| last  | `error`      | All                       | Map errors to the envelope       |

Permission keys are `<module>.<action>`; the role → keys map lives in `src/constants/permission.constants.ts` (or the DB if roles are configurable).

### 4.4 Errors and logging

| Rule           | Detail                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------- |
| Thrown errors  | Services throw `AppError`; the error middleware builds the envelope                                                 |
| Unknown errors | Logged with `requestId`; client gets `INTERNAL_ERROR` with no details                                               |
| Never log      | Passwords, tokens, card numbers, other PII listed in DB §7                                                          |
| Log format     | JSON: `timestamp, level, requestId, userId, method, path, status, durationMs`                                       |
| Idempotency    | GET, edit, delete are safe to retry; add `Idempotency-Key` on create only when the BRD needs it (payments, imports) |

---

## 5. Frontend Code

### 5.1 Folders

**Rule:** business UI lives in `src/feature/`. `src/components/` holds only generic UI. If a component imports a module type, calls a module API, or shows a module word, it is a feature.

```text
[project]-FE/src/
├── auth.ts                          # NextAuth config
├── app/                             # Routing only: thin pages that render a feature
│   └── <scope>/<module>/
│       ├── page.tsx                 # → feature/<module>/search
│       ├── create/page.tsx          # → feature/<module>/create
│       └── [<module>Id]/
│           ├── page.tsx             # → feature/<module>/view
│           ├── edit/page.tsx        # → feature/<module>/edit
│           └── <submodule>/…
├── components/
│   ├── ui/                          # shadcn CLI output (keep shadcn's component API)
│   ├── primitive/ · form/ · complex/ · featureUi/
├── feature/
│   ├── <module>/<action>/           # search, create, edit, view, viewDrawer, summary, <submodule>/
│   └── share/<component>/           # Business components used by 2+ modules
├── dataProvider/<scope>/            # Scope data: current user, permissions
├── hooks/ · context/ · navigation/<scope>/
├── libs/api/<module>/<submodule>/<action><Entity>.ts   # One wrapper per endpoint
├── libs/api/client.ts               # Auth header, JSON, envelope parsing, ApiError
├── style/globals.css                # Tailwind theme + shadcn tokens
└── types/                           # api.types.ts mirrors §3
```

**Feature folder** `feature/<module>/<action>/`: `<Module><Action>.tsx` (entry), `<module><Action>.hook.ts` (data + state), `.type.ts`, `.schema.ts` (zod form, create/edit), `.column.tsx` (table, search), `components/`.

| Action folder           | Renders                   | Calls                         |
| ----------------------- | ------------------------- | ----------------------------- |
| `search/`               | List + filters            | `search<Entity>`              |
| `create/`               | Create form               | `create<Entity>`              |
| `edit/`                 | Edit form                 | `get<Entity>`, `edit<Entity>` |
| `view/` / `viewDrawer/` | Detail page / side drawer | `get<Entity>`                 |
| `summary/`              | Dashboard cards           | summary endpoint              |

### 5.2 Design token rules

1. Components use semantic classes only (`bg-primary`, `text-muted-foreground`). No hex, rgb, or palette colors (`bg-blue-500`).
2. Every surface token has a `-foreground` pair; text on a surface uses its pair.
3. Tokens are defined once in `src/style/globals.css`: values in `:root` and `.dark`, mapped in `@theme inline`. Values use `oklch()`.
4. Start from a shadcn base theme (neutral / zinc / slate). The FE book lists only the tokens the project changes.
5. Extra tokens follow the shadcn pattern: `--success` / `--success-foreground`, `--warning`, `--info`.
6. Contrast: 4.5:1 for body text, 3:1 for large text and UI borders, in both modes.
7. Spacing, radius, shadows, breakpoints and motion use Tailwind / shadcn defaults (`--radius: 0.625rem`); respect `prefers-reduced-motion`.

```css
/* src/style/globals.css */
@import "tailwindcss";
@import "tw-animate-css";
@custom-variant dark (&:is(.dark *));
:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --primary: oklch(…); /* … */
}
.dark {
  --background: oklch(0.145 0 0); /* … */
}
@theme inline {
  --color-background: var(--background);
  --color-primary: var(--primary); /* one per token */
}
```

### 5.3 API layer

| Rule         | Detail                                                                                                     |
| ------------ | ---------------------------------------------------------------------------------------------------------- |
| Location     | `src/libs/api/<module>/<submodule>/<action><Entity>.ts`, one file per endpoint, folders mirror the backend |
| Client       | Every wrapper calls `libs/api/client.ts`; `success: false` throws `ApiError(code, message, data)`          |
| Types        | `ApiResponse`, `SearchBody`, `CreateBody`, `EditBody` mirror §3                                            |
| Search state | Filter, sort and page live in the URL (`useUrlSearch`)                                                     |

```ts
// src/libs/api/project/searchProject.ts
export const searchProject = (body: SearchBody<ProjectFilter>) =>
  apiClient.post<ApiResponse<Project[]>>("/api/v1/project/search", body);
```

**Default error handling:**

| Code                 | UI behavior                             |
| -------------------- | --------------------------------------- |
| VALIDATION_ERROR     | Map `data.errors[].path` to form fields |
| UNAUTHENTICATED      | Sign out, redirect to /login            |
| FORBIDDEN            | Toast "You don't have permission"       |
| `<MODULE>_NOT_FOUND` | Redirect to the list with a toast       |
| INTERNAL_ERROR       | Toast with retry                        |
| Anything else        | Show `message` in a toast               |

### 5.4 Screen defaults

Every screen handles these states the same way unless the FE book says otherwise:

| State         | Default                                     |
| ------------- | ------------------------------------------- |
| Loading       | Skeleton                                    |
| Empty         | `featureUi/emptyState` with the main action |
| Error         | Inline alert with retry                     |
| No permission | Hide the action; direct URL shows 403       |

### 5.5 State, forms, auth

| Concern          | Rule                                                                                                           |
| ---------------- | -------------------------------------------------------------------------------------------------------------- |
| Server data      | Data library cache (e.g. TanStack Query), keys `[module, action, params]`; invalidate after create/edit/delete |
| Scope data       | `dataProvider/<scope>/`                                                                                        |
| Global UI state  | `context/` (theme, toasts) only                                                                                |
| Forms            | react-hook-form + zod schema mirroring the backend; the backend stays the source of truth                      |
| Route protection | Scope `layout.tsx` checks the session; actions hidden by role, but the API always enforces                     |
| Server vs client | Pages are Server Components; stateful feature components use `'use client'`                                    |

### 5.6 Naming and ESLint

| Item         | Rule                                                                                                            |
| ------------ | --------------------------------------------------------------------------------------------------------------- |
| Folders      | camelCase (`viewDrawer`)                                                                                        |
| Components   | PascalCase file and export (`ProjectSearch.tsx`)                                                                |
| Hooks        | `use<Name>` from `<name>.hook.ts`                                                                               |
| Imports      | Absolute `@/`                                                                                                   |
| Boundaries   | `components/` never imports `feature/`; `feature/<a>` never imports `feature/<b>` (use `feature/share`)         |
| ESLint       | `next/core-web-vitals`, `next/typescript`, `no-restricted-imports` for the boundaries, `no-explicit-any: error` |
| CI must pass | `eslint .`, `tsc --noEmit`, build                                                                               |

### 5.7 Accessibility baseline

WCAG 2.2 AA · every action reachable by keyboard with a visible focus ring · every input labeled, icon buttons have `aria-label` · touch targets ≥ 44×44 px · dates stored in UTC, shown in the user's timezone.

---

## 6. Delivery & Operations

### 6.1 Authorization mechanism

`auth` middleware sets `req.user`; `permission` middleware checks the route's permission key against the user's roles; services check ownership. Projects list only their roles (BRD §2.1), permissions (BRD §4.4) and ownership rules (BE §6.3).

### 6.2 Security baseline

| Area          | Standard                                                                  |
| ------------- | ------------------------------------------------------------------------- |
| Transit       | HTTPS everywhere, HSTS                                                    |
| At rest       | Encrypted database and backups                                            |
| Secrets       | Secret manager / CI secrets; `.env` never committed                       |
| Input         | Zod on params, query and body of every endpoint                           |
| Rate limiting | Global per IP; stricter on login                                          |
| CORS          | Only the frontend origins of each environment                             |
| Headers       | helmet defaults; CSP on the frontend                                      |
| Audit log     | Who / what / when for create, edit, delete, status and permission changes |
| Dependencies  | Dependabot or `npm audit` in CI                                           |
| Uploads       | Allowed types, size limit, private bucket, signed URLs                    |

### 6.3 Branches and pipeline

- Trunk-based: short `feat/…` / `fix/…` branches into `main`; PRs need one review and green checks.
- Pipeline: lint + typecheck → unit tests → build → migrate → deploy → smoke test (`/health`). Merge to `dev` deploys Dev, merge to `main` deploys Staging, a tag + approval deploys Production.
- Rollback: redeploy the previous image; migrations are backward compatible (§2.3), so the database is not rolled back.

### 6.4 Observability

Structured JSON logs with `requestId`; metrics for latency, error rate and CPU; error tracking on frontend and backend; uptime check on `/health`; alerts on failed jobs. Every alert has a channel and an owner.

### 6.5 Backups

Managed PostgreSQL daily snapshot + point-in-time recovery, 30 days, stored in another region, encrypted. Object storage versioned. Restore to staging tested every quarter. Projects change this only if SRS §10.1 targets need more.

### 6.6 Testing

| Type                  | Scope                                            | When           |
| --------------------- | ------------------------------------------------ | -------------- |
| Unit (BE)             | `*.service.ts` (business rules), target 80%      | Every PR       |
| Integration (BE)      | Routes + DB                                      | Every PR       |
| Manual API            | Bruno collections mirror the route catalog       | Before release |
| Unit / component (FE) | Feature hooks, utils                             | Every PR       |
| E2E                   | The few flows that lose money or trust if broken | Before release |
