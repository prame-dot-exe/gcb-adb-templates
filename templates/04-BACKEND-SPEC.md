# [Project Name] — ADB 04: Backend Specification

> **Application Design Book (ADB) — Book 4 of 5.** API conventions, request/response contracts, the full route catalog, endpoint details, and the backend folder structure (TypeScript + Express.js, folder-per-module like Next.js).
> Guideline: `guidelines/04-BACKEND-SPEC.guide.md`. Replace every `[placeholder]`. Use `N/A — reason` and `TBD — see Q-xx`.
> **Fixed conventions** (sections 2–5 and 9) are pre-filled. Change them only through an ADR in ADB 02.

---

## 0. Document Info

| Field | Value |
|---|---|
| Project name | [Name] |
| Project code | [Code] |
| Book | ADB 04 — Backend Spec |
| Version | [0.1.0] |
| API version | v1 |
| Status | [Draft / In Review / Approved / Deprecated] |
| Author(s) | [Name, role] |
| Reviewer(s) | [Name, role] |
| Created | [YYYY-MM-DD] |
| Last updated | [YYYY-MM-DD] |
| Repository | [URL] |
| OpenAPI / Swagger URL | [URL per environment] |

### Change Log

| Version | Date | Author | Summary of changes |
|---|---|---|---|
| 0.1.0 | [YYYY-MM-DD] | [Name] | Initial draft |

---

## 1. Tech Summary

| Item | Value |
|---|---|
| Runtime | Node.js [version] |
| Language | TypeScript [version], `strict: true` |
| Framework | Express.js [version] |
| Validation | Zod [version] |
| DB access | [Knex / Drizzle / Prisma] → PostgreSQL |
| Auth | [JWT verified from NextAuth / own issuer] |
| API docs | OpenAPI generated from `*.docs.ts` |
| Manual API tests | Bruno (`bruno-API/`) |
| Package manager | [pnpm / npm] |

---

## 2. API Conventions (fixed)

| Item | Convention |
|---|---|
| Internal base path | `/api/v1` |
| External base path | `/api-ext/v1` (partner / machine access only) |
| Versioning | Major version in path (`/v1`, `/v2`) |
| Path segments | camelCase module names (`/project`, `/milestoneTask`) — singular |
| JSON keys | camelCase |
| IDs | uuid strings |
| Dates | ISO 8601 strings in UTC (`2026-11-05T06:00:00.000Z`) |
| Auth header | `Authorization: Bearer <token>` |
| Content type | `application/json` (`multipart/form-data` only for uploads) |
| Allowed methods | `GET`, `POST`, `PUT`, `DELETE` only |
| Action suffix | Every endpoint ends with an action: `/create`, `/edit`, `/search`, `/delete`, `/get` |

### 2.1 Endpoint Pattern

| Action | Method | Path | Body | Success status |
|---|---|---|---|---|
| Create | `POST` | `/api/v1/<module>/create` | `{ data }` | 201 |
| Search | `POST` | `/api/v1/<module>/search` | `{ filter, sort, pagination }` | 200 |
| Get one | `GET` | `/api/v1/<module>/:<module>Id/get` | — | 200 |
| Edit | `PUT` | `/api/v1/<module>/:<module>Id/edit` | `{ data }` | 200 |
| Delete | `DELETE` | `/api/v1/<module>/:<module>Id/delete` | — | 200 |

**Submodules** nest under the parent record:

| Action | Method | Path |
|---|---|---|
| Create | `POST` | `/api/v1/<module>/:<module>Id/<submodule>/create` |
| Search | `POST` | `/api/v1/<module>/:<module>Id/<submodule>/search` |
| Get one | `GET` | `/api/v1/<module>/:<module>Id/<submodule>/:<submodule>Id/get` |
| Edit | `PUT` | `/api/v1/<module>/:<module>Id/<submodule>/:<submodule>Id/edit` |
| Delete | `DELETE` | `/api/v1/<module>/:<module>Id/<submodule>/:<submodule>Id/delete` |

**Extra actions** (state transitions, exports, bulk operations) follow the same rule: `<METHOD> …/:<id>/<verbNoun>` — for example `PUT /api/v1/project/:projectId/close`, `POST /api/v1/project/export`. Each extra action must be listed in the route catalog with a reason.

---

## 3. Request Body Structure (fixed)

