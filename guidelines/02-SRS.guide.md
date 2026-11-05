# Guideline — ADB 02: Software Requirements Specification (SRS)

Explains every section of `templates/02-SRS.md`. Section numbers match the template.

**Purpose of this book:** the technical view of the whole system. It turns the BRD's business answers into measurable targets, an architecture, workflows, security, infrastructure, delivery pipeline, and recovery plan.

**Rules**
- **Trace back to the BRD.** NFRs cite the BRD answer they come from; workflows cite features; DR targets cite the acceptable-loss answers.
- **System level, not endpoint level.** Show workflows and components here; field-by-field contracts belong in ADB 04, tables in ADB 03, screens in ADB 05.
- **Every number is justified.** If a target can't be traced to a BRD answer or a constraint, record it as an assumption.
- **Decisions go in ADRs (§13)**, including any change to the fixed conventions in ADB 04/05.

---

## 0. Document Info
Same fields as the BRD, plus **Based on BRD version**: the BRD version this SRS was written against. When the BRD changes, review the SRS.

---

## 1. Non-Functional Requirements

| Field | Meaning | Example |
|---|---|---|
| ID | `NFR-NN` | `NFR-01` |
| Category | Performance, Availability, Scalability, Security, Usability/Accessibility, Maintainability, Compatibility, Data retention, Localization | Performance |
| Requirement | Quality in words | Search screens respond quickly at peak |
| Measurable target | A number or verifiable condition | p95 `POST …/search` < 500 ms at 150 req/s |
| Derived from (BRD) | The business answer behind it | BRD 3.5: 120 users active at Monday 9 am |

**Translation cheatsheet**

| BRD answer | NFR |
|---|---|
| "Down for an hour = meetings without data, not costly" | Availability 99.5% monthly; single region acceptable |
| "Down for an hour = we lose sales" | Availability 99.9%+; multi-AZ, health-checked autoscaling |
| "Users in the field with bad signal" | Compatibility: works on 3G; consider offline cache |
| "Must keep records 5 years for audit" | Data retention 5 years; archive after 2 |
| "Thai and English users" | Localization: th/en UI, Asia/Bangkok display time, UTC storage |

### 1.1 Capacity Estimate
Show the arithmetic so reviewers can check it.

`peak RPS ≈ peak concurrent users × actions per user per minute ÷ 60 × requests per action`

Example: 120 users × 4 actions/min ÷ 60 × 3 requests ≈ 24 req/s → design target 100 req/s (×4 headroom). Data growth: records/day × average row size × 365, plus indexes (~×1.5).

---

## 2. Architecture

| Section | What to write | Tip |
|---|---|---|
| 2.1 Architecture Style | Monolith / modular monolith / microservices / serverless + why | A modular monolith (one Express app, one folder per module) is the default for this stack. Justify anything more distributed with an ADR. |
| 2.2 System Context | The system as one box with users and external systems around it | Keep to one diagram a business person can follow |
| 2.3 Container / Component | Deployable pieces (FE, BE, DB, cache, storage, jobs) and their connections | Label arrows with protocol (HTTPS, SQL, queue) |
| 2.4 System Components | Responsibility, technology, who it talks to, owner | One responsibility per component |

### 2.5 Tech Stack
| Field | Meaning | Example |
|---|---|---|
| Layer | Part of the system | Validation |
| Technology | Chosen tool | Zod |
| Version | Pinned major | 3.x |
| Reason for choice | Why it fits **this** project (requirement, constraint, team skill) | Shared schemas FE/BE; required by team convention |
| Alternatives considered | What else was evaluated | Joi, class-validator |

The fixed stack for this ADB is Next.js + shadcn/ui + Tailwind + ESLint (FE) and TypeScript + Express + Zod + PostgreSQL (BE). Fill the remaining rows (ORM/query builder, cache, hosting, monitoring) from the prototype or the interview.

---

## 3. Workflows

### 3.1 Workflow Catalog
| Field | Meaning | Example |
|---|---|---|
| Workflow ID | `WF-NN` | `WF-04` |
| Workflow | A user or system journey crossing components | Close project |
| Modules | Modules involved | PRJ, MLS, ISS |
| Trigger | What starts it | PM clicks Close |
| Related features | BRD feature IDs | F-PRJ-03 |

**Which workflows to include:** login/logout/token refresh, every create/edit with business rules, every status transition, anything touching an external service, file upload, background jobs. Plain CRUD with no rules can share one generic workflow.

### 3.2 Sequence Diagrams
One per workflow. Use the real endpoint paths from ADB 04 (`PUT /api/v1/project/:projectId/close`) and the real response codes. Always show the validation failure branch, the main business-rule failure, and timeouts/retries for external calls. The notes table lists which `BR-` rules apply at which step.

### 3.3 State Diagrams
One per entity with a status. States use the **exact enum values** from ADB 03 (lowercase snake_case). The transition table lists the endpoint that performs each transition, the roles allowed, and the guarding rule. Transitions not listed must be rejected by the API.

### 3.4 Background Jobs
| Field | Meaning | Example |
|---|---|---|
| Job ID | `JOB-NN` | `JOB-02` |
| Schedule / trigger | Cron expression or event | `0 1 * * *` (01:00 daily) |
| What it does | Effect | Mark overdue tasks and notify owners |
| Retry policy | Attempts and backoff | 3 attempts, exponential |
| Failure alert | Where failures go | #alerts |

