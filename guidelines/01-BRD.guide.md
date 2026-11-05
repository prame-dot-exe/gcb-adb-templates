# Guideline — ADB 01: Business Requirements Document (BRD)

Explains every section of `templates/01-BRD.md`. Section numbers match the template.

**Purpose of this book:** describe the business: why the app exists, who uses it, what each module does, and what is not included. A non-technical stakeholder must be able to read and approve it.

**Rules**

- **No technology.** No frameworks, tables, endpoints, or response codes. If you're writing `uuid` or `POST`, it belongs in ADB 02–05.
- **Keep it short.** Tables and bullets, one idea per line. A small app gets a short BRD.
- **Core sections are always filled; Optional sections** are left out when they don't apply and listed in Document Info → *Sections left out*.
- **Every feature is testable.** Each feature has acceptance criteria in Given / When / Then form.
- **The BRD is upstream.** Other books reference BRD IDs (`F-PRJ-01`, `BR-PRJ-02`, `ROLE-01`). Never renumber an approved ID; retire it and add a new one.

---

## 0. Document Info · Core

| Field        | Meaning                                   | Example                       |
| ------------ | ----------------------------------------- | ----------------------------- |
| Project code | Short code used in IDs and repo names     | `NXW`                         |
| Version      | Semver of this document                   | `0.3.0`                       |
| Status       | Draft → In Review → Approved → Deprecated | `In Review`                   |
| Change Log   | One row per version                       | `0.2.0 — added Budget module` |

---

## 1. Overview · Core

| Section                  | What to write                                                     | Example                                                                            | Common mistake                                       |
| ------------------------ | ----------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ---------------------------------------------------- |
| 1.1 Background & Problem | How work happens today and what goes wrong, with numbers if known | "PMs track 40 projects in Excel shared by email; compiling status takes 4 h/week." | Jumping to the solution; vague "inefficient process" |
| 1.2 Purpose              | 1–3 sentences: why this app exists                                | "Give one live view of every project's status, budget and risk."                   | Listing features                                     |

### 1.3 Objectives & Success Metrics

`OBJ-NN`. Each objective has a measurable metric, today's value (makes the target meaningful), a target, and a date. Example: Reduce reporting effort — hours/week compiling reports — 4 h → < 0.5 h by 2027-06-30. One to three objectives is normal.

---

## 2. Users · Core

### 2.1 User Roles

`ROLE-NN`, reused by every other book. One row per group of people with the same permissions.

| Field            | Example                                  |
| ---------------- | ---------------------------------------- |
| Role             | Project Manager                          |
| Who holds it     | PMs, team leads                          |
| Responsible for  | Their projects' data                     |
| Est. users       | 30                                       |
| Device & context | Laptop in office, phone between meetings |

Device & context drives mobile layout and offline needs in ADB 05.

### 2.2 Stakeholders & Personas · Optional

Use when several departments are involved or users behave very differently. `PER-NN` for personas. Decision power: approve / consult / inform.

---

## 3. Main Application · Core

| Section                                       | What to write                                                                    | Example                                                                                             |
| --------------------------------------------- | -------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
| 3.1 Application Concept                       | The end-to-end story in one short paragraph                                      | "A PM creates a project, adds milestones, the team updates progress, executives see the dashboard." |
| 3.2 Application Scopes                        | Top-level areas of the app. They become route groups in ADB 05 (`/admin`, `/me`) | `admin` — PMO setup; `me` — my projects                                                             |
| 3.3 Process Today vs. With the App · Optional | Step-by-step comparison, only when replacing an existing process                 | Excel → email → merge vs. update in app → live dashboard                                            |

### 3.4 Usage & Volume Expectations

These answers feed the NFRs, capacity estimate and recovery targets in ADB 02. Ask them in everyday words; never ask "requests per second."

| Question                               | Drives                   | Example answer                                       |
| -------------------------------------- | ------------------------ | ---------------------------------------------------- |
| Users at launch / after 1 year         | Sizing, cost             | 50 / 300                                             |
| Active at the busiest moment, and when | Peak load                | ~40% at Monday 9 am                                  |
| Key records created per day            | Database growth          | ~200 task updates/day                                |
| How far back history is needed         | Retention                | 5 years (audit)                                      |
| Impact of 1 hour down at peak          | Availability             | "Meetings run without data, annoying but not costly" |
| Acceptable data loss                   | Backup frequency         | "Losing an hour is OK; a day is not"                 |
| Where users are                        | Hosting region, timezone | Thailand only                                        |

