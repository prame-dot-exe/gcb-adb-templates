# Guideline — ADB 05: Frontend Specification

Explains every section of `templates/05-FRONTEND-SPEC.md`. Section numbers match the template.

**Purpose of this book:** the brand look, every route and screen, and how screens use the backend. Folder structure, token rules, API layer, default screen states, forms, auth and lint rules are in `CONVENTIONS.md` §5 and are not repeated here.

**Rules**

- **Standard screens need no detail.** A list, create form, edit form or detail page that follows the conventions is fully described by its row in the screen list.
- **Tokens, never raw colors.** List only the tokens that differ from the shadcn base theme.
- **Every screen traces** to a BRD feature, a page in the sitemap, and the endpoints it calls (ADB 04 EP IDs).
- **Standard sections** list only differences from `CONVENTIONS.md`, each with an ADR. Optional sections that don't apply, and Standard sections with no differences, are left out and listed in Document Info → *Sections left out*.

---

## 0. Document Info · Core

Standard fields plus repository and design file link.

## 1. Tech Summary · Core

Only the open choices: Next.js version, shadcn style and base color, server-state library, forms, icons, i18n and languages, package manager. Take them from the prototype's `package.json` when available.

## 2. Folder Structure · Standard

`CONVENTIONS.md` §5.1. Feature folders follow the BRD module tree.

---

## 3. Design Tokens · Core

Token rules and `globals.css` pattern are standard (`CONVENTIONS.md` §5.2).

| Section                      | What to write                                                                                                                                                                                                                                          |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 3.1 Brand Colors             | Start from the base color in §1; list only overridden tokens (usually `primary`, sometimes `ring`, `chart-*`, `sidebar-primary`) with light and dark `oklch()` values. Add `success` / `warning` / `info` only if used. Check contrast in both modes   |
| 3.2 Status Colors · Optional | Every enum value from ADB 03 §3 that users see → token and label. Unlisted values show as a gray chip                                                                                                                                                  |
| 3.3 Typography               | Fonts (loaded with `next/font`). For Thai or Vietnamese, pair a Latin font with one that covers the script (e.g. Inter + Noto Sans Thai) and allow a slightly taller line height. Spacing and radius stay on defaults unless the brand needs otherwise |

---

## 4. Sitemap · Core

One plain-text tree of every page, written with `|-` and indented with `|  ` per level:

```text
|- Projects (/admin/project)                [nav]
|  |- Project detail (/admin/project/:projectId)
|  |  |- Edit project (/admin/project/:projectId/edit)
```

- Each line: page name in plain words, then the route in brackets.
- Child pages sit under the page you reach them from. Route params are named after the module (`:projectId`), matching the backend.
- Standard pages per module: list, create, detail, edit (`CONVENTIONS.md` §5.1). Leave out pages the module doesn't have.
- Mark sidebar pages with `[nav]`. Who can open each page comes from BRD §4.4, so the tree has no roles.

---

## 5. Screens · Core

### 5.1 Screen List

One row per page in the sitemap: ID, name, route, **what it shows**, BRD feature, and the EP IDs it calls. *Shows* is one short line: the columns and filters of a list, the main sections of a detail page, or the fields of a form if they aren't simply the table's *Set by: User* columns.

### 5.2 Screen Detail

**Every dashboard and report** gets a block with a *Metrics* table: each number, how it's calculated in plain words (what's counted, which filter, whose data), and the endpoint that provides it. Other screens only when more than standard: multi-step flows, conditional buttons, forms whose fields differ from the endpoint, or non-default loading/empty/error behavior (defaults: `CONVENTIONS.md` §5.4). One line of purpose, then the special elements and their behavior.

---

## 6. Navigation · Standard

The sidebar shows the `[nav]` pages of the sitemap, hidden by role per BRD §4.4. List only exceptions (e.g. a top bar item, an external link).

## 7. Components Inventory · Optional

Custom components beyond shadcn/ui: folder, what it's built on, who uses it.

## 8. API Layer & Error Handling · Standard

`CONVENTIONS.md` §5.3. List only module codes that need special UI behavior (redirect, confirmation, showing related data). Everything else shows `message` in a toast.

## 9–10. State, Forms, Auth, Conventions & ESLint · Standard

`CONVENTIONS.md` §5.5–5.6. List differences only.

## 11. Accessibility & Localization · Core

Languages, date/number/currency format and timezone. The accessibility baseline is standard (`CONVENTIONS.md` §5.7); add only extra needs.

## 12. Open Questions · Core

Global numbering shared with all books.