### 3.1 Create / Edit

```json
{
  "data": {
    "[field]": "[value]"
  }
}
```

- `create`: `data` contains every required field.
- `edit`: `data` contains only the fields to change; omitted fields are left as they are. `id` is never in the body; it comes from the path.

### 3.2 Search

```json
{
  "filter": {
    "keyword": "string",
    "[field]": "[value or array of values]",
    "[dateField]From": "2026-01-01T00:00:00.000Z",
    "[dateField]To": "2026-12-31T23:59:59.999Z"
  },
  "sort": { "field": "createdAt", "order": "desc" },
  "pagination": { "page": 1, "pageSize": 20 }
}
```

| Key | Required | Default | Rule |
|---|---|---|---|
| `filter` | No | `{}` | Only fields listed in the endpoint's filter table |
| `filter.keyword` | No | — | Free-text search over the endpoint's keyword columns |
| `sort.field` | No | `createdAt` | Only fields listed as sortable |
| `sort.order` | No | `desc` | `asc` / `desc` |
| `pagination.page` | No | `1` | Integer ≥ 1 |
| `pagination.pageSize` | No | `20` | Integer 1–[100] |

### 3.3 Get / Delete
No body. IDs come from the path. Optional query parameters only where listed (for example `?include=members`).

### 3.4 TypeScript

```ts
// src/types/api.types.ts
export type CreateBody<T> = { data: T };
export type EditBody<T> = { data: Partial<T> };

export type SearchBody<F> = {
  filter?: F & { keyword?: string };
  sort?: { field: string; order: 'asc' | 'desc' };
  pagination?: { page: number; pageSize: number };
};
```

---

## 4. Response Structure (fixed)

### 4.1 Envelope

```ts
// src/types/api.types.ts
export type ApiResponse<T = unknown> = {
  success: boolean;
  message: string;      // Human-readable, safe to show the user
  code: string;         // Stable machine code, see §5
  data?: T;             // Payload: object for get/create/edit/delete, array for search
  items?: {             // Search only
    total: number;      // Total matching records (all pages)
    pagination: {
      next: number | null; // Next page number, null on last page
      prev: number | null; // Previous page number, null on first page
    };
  };
};
```

### 4.2 Examples

**Create — 201**
```json
{ "success": true, "message": "Project created", "code": "PROJECT_CREATE_SUCCESS", "data": { "id": "3f2a…", "name": "Alpha" } }
```

**Get — 200**
```json
{ "success": true, "message": "Project retrieved", "code": "PROJECT_GET_SUCCESS", "data": { "id": "3f2a…", "name": "Alpha" } }
```

**Search — 200**
```json
{
  "success": true,
  "message": "Projects retrieved",
  "code": "PROJECT_SEARCH_SUCCESS",
  "data": [ { "id": "3f2a…", "name": "Alpha" } ],
  "items": { "total": 125, "pagination": { "next": 3, "prev": 1 } }
}
```

**Edit — 200**
```json
{ "success": true, "message": "Project updated", "code": "PROJECT_EDIT_SUCCESS", "data": { "id": "3f2a…", "name": "Alpha v2" } }
```

**Delete — 200**
```json
{ "success": true, "message": "Project deleted", "code": "PROJECT_DELETE_SUCCESS", "data": { "id": "3f2a…" } }
```

**Validation error — 400**
```json
{
  "success": false,
  "message": "Invalid input",
  "code": "VALIDATION_ERROR",
  "data": { "errors": [ { "path": "data.name", "message": "Required" } ] }
}
```

**Business error — 409**
```json
{ "success": false, "message": "Project has open issues and cannot be closed", "code": "PROJECT_CLOSE_HAS_OPEN_ISSUES" }
```

---

## 5. Response Codes

### 5.1 Code Format
`<MODULE>_<ACTION>_<RESULT>` in UPPER_SNAKE_CASE. Examples: `PROJECT_CREATE_SUCCESS`, `PROJECT_NOT_FOUND`, `MILESTONE_EDIT_LOCKED`.

### 5.2 Global Codes (fixed)