---

## 4. Data Flow

| Section | What to write |
|---|---|
| 4.1 Diagram | Sources → validation → services → storage → consumers (views, reports, exports, integrations) |
| 4.2 Catalog | One row per significant flow: source, data, destination, transform, PII yes/no, frequency |

**Tip:** mark every flow carrying PII. Those rows drive §6.3 security measures and ADB 03 §7 retention.

---

## 5. Integrations
| Field | Meaning | Example |
|---|---|---|
| ID | `INT-NN` | `INT-01` |
| Service | External system | LINE Messaging API |
| Purpose | Why | Notify overdue tasks |
| Type | REST / SDK / webhook / file | REST |
| Direction | Outbound (we call) / inbound (they call us at `/api-ext/v1`) | Outbound |
| Auth method | How we authenticate | Channel access token |
| Rate limit | Provider limit | 500 msg/min |
| Failure fallback | Behavior if unavailable | Queue and retry 3×, then email |
| Docs | Official link | URL |

---

## 6. Security

### 6.1 Authentication
Fill each row concretely. Example: login via Google SSO (NextAuth), backend verifies a JWT signed by NextAuth, access token 15 min, refresh by NextAuth session rotation, MFA via Google, logout clears the session cookie.

### 6.2 Authorization
Describe the mechanism: `auth.middleware` sets `req.user`; `permission.middleware` checks a permission key (`project.edit`) against the user's roles; services check ownership. The endpoint-level map lives in ADB 04 §6.3.

### 6.3 Security Measures
| Area | What good looks like |
|---|---|
| Data in transit | HTTPS everywhere, HSTS |
| Data at rest | Encrypted DB and backups; sensitive columns encrypted if required |
| Secrets | Secret manager or CI secrets; `.env` never committed |
| Input validation | Zod on params, query and body of every endpoint |
| Rate limiting | Global per IP + stricter on login |
| CORS | Only the frontend origins per environment |
| Security headers | helmet defaults + CSP on the frontend |
| Audit log | Who/what/when for create, edit, delete, status changes, permission changes |
| Dependency scanning | Dependabot / `npm audit` in CI |
| File upload | Allowed MIME types, size limit, private bucket, signed URLs |

### 6.4 Compliance Implementation
Each BRD §7 item → the concrete technical measure (table, endpoint, job, setting).

---

## 7. Infrastructure

| Section | What to write |
|---|---|
| 7.1 Deployment Diagram | Where each container runs, networking, managed services |
| 7.2 Inventory | Provider, size/tier, region, notes, and an estimated monthly cost (compare with the BRD budget constraint) |
| 7.3 Environments | URLs for FE and BE, purpose, data type, deploy trigger |

**Tip:** staging should match production's shape (same services, smaller sizes). Never put real personal data outside production unless anonymized.

---

## 8. CI/CD Pipeline

| Section | What to write |
|---|---|
| 8.1 Branch Strategy | Model, branch names (`feat/…`, `fix/…`), PR rules (reviewers, required checks) |
| 8.2 Pipeline Stages | Each stage, the tool, when it runs, and what fails the build |
| 8.3 Release & Rollback | Version tags, release notes, how to roll back app and DB |

**Migration rule:** migrations run before the new app version starts and must be backward compatible with the previous version (expand → migrate → contract) so a rollback doesn't break.

---

## 9. Observability
Logs (structured JSON with `requestId` from the BE middleware), metrics (latency, error rate, saturation), error tracking on FE and BE, uptime checks on `/health`, and job failures. Each row needs an alert condition and a recipient, or it won't be noticed.

---

## 10. Backup & Disaster Recovery

| Term | Meaning | Comes from |
|---|---|---|
| RPO | Maximum acceptable data loss | BRD 3.5 "acceptable data loss" |
| RTO | Maximum acceptable downtime | BRD 3.5 "impact of downtime" |

| Section | What to write |
|---|---|
| 10.2 Backup Plan | Per asset: method, frequency, retention, location (preferably another region/account), encryption |
| 10.3 Disaster Scenarios | At minimum: data corruption/accidental delete, provider/region outage, bad deployment, leaked secret. Detection, steps, owner, expected time (must be ≤ RTO) |
| 10.4 DR Testing | Scheduled restore tests and results |

**Tip:** a backup you've never restored is not a backup.

---

## 11. Testing Strategy
Per test type: tool, scope, coverage target, and when it runs. Unit tests focus on `*.service.ts` (where business rules live). Bruno collections in `bruno-API/` mirror the route catalog. E2E covers the few flows that lose money or trust if broken.

## 12. Traceability Matrix
One row per BRD feature, linking workflow → endpoint(s) → table(s) → screen(s) → test. Gaps in this table are gaps in the design: a feature with no endpoint, or an endpoint with no feature.

## 13. ADRs
| Field | Meaning |
|---|---|
| Context | The problem and forces (requirements, constraints) |
| Decision | What was chosen |
| Alternatives | What else was considered |
| Consequences | Trade-offs, good and bad |

Never edit an accepted ADR to change the decision; add a new one that supersedes it.

## 14. Open Questions
Global numbering shared with all books (see BRD guideline §10).
