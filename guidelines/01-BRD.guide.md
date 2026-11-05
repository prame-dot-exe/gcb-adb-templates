# Guideline — ADB 01: Business Requirements Document (BRD)

Explains every section of `templates/01-BRD.md`. Section numbers match the template.

**Purpose of this book:** describe the business: why the app exists, who uses it, what each module does, and what is not included. A non-technical stakeholder must be able to read and approve it.

**Rules**
- **No technology.** No frameworks, tables, endpoints, or response codes. If you're writing `uuid` or `POST`, it belongs in ADB 02–05.
- **Every feature is testable.** Each feature has acceptance criteria in Given / When / Then form.
- **The BRD is upstream.** Other books reference BRD IDs (`F-PRJ-01`, `BR-PRJ-02`, `ROLE-01`). Never renumber an approved ID; retire it and add a new one.
- **Write `N/A — reason`** instead of deleting a section; write `TBD — see Q-xx` for unknowns.

---

## 0. Document Info

| Field | Meaning | Example |
|---|---|---|
| Project name | Official name | `NexWork` |
| Project code | Short code used in IDs and repo names | `NXW` |
| Book | Fixed label for this book | `ADB 01 — BRD` |
| Version | Version of this document (semver) | `0.3.0` |
| Status | Draft → In Review → Approved → Deprecated | `In Review` |
| Author / Reviewer / Approver | Who wrote, checked, and signs off | `PO — K. Supawit` |
| Created / Last updated | ISO dates | `2026-11-05` |
| Related books | Links to ADB 02–05 | repo links |
| Change Log | One row per version | `0.2.0 — added Budget module` |

---

## 1. Overview

| Section | What to write | Example | Common mistake |
|---|---|---|---|
| 1.1 Background | How work happens today, tools used, volume | "Project managers track 40 projects in Excel files shared by email." | Jumping to the solution |
| 1.2 Problem Statement | The specific pains and who feels them | "Status is out of date; managers spend 4 h/week compiling reports." | Vague "inefficient process" |
| 1.3 Purpose | 1–3 sentences: why this app exists | "Give one live view of every project's status, budget and risk." | Listing features |
| 1.5 Description | 2–5 sentence elevator pitch | "NexWork is a web app for PMOs to manage projects, milestones, budgets and risks…" | Too long |

### 1.4 Objectives & Success Metrics

| Field | Meaning | Example |
|---|---|---|
| ID | `OBJ-NN` | `OBJ-01` |
| Objective | Business goal | Reduce reporting effort |
| Success metric | How it's measured | Hours/week spent compiling reports |
| Baseline | Today's value (makes the target meaningful) | 4 h |
| Target | Value that means success | < 0.5 h |
| By when | Date | 2027-06-30 |

---

## 2. Stakeholders & Users

### 2.1 Stakeholders
| Field | Meaning | Example |
|---|---|---|
| Stakeholder | Person or group | Finance team |
| Role in project | Sponsor / owner / user / operator | User |
| Interest / concern | What they care about | Accurate budget numbers |
| Decision power | Approve / consult / inform (RACI-style) | Consult |
| Contact | Email or channel | finance@… |

### 2.2 User Personas
| Field | Meaning | Example |
|---|---|---|
| Persona ID | `PER-NN` | `PER-02` |
| Persona | Name + role | "Mai — project manager" |
| Description | Background, tech comfort | 8 years PM, comfortable with web apps |
| Goals | What they want | Update status in under 2 minutes |
| Pain points | Frustrations today | Chasing team members for updates |
| Device & context | Where and how they use it | Laptop in office, phone between meetings |

**Tip:** device & context drives responsive design and offline needs in ADB 02/05.

### 2.3 User Roles (business view)
| Field | Meaning | Example |
|---|---|---|
| Role ID | `ROLE-NN`, reused by permission tables in ADB 02/04/05 | `ROLE-02` |
| Role | Permission group name | Project Manager |
| Who holds it | Job titles | PMs, team leads |
| Responsible for | What they own | Their projects' data |
| Est. number of users | Rough count | 30 |

**Persona vs role:** a persona describes behavior; a role describes permissions. One persona can hold several roles.

---

## 3. Main Application

| Section | What to write | Example |
|---|---|---|
| 3.1 Application Concept | The end-to-end story in plain language | "A PM creates a project, adds milestones and tasks, the team updates progress, and executives see the portfolio dashboard." |
| 3.2 Application Scopes | Top-level areas of the app. These become route groups in ADB 05 (`/admin`, `/me`) | `admin` — PMO setup; `me` — my projects and tasks |
| 3.3 As-Is process | Today's flow as a diagram + pain points | Excel → email → manual merge |
| 3.4 To-Be process | Future flow inside the app + improvements | Update in app → dashboard updates live |

### 3.5 Usage & Volume Expectations
These answers feed the NFRs, capacity estimate, and DR targets in ADB 02. Ask them in business language; never ask "requests per second."