| Code | HTTP | success | When |
|---|---|---|---|
| VALIDATION_ERROR | 400 | false | Zod validation failed |
| UNAUTHENTICATED | 401 | false | Missing / invalid / expired token |
| FORBIDDEN | 403 | false | Authenticated but not allowed |
| NOT_FOUND | 404 | false | Route does not exist |
| CONFLICT | 409 | false | Generic state conflict |
| PAYLOAD_TOO_LARGE | 413 | false | Body or file too large |
| RATE_LIMITED | 429 | false | Too many requests |
| INTERNAL_ERROR | 500 | false | Unexpected failure (details only in logs) |
| SERVICE_UNAVAILABLE | 503 | false | Dependency down / maintenance |

### 5.3 Module Codes

| Code | HTTP | success | Endpoint(s) | When | User message |
|---|---|---|---|---|---|
| [PROJECT_CREATE_SUCCESS] | 201 | true | [POST /project/create] | [Created] | [Project created] |
| [PROJECT_NOT_FOUND] | 404 | false | [get, edit, delete] | [ID not found or soft-deleted] | [Project not found] |

---

## 6. Authentication & Middleware

### 6.1 Auth Flow
[How the frontend obtains a token and how the backend verifies it. Reference ADB 02 §6.1.]

### 6.2 Middleware Chain

| Order | Middleware | File | Applies to | Purpose |
|---|---|---|---|---|
| 1 | Request ID | `middleware/requestId.middleware.ts` | All | Adds `x-request-id` |
| 2 | Logger | `middleware/logger.middleware.ts` | All | Structured access log |
| 3 | Security | `middleware/security.middleware.ts` | All | helmet, CORS |
| 4 | Rate limit | `middleware/rateLimit.middleware.ts` | [All / auth routes] | [Limits] |
| 5 | Auth | `middleware/auth.middleware.ts` | `/api/v1/*` except public | Verify token, set `req.user` |
| 6 | Permission | `middleware/permission.middleware.ts` | Per route | Role / ownership check |
| 7 | Validate | `middleware/validate.middleware.ts` | Per route | Run zod schema |
| last | Error handler | `middleware/error.middleware.ts` | All | Map errors to the envelope |

### 6.3 Permission Map

| Endpoint | [ROLE-01] | [ROLE-02] | [ROLE-03] | Ownership rule |
|---|---|---|---|---|
| [POST /api/v1/project/create] | ✅ | ✅ | ❌ | — |
| [PUT /api/v1/project/:projectId/edit] | ✅ | Own only | ❌ | [`created_by = req.user.id` or project member] |

---

## 7. Route Catalog

> Every endpoint in the system. Grouped by module, in folder order.

### 7.1 Module [CODE] — [Module name]

| # | Method | Path | Action | Description | Feature (BRD) | Roles | Tables (ADB 03) |
|---|---|---|---|---|---|---|---|
| 1 | POST | /api/v1/[module]/create | create | [Create a …] | F-[CODE]-xx | [ROLE-xx] | [table] |
| 2 | POST | /api/v1/[module]/search | search | [Search …] | | | |
| 3 | GET | /api/v1/[module]/:[module]Id/get | get | [Get one …] | | | |
| 4 | PUT | /api/v1/[module]/:[module]Id/edit | edit | [Edit …] | | | |
| 5 | DELETE | /api/v1/[module]/:[module]Id/delete | delete | [Soft delete …] | | | |

#### Submodule [CODE]-[SUB] — [Submodule name]

| # | Method | Path | Action | Description | Feature (BRD) | Roles | Tables (ADB 03) |
|---|---|---|---|---|---|---|---|
| 1 | POST | /api/v1/[module]/:[module]Id/[submodule]/create | create | | | | |

### 7.2 External API (`/api-ext/v1`)

| # | Method | Path | Consumer | Auth | Description |
|---|---|---|---|---|---|
| 1 | [POST] | [/api-ext/v1/…] | [Partner system] | [API key] | [Description] |

---

## 8. Endpoint Details

> Repeat this block for every endpoint in §7.

### EP-[CODE]-[NN]: `[METHOD] [path]`

| Field | Value |
|---|---|
| Description | [What it does] |
| Feature (BRD) | F-[CODE]-xx |
| Workflow (ADB 02) | WF-xx |
| Roles | [ROLE-xx] |
| Ownership rule | [Rule or —] |
| Files | `src/api/[module]/[module].route.ts` → `.handler.ts` → `.service.ts` |
| Tables | [table (read / write)] |
| Idempotent | [Yes / No] |