---

## 4. Modules & Work · Core

### 4.1 Module Map

| Field       | Meaning                                     | Example   |
| ----------- | ------------------------------------------- | --------- |
| Module code | 2–4 letter UPPERCASE code used in every ID  | `PRJ`     |
| Module      | Name                                        | Project   |
| Parent      | Parent module code for submodules, else `—` | `—`       |
| Scope(s)    | Which app areas show it                     | admin, me |
| Priority    | Must / Should / Could                       | Must      |

**Naming rule:** the module name becomes the folder and URL segment in ADB 04/05: `Milestone Task` → `milestoneTask`. Decide names here and keep them. A **submodule** only exists inside a parent record (a milestone belongs to one project); if it can exist on its own, it's a module.

### 4.2 Module Detail

One block per module and submodule.

- **Features** — `F-<CODE>-NN`. Think in the standard actions (create, search, view, edit, delete) plus special ones (approve, close, export). Acceptance criteria: "Given all milestones are done, when the PM clicks Close, then the project becomes Closed and read-only."
- **Business rules** — `BR-<CODE>-NN`, policy in plain language: "A closed project cannot be edited except by Admin."
- **Statuses** — only if the main item goes through steps. These names become the enum in ADB 03 and the state diagram in ADB 02, so keep them stable.
- **Notifications & reports** — only if the app sends or produces anything: alerts, emails, scheduled reports, exports.

### 4.3 Cross-Module Rules · Optional

Rules involving more than one module, ID `BR-X-NN`.

### 4.4 Permissions

Feature × role grid with ✅, ❌, or a condition ("Own only", "Project members"). This is the **only** permission table in the ADB; ADB 04 adds how conditions are checked and ADB 05 hides pages and actions by it.

---

## 5. Scope · Core

| Section                       | Meaning                                                                                                             | Example                             |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| 5.1 In Scope                  | Committed for this release                                                                                          | IS-03 Budget tracking — F-BGT-01…05 |
| 5.2 Out of Scope              | Things people might expect but won't get, with reason and phase. Be generous: this list prevents the most arguments | OS-02 Mobile app — Phase 2          |
| 5.3 Release Phases · Optional | Feature groups by delivery date                                                                                     | MVP: PRJ, MLS — 2027-02-01          |

---

## 6. Limitations, Assumptions & Constraints · Core

| Section                                     | Meaning                                                                                                        | Example                                                             |
| ------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------- |
| 6.1 Known Limitations · Optional (`LIM-NN`) | Things this release deliberately can't do                                                                      | LIM-01: No offline mode                                             |
| 6.2 Assumptions (`A-NN`)                    | Believed true but unverified; impact if wrong; who confirms. Every `(Assumption A-NN)` in any book points here | A-02: All users have company Google accounts                        |
| 6.3 Constraints & Dependencies (`C-NN`)     | Fixed limits (budget, time, legal, mandated technology) and things outside the team that must happen           | C-01: Go-live before 2027-03-01; C-02 (dependency): IT provides SSO |

An assumption might be wrong; a limitation is a known choice.

---

## 7. Compliance & Policy · Optional

Each law or policy, why it applies, and the requirement in business words. Example: "PDPA — we store employee phone numbers — employees can correct their data; data deleted 1 year after leaving." ADB 02 §6.3 maps these to technical measures.

## 8. Risks · Core

`R-NN` business and delivery risks with likelihood, impact, mitigation, owner. Three to five is normal.

## 9. Glossary · Optional

Domain terms and abbreviations outsiders won't know.

## 10. Open Questions · Core

IDs are **global across all five books** (one sequence: Q-01, Q-02…), so `TBD — see Q-07` in any book points to one place. Keep answered rows as a record.

## 11. Sign-off · Optional

Who approved, their decision, and the date, when formal approval is required.