| Question | Why it matters | Example answer |
|---|---|---|
| Users at launch / after 1 year | Sizing and cost | 50 / 300 |
| Active at the busiest moment | Peak load | ~40% at Monday 9 am stand-up |
| Busiest times / peaks | Scaling and maintenance windows | Month-end reporting |
| Key records created per day | Database growth | ~200 task updates/day |
| How far back history is needed | Retention, archiving | 5 years (audit) |
| Impact of 1 hour downtime at peak | Availability target | "Meetings run without data — annoying but not costly" |
| Acceptable data loss | Backup frequency (RPO) | "Losing an hour of updates is OK; a day is not" |
| Where users are | Hosting region, timezone, compliance | Thailand only |

---

## 4. Modules & Work

### 4.1 Module Map
| Field | Meaning | Example |
|---|---|---|
| Module code | 2–4 letter UPPERCASE code used in every ID across the ADB | `PRJ` |
| Module | Name | Project |
| Purpose | Business need | Manage project master data |
| Scope(s) | Which app areas show it | admin, me |
| Primary roles | Main users | ROLE-01, ROLE-02 |
| Priority | Must / Should / Could | Must |

**Naming rule:** the module name (camelCase, singular) becomes the folder and URL segment in ADB 04/05: `Project` → `project`, `Milestone Task` → `milestoneTask`. Decide names here and keep them.

### 4.2 Module Detail
One block per module; submodules get their own block (e.g. `MOD-PRJ-MLS: Milestone` under Project). A submodule is something that **only exists inside a parent record** (a milestone belongs to one project). If it can exist on its own, it's a module.

**Work (features)**
| Field | Meaning | Example |
|---|---|---|
| Feature ID | `F-<CODE>-NN` | `F-PRJ-03` |
| Feature | Short verb phrase | Close a project |
| Description | What the user can do | PM marks a project closed when all milestones are done |
| Actor | Role | ROLE-02 |
| Priority | MoSCoW | Must |
| Acceptance criteria | Given / When / Then | Given all milestones are done, when the PM clicks Close, then status becomes Closed and the project is read-only |

**Tip:** think in the standard actions (create, search, view, edit, delete) plus special actions (approve, close, export). Each standard action usually becomes one feature row.

**Business rules** — `BR-<CODE>-NN`. Policy in plain language: "A closed project cannot be edited except by Admin." These become validation and service logic.

**Statuses / lifecycle** — every status the business uses, who can move a record there, and the next possible statuses. ADB 02 turns this into a state diagram and ADB 03 into an enum, so the names must match.

**Notifications & reports** — anything the system sends or produces: alerts, emails, scheduled reports, exports.

### 4.3 Cross-Module Rules
Rules involving more than one module, with ID `BR-X-NN`. Example: "Deleting a project archives its milestones, tasks and budgets."

### 4.4 Permissions (business view)
Feature × role grid. Values: ✅, ❌, or a condition ("Own only", "Members of the project"). ADB 04 turns this into the endpoint permission map.

---

## 5. Scope

| Section | Meaning | Example |
|---|---|---|
| 5.1 In Scope | Committed for this release, linked to modules/features | IS-03 Budget tracking — F-BGT-01…05 |
| 5.2 Out of Scope | Things people might expect but won't get, with reason and phase | OS-02 Mobile app — Phase 2 |
| 5.3 Release Phases | Groups of features by delivery date | MVP: PRJ, MLS, ISS — 2027-02-01 |

**Tip:** the out-of-scope list prevents the most arguments. Be generous.

---

## 6. Limitations, Assumptions & Constraints

| Section | Meaning | Example |
|---|---|---|
| 6.1 Known Limitations (`LIM-NN`) | Things the release deliberately can't do, with impact and workaround | LIM-01: No offline mode — users on site must wait for signal |
| 6.2 Assumptions (`A-NN`) | Believed true but unverified; what happens if wrong; who validates | A-02: All users have company Google accounts — else we need email login |
| 6.3 Constraints (`C-NN`) | Fixed limits: budget, time, legal, organization, mandated technology | C-01: Go-live before 2027-03-01 |
| 6.4 Dependencies (`D-NN`) | Things outside the team that must happen | D-01: IT provides SSO configuration |

**Assumption vs limitation:** an assumption might be wrong; a limitation is a known choice.

---

## 7. Compliance & Policy
Each law or internal policy, why it applies, and the requirement in business words. Example: "PDPA — we store employee names and phone numbers — employees can see and correct their data; data deleted 1 year after leaving." ADB 02 §6.4 maps these to technical measures.

---

## 8. Risks
Business and delivery risks (`R-NN`) with likelihood, impact, mitigation, owner. Example: "R-03: PMs keep using Excel — High/High — import tool + training + make the dashboard the official report."

## 9. Glossary
Every domain term and abbreviation. Add one whenever two people use different words for the same thing.

## 10. Open Questions
IDs are **global across all five books** (one sequence: Q-01, Q-02…), so a `TBD — see Q-07` in ADB 04 points to one place. Keep answered rows as a record.

## 11. Sign-off
Who approved, their decision, and the date. Developers build only from an approved BRD.