**Path parameters**

| Name | Type | Rule |
|---|---|---|
| [projectId] | uuid | Must exist and not be soft-deleted |

**Body — `data` fields** *(create / edit)*

| Field | Type | Required (create) | Editable | Rule | Example |
|---|---|---|---|---|---|
| [name] | string | Yes | Yes | [1–200 chars, trimmed] | [Alpha] |

**Search — filter & sort** *(search)*

| Filter key | Type | Matches column | Operator |
|---|---|---|---|
| keyword | string | [name, code] | ILIKE `%value%` |
| [status] | [enum[]] | [status] | IN |

| Sortable fields | Default |
|---|---|
| [createdAt, name] | createdAt desc |

**Zod schema**

```ts
// src/api/[module]/[module].validation.ts
export const [action][Entity]Schema = z.object({
  params: z.object({ [module]Id: z.string().uuid() }),
  body: z.object({
    data: z.object({
      [field]: z.string().trim().min(1).max(200),
    }),
  }),
});
```

**Success response — `data` shape**

```ts
type [Action][Entity]Data = {
  id: string;
  [field]: [type];
};
```

**Response codes**

| Code | HTTP | When |
|---|---|---|
| [MODULE_ACTION_SUCCESS] | [200] | [Success] |
| [MODULE_NOT_FOUND] | 404 | [ID not found] |

**Business rules**

| Rule (BRD) | Behavior |
|---|---|
| BR-[CODE]-xx | [How the service enforces it, which code is returned] |

**Side effects**

| Effect | Detail |
|---|---|
| [Audit log / notification / job] | [Detail] |

---

## 9. Folder Structure (fixed pattern)

### 9.1 Repository Tree

```text
[project]-BE/
├── AGENTS.md                         # Agent single source of truth (CLAUDE.md / GEMINI.md import it)
├── .claude/skills/<skill>/           # Agent playbooks
├── .github/workflows/                # CI / deploy pipelines
├── bruno-API/                        # Manual API test collections, one folder per module
├── migrations/                       # v-<x>-<y>-<z>_<topic>.ts
├── seeds/                            # v-<x>-<y>-<z>_<topic>.ts
├── dev/
│   ├── docs/                         # System docs (English source of truth)
│   ├── report/                       # Append-only change reports
│   ├── plan/                         # Plans for large work
│   ├── db-design/                    # Schema snapshots per DB version
│   └── set-up-environment/           # Local services (Docker compose)
└── src/
    ├── server.ts                     # Starts HTTP server
    ├── app.ts                        # Express app, global middleware, version mounts
    ├── config/                       # Typed env (zod-validated)
    ├── constants/                    # <domain>.constants.ts (incl. response codes)
    ├── types/                        # api.types.ts, db/*.types.ts, express.d.ts
    ├── middleware/                   # <concern>.middleware.ts
    ├── jobs/                         # Background jobs
    ├── utils/
    │   ├── <singleton>.ts            # db, redis, storageClient, logger, swagger
    │   ├── response.ts               # ok(), created(), fail(), searchResult()
    │   └── <domain>/                 # <name>.helper.ts | .service.ts | .schema.ts
    └── api/
        ├── api.routes.ts             # Internal router, mounted at /api/v1
        ├── apiExt.routes.ts          # External router, mounted at /api-ext/v1
        ├── api.docs.ts               # OpenAPI registry list
        └── <module>/                 # URL: /api/v1/<module>/...
            ├── <module>.route.ts
            ├── <module>.handler.ts
            ├── <module>.validation.ts
            ├── <module>.service.ts
            ├── <module>.docs.ts
            ├── <module>.extRoute.ts          # optional — external endpoints
            ├── _<module>.utils.ts            # optional — private helpers
            └── <submodule>/                  # URL: /api/v1/<module>/:<module>Id/<submodule>/...
                ├── <submodule>.route.ts
                ├── <submodule>.handler.ts
                ├── <submodule>.validation.ts
                ├── <submodule>.service.ts
                ├── <submodule>.docs.ts
                └── <subSubmodule>/           # nests the same way
```

### 9.2 File Responsibilities

