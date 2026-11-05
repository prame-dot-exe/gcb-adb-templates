# [Project Name] — ADB 01: Business Requirements Document (BRD)

> **Application Design Book (ADB) — Book 1 of 5.** Business view only: why, who, what, and what not. No technology here.
> Guideline: `guidelines/01-BRD.guide.md`. Section tags: **Core** = always fill · **Optional** = fill only if it applies, otherwise leave the section out · **Standard** = follows `CONVENTIONS.md`; leave it out unless the project differs (each difference needs an ADR in ADB 02 §13). Untagged subsections follow their parent's tag. List every section left out in Document Info → *Sections left out*; keep the numbers of the sections you write.
> Keep it short: tables and bullets, plain words. Use `TBD — see Q-NN` for unresolved items.

---

## 0. Document Info

> **Core**

| Field        | Value                                       |
| ------------ | ------------------------------------------- |
| Project name | [Name]                                      |
| Project code | [Short code, e.g. NXW]                      |
| Book         | ADB 01 — BRD                                |
| Version      | [0.1.0]                                     |
| Status       | [Draft / In Review / Approved / Deprecated] |
| Author(s)    | [Name, role]                                |
| Approver(s)  | [Name, role]                                |
| Last updated | [YYYY-MM-DD]                                |
| Sections left out | [§x Name (not used) · §y Name (standard, no changes)] |

### Change Log

| Version | Date         | Author | Summary of changes |
| ------- | ------------ | ------ | ------------------ |
| 0.1.0   | [YYYY-MM-DD] | [Name] | Initial draft      |

---

## 1. Overview

> **Core**

### 1.1 Background & Problem

[How work is done today and what goes wrong. Who suffers and how. 2–5 bullets.]

### 1.2 Purpose

[1–3 sentences: why this application exists.]

### 1.3 Objectives & Success Metrics

