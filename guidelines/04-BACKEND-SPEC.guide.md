# Guideline — ADB 04: Backend Specification

Explains every section of `templates/04-BACKEND-SPEC.md`. Section numbers match the template.

**Purpose of this book:** the list of every endpoint and the rules specific to this project. The API shape, request/response contract, folder structure, file roles, middleware and error handling are in `CONVENTIONS.md` §3–4 and are not repeated here.

**Rules**

- **One home per fact.** Endpoints are listed once (route catalog §7). Response codes once (§5.1). Who can do what is in BRD §4.4; this book adds only the ownership checks (§6.3).
- **Detail only where needed.** A plain get/delete or a create/edit whose fields are just the table columns needs only its catalog row.
- **Every endpoint traces to a BRD feature** and lists the tables it touches.
- **Validation mirrors the database:** `max(200)` ↔ `varchar(200)`.
- **Standard sections** list only differences from `CONVENTIONS.md`, each with an ADR. **Optional sections** get one line `N/A — reason` when they don't apply.

---

## 0. Document Info · Core

Standard fields plus API version (`v1`) and repository.

## 1. Tech Summary · Core

Only the open choices: Node version, DB access library, how tokens are verified, package manager.

---

## 2–4. API Conventions, Request Body, Response Structure · Standard

`CONVENTIONS.md` §3.1–3.4. Fill the search page size limit if not 100. List differences, if any, in the single table under §4.

## 5. Response Codes

Format and global codes are standard (`CONVENTIONS.md` §3.5).

### 5.1 Module Codes · Core

Every module code once: HTTP status, when it happens, and the user message (the frontend can show it as-is). Include each module's `_SUCCESS` codes, `_NOT_FOUND`, and one code per business rule that can fail (`PROJECT_CLOSE_HAS_OPEN_ISSUES`). Codes are permanent.

---

## 6. Authentication & Permissions · Core

| Section                   | What to write                                                                                              |
| ------------------------- | ---------------------------------------------------------------------------------------------------------- |
| 6.1 Auth Flow             | One or two lines; details in ADB 02 §6.1                                                                   |
| 6.2 Middleware · Standard | Only middleware beyond `CONVENTIONS.md` §4.3                                                               |
| 6.3 Ownership Rules       | Every "own only" or conditional cell in BRD §4.4: which endpoints, the rule, and how the service checks it |

---

## 7. Route Catalog · Core

Every endpoint, grouped by module then submodule, in folder order. The **EP ID** (`EP-<CODE>-NN`) is assigned here.

| Field         | Example                                                           |
| ------------- | ----------------------------------------------------------------- |
| Method / Path | `PUT /api/v1/project/:projectId/close`                            |
| Description   | Close a project (extra action: a status change, not a field edit) |
| Feature (BRD) | F-PRJ-03                                                          |
| Roles         | ROLE-01, ROLE-02 (from BRD §4.4)                                  |
| Tables        | projects, issues (read)                                           |

Extra actions need a reason in the description. **7.2 External API · Optional:** endpoints under `/api-ext/v1` for partners and webhooks, with their auth (API key / HMAC).

---

## 8. Endpoint Details · Core (selective)

Write a block only for:

- every **search** endpoint: filter keys → columns and operators, sortable fields, default sort (these drive the indexes in ADB 03);
- every **extra action**: body fields and rules;
- every **create/edit** with business rules or fields that differ from the table columns.

Each block lists body fields with rules, business rules with the code returned (from §5.1), and side effects (notifications, audit, jobs). Zod schemas and response types are written in code, not here.

---

## 9. Folder Structure · Standard

`CONVENTIONS.md` §4.1–4.2. Folders follow the module tree, so no project tree is needed.

## 10. Configuration · Core

Only project-specific environment variables (outside services, feature settings). Examples must never be real secrets.

## 11. Background Jobs · Optional

Each ADB 02 `JOB-NN` → its file and the service function it calls. Jobs don't contain business logic.

## 12. Error Handling & Logging · Standard

`CONVENTIONS.md` §4.4. Add any project data that must never be logged.

## 13. Open Questions · Core

Global numbering shared with all books.