| File | Responsibility | May import | Must not |
|---|---|---|---|
| `.route.ts` | Declare paths, methods, middleware order; mount child submodule routers | handler, validation, middleware | Contain logic |
| `.handler.ts` | Read `req`, call service, return the envelope via `utils/response` | service, utils/response | Query the DB |
| `.validation.ts` | Zod schemas for params, query, body; export inferred types | zod | Import service |
| `.service.ts` | Business logic, DB access, transactions, rule enforcement | utils/db, other services, constants | Touch `req` / `res` |
| `.docs.ts` | OpenAPI path definitions built from the zod schemas | validation | Contain logic |

### 9.3 Project Module Tree

```text
src/api/
├── [module]/
│   ├── [module].route.ts
│   ├── [module].handler.ts
│   ├── [module].validation.ts
│   ├── [module].service.ts
│   ├── [module].docs.ts
│   └── [submodule]/
│       └── … same 5 files
```

### 9.4 Folder Index

| Folder | Module code | URL prefix | Endpoints | Owner |
|---|---|---|---|---|
| `src/api/[module]/` | [CODE] | `/api/v1/[module]` | [N] | [Name] |

### 9.5 Reference Skeletons

```ts
// src/api/project/project.route.ts
import { Router } from 'express';
import { validate } from '@/middleware/validate.middleware';
import { permit } from '@/middleware/permission.middleware';
import * as h from './project.handler';
import * as v from './project.validation';
import { milestoneRouter } from './milestone/milestone.route';

export const projectRouter = Router();

projectRouter.post('/create', permit('project.create'), validate(v.createProjectSchema), h.createProject);
projectRouter.post('/search', permit('project.search'), validate(v.searchProjectSchema), h.searchProject);
projectRouter.get('/:projectId/get', permit('project.get'), validate(v.getProjectSchema), h.getProject);
projectRouter.put('/:projectId/edit', permit('project.edit'), validate(v.editProjectSchema), h.editProject);
projectRouter.delete('/:projectId/delete', permit('project.delete'), validate(v.deleteProjectSchema), h.deleteProject);

projectRouter.use('/:projectId/milestone', milestoneRouter); // Router({ mergeParams: true })
```

```ts
// src/api/project/project.handler.ts
import type { RequestHandler } from 'express';
import { searchResult } from '@/utils/response';
import * as service from './project.service';

export const searchProject: RequestHandler = async (req, res) => {
  const { rows, total, page, pageSize } = await service.searchProject(req.body, req.user);
  res.status(200).json(searchResult({ code: 'PROJECT_SEARCH_SUCCESS', message: 'Projects retrieved', rows, total, page, pageSize }));
};
```

```ts
// src/utils/response.ts
export const searchResult = <T>(o: { code: string; message: string; rows: T[]; total: number; page: number; pageSize: number }) => ({
  success: true,
  message: o.message,
  code: o.code,
  data: o.rows,
  items: {
    total: o.total,
    pagination: {
      next: o.page * o.pageSize < o.total ? o.page + 1 : null,
      prev: o.page > 1 ? o.page - 1 : null,
    },
  },
});
```

---

## 10. Configuration

| Env variable | Required | Example | Description |
|---|---|---|---|
| NODE_ENV | Yes | `production` | Environment |
| PORT | Yes | `4000` | HTTP port |
| DATABASE_URL | Yes | `postgres://…` | PostgreSQL connection |
| [JWT_SECRET / AUTH_ISSUER] | Yes | | Token verification |
| CORS_ORIGINS | Yes | `https://app.example.com` | Allowed origins |
| [VAR] | | | |

---

## 11. Background Jobs

| Job (ADB 02 JOB-xx) | File | Queue / schedule | Service called |
|---|---|---|---|
| [JOB-01] | `src/jobs/[name].job.ts` | [cron / queue] | [service fn] |

---

## 12. Error Handling & Logging Rules

| Rule | Detail |
|---|---|
| Thrown errors | Services throw `AppError(code, httpStatus, message)`; the error middleware builds the envelope |
| Unknown errors | Logged with `requestId`; client gets `INTERNAL_ERROR` without details |
| Never log | Passwords, tokens, full card numbers, [other PII] |
| Log format | JSON: `timestamp, level, requestId, userId, method, path, status, durationMs` |

---

## 13. Open Questions

| ID | Question | Raised by | Owner | Due | Answer |
|---|---|---|---|---|---|
| Q-01 | [Question] | [Name] | [Name] | [YYYY-MM-DD] | [Pending] |