| ID     | Objective   | Success metric      | Today           | Target         | By when      |
| ------ | ----------- | ------------------- | --------------- | -------------- | ------------ |
| OBJ-01 | [Objective] | [How it's measured] | [Current value] | [Target value] | [YYYY-MM-DD] |

---

## 2. Users

> **Core**

### 2.1 User Roles

| Role ID | Role    | Who holds it | Responsible for    | Est. users | Device & context                    |
| ------- | ------- | ------------ | ------------------ | ---------- | ----------------------------------- |
| ROLE-01 | [Admin] | [Job title]  | [Responsibilities] | [e.g. 5]   | [Desktop in office / phone on site] |

### 2.2 Stakeholders & Personas

> **Optional:** for projects with several departments or very different user types.

| ID     | Name / group            | Type                        | What they care about    | Decision power               |
| ------ | ----------------------- | --------------------------- | ----------------------- | ---------------------------- |
| PER-01 | [Mai — project manager] | [Persona / sponsor / owner] | [Goals and pain points] | [Approve / consult / inform] |

---

## 3. Main Application

> **Core**

### 3.1 Application Concept

[The end-to-end story in plain language: what the user sees first, the main loop of work, and the outcome. One short paragraph.]

### 3.2 Application Scopes (areas)

| Scope   | Purpose                  | Who uses it |
| ------- | ------------------------ | ----------- |
| [admin] | [Back-office management] | [ROLE-01]   |
| [me]    | [Personal workspace]     | [All users] |

### 3.3 Process Today vs. With the App

> **Optional:** when the app replaces an existing manual or legacy process.

| Step | Today                     | With the app        |
| ---- | ------------------------- | ------------------- |
| [1]  | [Fill Excel and email it] | [Update in the app] |

### 3.4 Usage & Volume Expectations

| Question                                     | Answer                        |
| -------------------------------------------- | ----------------------------- |
| Users at launch / after 1 year               | [N / N]                       |
| Users active at the busiest moment, and when | [N, e.g. Monday 9 am]         |
| Key records created per day                  | [e.g. 500 orders / day]       |
| How far back users need history              | [e.g. 5 years]                |
| Impact if the app is down for 1 hour at peak | [Business consequence]        |
| Acceptable data loss                         | [None / last hour / last day] |
| Where users are located                      | [Countries / sites]           |

---

## 4. Modules & Work

> **Core**

### 4.1 Module Map

| Module code | Module    | Parent | Purpose           | Scope(s)    | Priority                |
| ----------- | --------- | ------ | ----------------- | ----------- | ----------------------- |
| [PRJ]       | [Project] | [—]    | [What it manages] | [admin, me] | [Must / Should / Could] |

### 4.2 Module Detail

> Repeat this block for every module and submodule.

#### MOD-[CODE]: [Module name]

[One sentence: what business need it serves.]

**Features**

| Feature ID  | Feature          | Actor     | Priority                        | Acceptance criteria   |
| ----------- | ---------------- | --------- | ------------------------------- | --------------------- |
| F-[CODE]-01 | [Create project] | [ROLE-xx] | [Must / Should / Could / Won't] | [Given / when / then] |

**Business rules**

| Rule ID      | Rule                                                  | Applies to  |
| ------------ | ----------------------------------------------------- | ----------- |
| BR-[CODE]-01 | [A project cannot be closed while it has open issues] | F-[CODE]-xx |

**Statuses** _(only if the main item has steps)_

| Status  | Meaning   | Who moves it here | Next statuses       |
| ------- | --------- | ----------------- | ------------------- |
| [Draft] | [Meaning] | [ROLE-xx]         | [Active, Cancelled] |

**Notifications & reports** _(only if any)_

| Item            | Type                             | Trigger / schedule | Recipient |
| --------------- | -------------------------------- | ------------------ | --------- |
| [Overdue alert] | [Notification / report / export] | [When]             | [ROLE-xx] |

### 4.3 Cross-Module Rules

> **Optional**

| Rule ID | Rule                                                   | Modules    |
| ------- | ------------------------------------------------------ | ---------- |
| BR-X-01 | [Deleting a project archives its milestones and tasks] | [PRJ, MLS] |

### 4.4 Permissions

> The single source for who can do what. ADB 04 and 05 reference this table instead of repeating it.

| Feature ID  | [ROLE-01] | [ROLE-02] | [ROLE-03] |
| ----------- | --------- | --------- | --------- |
| F-[CODE]-01 | ✅        | Own only  | ❌        |

---

## 5. Scope

> **Core**

### 5.1 In Scope (this release)

| #     | Item         | Modules / features |
| ----- | ------------ | ------------------ |
| IS-01 | [Capability] | [MOD / F-IDs]      |

### 5.2 Out of Scope

| #     | Item         | Reason        | Planned phase     |
| ----- | ------------ | ------------- | ----------------- |
| OS-01 | [Capability] | [Why not now] | [Phase 2 / never] |

### 5.3 Release Phases

> **Optional:** when delivery is split into phases.

| Phase | Goal   | Modules / features | Target date  |
| ----- | ------ | ------------------ | ------------ |
| [MVP] | [Goal] | [MOD / F-IDs]      | [YYYY-MM-DD] |

---

## 6. Limitations, Assumptions & Constraints

> **Core**

### 6.1 Known Limitations

> **Optional**

| ID     | Limitation              | Business impact | Workaround   |
| ------ | ----------------------- | --------------- | ------------ |
| LIM-01 | [No offline mode in v1] | [Impact]        | [Workaround] |

### 6.2 Assumptions

| ID   | Assumption                     | Impact if wrong | Who confirms |
| ---- | ------------------------------ | --------------- | ------------ |
| A-01 | [Believed true but unverified] | [Impact]        | [Name]       |

### 6.3 Constraints & Dependencies

| ID   | Type                                                             | Item                        | Owner  |
| ---- | ---------------------------------------------------------------- | --------------------------- | ------ |
| C-01 | [Budget / time / legal / organization / technology / dependency] | [Go-live before 2027-03-01] | [Name] |

---

## 7. Compliance & Policy

> **Optional:** when personal data, finance, or regulated work is involved.

| Regulation / policy | Applies because                   | Business requirement                            |
| ------------------- | --------------------------------- | ----------------------------------------------- |
| [PDPA]              | [We store customer personal data] | [Consent at signup; users can request deletion] |

---

## 8. Risks

> **Core**

| ID   | Risk   | Likelihood         | Impact             | Mitigation | Owner  |
| ---- | ------ | ------------------ | ------------------ | ---------- | ------ |
| R-01 | [Risk] | [Low / Med / High] | [Low / Med / High] | [Plan]     | [Name] |

---

## 9. Glossary

> **Optional:** when the business uses terms outsiders won't know.

| Term   | Meaning                   |
| ------ | ------------------------- |
| [Term] | [Meaning in this project] |

---

## 10. Open Questions

> **Core**

| ID   | Question   | Owner  | Due          | Answer    |
| ---- | ---------- | ------ | ------------ | --------- |
| Q-01 | [Question] | [Name] | [YYYY-MM-DD] | [Pending] |

---

## 11. Sign-off

> **Optional:** when formal approval is required.

| Name   | Role   | Decision                                      | Date         |
| ------ | ------ | --------------------------------------------- | ------------ |
| [Name] | [Role] | [Approved / Approved with changes / Rejected] | [YYYY-MM-DD] |
