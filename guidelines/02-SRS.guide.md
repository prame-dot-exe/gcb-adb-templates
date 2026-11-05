# Guideline — ADB 02: Software Requirements Specification (SRS)

Explains every section of `templates/02-SRS.md`. Section numbers match the template.

**Purpose of this book:** the technical view of the whole system. It turns the BRD's business answers into measurable targets, an architecture, workflows, security, infrastructure and recovery targets. Everything that is the same in every project (pipeline, monitoring, backups, security baseline, testing) lives in `CONVENTIONS.md` §6; this book only records what is specific.

**Rules**

- **Trace back to the BRD.** NFRs and recovery targets cite the BRD answer they come from; workflows cite features. If a number can't be traced, record it as an assumption.
- **System level, not endpoint level.** Field-level detail belongs in ADB 03–05.
- **Standard sections** list only differences from `CONVENTIONS.md`, each with an ADR in §13. Write `None` when there are none.
- **Optional sections** get one line `N/A — reason` when they don't apply.

---

## 0. Document Info · Core

Same fields as the BRD, plus **Based on BRD version**. When the BRD changes, review the SRS.

---

## 1. Non-Functional Requirements · Core

`NFR-NN`. Only categories that matter for this project (performance, availability, security, compatibility, retention, localization…). Each target is a number or a checkable condition.

| BRD answer                                             | NFR                                                   |
| ------------------------------------------------------ | ----------------------------------------------------- |
| "Down for an hour = meetings without data, not costly" | Availability 99.5% monthly; single region is fine     |
| "Down for an hour = we lose sales"                     | Availability 99.9%+; multi-zone, health-checked       |
| "Users in the field with bad signal"                   | Works on slow mobile networks; consider offline cache |
| "Must keep records 5 years for audit"                  | Retention 5 years                                     |
| "Thai and English users"                               | th/en UI, Asia/Bangkok display time, UTC storage      |

### 1.1 Capacity Estimate

Show the arithmetic so reviewers can check it:

`peak RPS ≈ peak concurrent users × actions per user per minute ÷ 60 × requests per action`

Example: 120 × 4 ÷ 60 × 3 ≈ 24 req/s → design target 100 req/s (×4 headroom). Data growth: records/day × row size × 365 × 1.5 for indexes.

---

## 2. Architecture · Core

| Section                   | What to write                                                                                                                          |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| 2.1 Architecture Style    | Default is a modular monolith. Anything more distributed needs a reason and an ADR                                                     |
| 2.2 System Context        | One diagram a business person can follow: users, the app, outside systems                                                              |
| 2.3 Components · Optional | Only when there is more than frontend, backend and database (jobs, cache, storage, queue)                                              |
| 2.4 Tech Stack            | Only the open choices (`CONVENTIONS.md` §1), each with a one-line reason. Take them from the prototype's `package.json` when available |

---

## 3. Workflows · Core

### 3.1 Workflow Catalog

`WF-NN`. Only the main or hard flows that a developer couldn't work out from the backend spec: multi-step processes, approvals, money, outside services, unusual login. Usually 2–5 per project. Plain create/search/get/edit/delete and simple one-step rules need no workflow. The "Why it needs a diagram" column says in one line what makes it non-obvious; if you can't fill it, drop the workflow.

### 3.2 Sequence Diagrams

One per workflow. Write messages as plain actions ("Close project", "Check open issues"), not endpoint paths, module codes or response codes; those are mapped during implementation. Show the main failure path and retries for outside services; validation errors are standard and left out.

### 3.3 State Diagrams · Optional

One per item with statuses (BRD §4.2). States use the **exact enum values** from ADB 03 (lowercase snake_case). The table lists the action for each transition (plain words, not an endpoint) and the guarding rule; who may do it comes from BRD §4.4. Transitions not listed are rejected by the API.

### 3.4 Background Jobs · Optional

`JOB-NN`: schedule (cron or event), what it does, and what happens on failure (retries, alert).

---

## 4. Data Flow · Optional

`DF-NN`. Use when data comes from imports or other systems, or feeds reports and exports. Mark every flow carrying personal data; those drive ADB 03 §7.

## 5. Integrations · Optional

`INT-NN`. One row per outside service: purpose, direction (outbound = we call; inbound = they call `/api-ext/v1`), auth, and what happens when it is down (from the BRD answer).

---

## 6. Security · Core

| Section                                          | What to write                                                                                                        |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| 6.1 Authentication                               | Login method, session/token lifetime, MFA. Example: Google SSO via NextAuth, 15-min access token, MFA through Google |
| 6.2 Authorization & Security Measures · Standard | `CONVENTIONS.md` §6.1–6.2. List only differences (e.g. encrypted columns, IP allow-list)                             |
| 6.3 Compliance Implementation · Optional         | Each BRD §7 item → the concrete measure (table, endpoint, job, setting)                                              |

---

## 7. Infrastructure · Core

| Section                           | What to write                                                                                                               |
| --------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| 7.1 Inventory                     | Hosting for frontend, backend, database, other services; size; region; estimated monthly cost (compare with the BRD budget) |
| 7.2 Environments                  | URLs and data type for Dev, Staging, Production. Never put real personal data outside production unless anonymized          |
| 7.3 Deployment Diagram · Optional | Only when the setup is more than one frontend, backend and database                                                         |

---

## 8. CI/CD Pipeline · Standard

`CONVENTIONS.md` §6.3. List only differences.

## 9. Observability · Standard

`CONVENTIONS.md` §6.4. Fill who receives alerts; an alert with no owner is not noticed.

---

## 10. Backup & Disaster Recovery · Core

| Term | Meaning                      | Comes from                     |
| ---- | ---------------------------- | ------------------------------ |
| RPO  | Maximum acceptable data loss | BRD 3.4 "acceptable data loss" |
| RTO  | Maximum acceptable downtime  | BRD 3.4 "impact of downtime"   |

**10.2 Backup Plan & Scenarios · Standard:** the standard backup (`CONVENTIONS.md` §6.5) gives roughly RPO ≤ 5 min and RTO of a few hours. List changes only when the targets need more (e.g. multi-region for RTO < 1 hour).

---

## 11. Testing Strategy · Standard

`CONVENTIONS.md` §6.6. Name the critical flows that get end-to-end tests: the ones that lose money or trust if broken.

## 12. Traceability Matrix · Optional

For large projects or audits. Normally the feature column in ADB 04 §7 and ADB 05 §5.1 already gives the links.

## 13. ADRs · Optional

One per decision that differs from `CONVENTIONS.md` or isn't obvious: context, decision, alternatives, consequences. Never edit an accepted ADR's decision; add a new one that supersedes it.

## 14. Open Questions · Core

Global numbering shared with all books (see BRD guideline §10).
