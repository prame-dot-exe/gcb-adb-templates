# [Project Name] — ADB 05: Frontend Specification

> **Application Design Book (ADB) — Book 5 of 5.** The brand look, the sitemap, every screen, and how screens use the backend. Folder structure, token rules, API layer and code patterns are in `CONVENTIONS.md` §5.
> Guideline: `guidelines/05-FRONTEND-SPEC.guide.md`. Section tags: **Core** = always fill · **Optional** = fill only if it applies, otherwise leave the section out · **Standard** = follows `CONVENTIONS.md`; leave it out unless the project differs (each difference needs an ADR in ADB 02 §13). Untagged subsections follow their parent's tag. List every section left out in Document Info → *Sections left out*; keep the numbers of the sections you write.
> Keep it short: tables, not prose. Use `TBD — see Q-NN` for unresolved items.

---

## 0. Document Info

> **Core**

| Field               | Value                                       |
| ------------------- | ------------------------------------------- |
| Project name        | [Name]                                      |
| Project code        | [Code]                                      |
| Book                | ADB 05 — Frontend Spec                      |
| Version             | [0.1.0]                                     |
| Status              | [Draft / In Review / Approved / Deprecated] |
| Author(s)           | [Name, role]                                |
| Last updated        | [YYYY-MM-DD]                                |
| Sections left out | [§x Name (not used) · §y Name (standard, no changes)] |
| Repository          | [URL]                                       |
| Figma / design file | [URL / none]                                |

### Change Log

| Version | Date         | Author | Summary of changes |
| ------- | ------------ | ------ | ------------------ |
| 0.1.0   | [YYYY-MM-DD] | [Name] | Initial draft      |

---

## 1. Tech Summary

> **Core.** Fixed stack: `CONVENTIONS.md` §1. Fill only the open choices.

| Item            | Value                                        |
| --------------- | -------------------------------------------- |
| Next.js         | [version]                                    |
| shadcn/ui       | [style], base color [neutral / zinc / slate] |
| Server state    | [TanStack Query / fetch + server actions]    |
| Forms           | [react-hook-form + zod]                      |
| Icons           | [lucide-react]                               |
| i18n            | [next-intl / none] — languages: [en, th, vi] |
| Package manager | [pnpm / npm]                                 |

---

## 2. Folder Structure

> **Standard:** `CONVENTIONS.md` §5.1. Feature folders follow the module tree in BRD §4.1.

| Difference from the standard | Reason | ADR |
| ---------------------------- | ------ | --- |
| [None]                       |        |     |

---

## 3. Design Tokens (shadcn/ui)

> Token rules and `globals.css` pattern: **Standard**, `CONVENTIONS.md` §5.2.

### 3.1 Brand Colors

> **Core.** Only the tokens that differ from the shadcn base color in §1. Everything not listed uses the base theme.

| Token                                  | Light      | Dark       | Usage                       |
| -------------------------------------- | ---------- | ---------- | --------------------------- |
| `--primary` / `--primary-foreground`   | [oklch(…)] | [oklch(…)] | [Brand color, main actions] |
| [`--success` / `--warning` / `--info`] |            |            | [If needed]                 |

### 3.2 Status Colors

> **Optional:** one row per enum value in ADB 03 §3 that users see.

| Enum (ADB 03)    | Value    | Token     | Label    |
| ---------------- | -------- | --------- | -------- |
| [project_status] | [active] | [success] | [Active] |

### 3.3 Typography

> **Core**

| Item                     | Value                                    |
| ------------------------ | ---------------------------------------- |
| Sans font                | [Inter + Noto Sans Thai]                 |
| Mono font                | [JetBrains Mono / none]                  |
| Scale / spacing / radius | [Tailwind and shadcn defaults / changes] |

---

## 4. Sitemap

> **Core.** Every page as a simple tree: page name, then its route. Indent child pages under their parent. Pages shown in the sidebar are marked `[nav]`.

```text
|- Login (/login)
|- Admin (/admin)                                [nav]
|  |- Projects (/admin/project)                  [nav]
|  |  |- Create project (/admin/project/create)
|  |  |- Project detail (/admin/project/:projectId)
|  |  |  |- Edit project (/admin/project/:projectId/edit)
|  |  |  |- Milestones (/admin/project/:projectId/milestone)
|- My work (/me)                                 [nav]
|  |- My tasks (/me/task)                         [nav]
```

---

## 5. Screens

> **Core**

### 5.1 Screen List

| Screen ID | Screen | Route | Shows | Feature (BRD) | Endpoints (ADB 04) |
| --- | --- | --- | --- | --- | --- |
| SC-01 | [Project list] | /[scope]/[module] | [Columns: name, owner, status, due date · Filters: status, owner] | F-[CODE]-xx | [EP-CODE-02] |

### 5.2 Screen Detail

> Only for screens that are more than a standard list, form, or detail page (`CONVENTIONS.md` §5.1, §5.4): multi-step flows, special behavior, or forms whose fields differ from the endpoint. **Every dashboard and report needs a block** with its metrics.

#### SC-[NN]: [Screen name]

[One line: what the user does here.]

| Element        | Behavior                                                                   |
| -------------- | -------------------------------------------------------------------------- |
| [Close button] | [Shown only when all issues are closed; confirms before calling EP-PRJ-06] |

**Metrics** _(dashboards and reports)_

| Metric | How it's calculated | Endpoint |
| --- | --- | --- |
| [Overdue tasks] | [Open tasks with due date before today, my projects only] | [EP-DSH-01] |

---

## 6. Navigation

> **Standard:** the sidebar shows the `[nav]` pages of the sitemap in tree order; pages a role can't use (BRD §4.4) are hidden. List only exceptions.

| Exception | Detail |
| --------- | ------ |
| [None]    |        |

---

## 7. Components Inventory

> **Optional:** custom components beyond shadcn/ui that the project needs.

| Component    | Folder                             | Built on | Used by |
| ------------ | ---------------------------------- | -------- | ------- |
| [statusChip] | `components/primitive/statusChip/` | [badge]  | [Lists] |

---

## 8. API Layer & Error Handling

> **Standard:** `CONVENTIONS.md` §5.3. List only module codes that need special UI behavior.

| Code (ADB 04 §5.1)              | UI behavior                    |
| ------------------------------- | ------------------------------ |
| [PROJECT_CLOSE_HAS_OPEN_ISSUES] | [Show the list of open issues] |

---

## 9. State, Forms & Auth

> **Standard:** `CONVENTIONS.md` §5.5.

## 10. Conventions & ESLint

> **Standard:** `CONVENTIONS.md` §5.6.

| Difference from §9–10 standards | Reason | ADR |
| ------------------------------- | ------ | --- |
| [None]                          |        |     |

---

## 11. Accessibility & Localization

> **Core.** Accessibility baseline: `CONVENTIONS.md` §5.7.

| Item                          | Value                        |
| ----------------------------- | ---------------------------- |
| Languages                     | [en / th / vi]               |
| Date, number, currency format | [Locale, timezone, currency] |
| Extra accessibility needs     | [None]                       |

---

## 12. Open Questions

> **Core**

| ID   | Question   | Owner  | Due          | Answer    |
| ---- | ---------- | ------ | ------------ | --------- |
| Q-01 | [Question] | [Name] | [YYYY-MM-DD] | [Pending] |
