# INVEST Requirements — Green Car V1

> Design status: TARGET V1

Each requirement is small enough to implement and test independently. Cross-cutting authentication, authorization, validation, auditing, and error handling are acceptance constraints, not hidden features.

## INVEST checklist

| Principle | Application in this backlog |
|---|---|
| Independent | Each story produces a demonstrable outcome and identifies only unavoidable prerequisites. |
| Negotiable | UI layout and implementation details may change while the stated outcome and permissions remain fixed. |
| Valuable | Every story names the user benefit. |
| Estimable | Inputs, outputs, roles, and exceptional outcomes are explicit. |
| Small | Each story targets one capability and can be split into API, application, persistence, and UI tasks. |
| Testable | Every story links to observable acceptance criteria in `06-acceptance-criteria.md`. |

## Product backlog

| ID | User story | Value | Dependencies | Acceptance criteria |
|---|---|---|---|---|
| US01 | As an active user, I want to log in so that I can securely access my permitted workspace. | Protected access | Seeded user account | AC-UC01 |
| US02 | As an authenticated user, I want to log out so that this client discards its V1 access token and private session state. | Session safety | US01 | AC-UC02 |
| US03 | As any role, I want a filtered dashboard so that I can understand cases, SLA, and recent activity. | Fast overview | US01, case data | AC-UC03 |
| US04 | As any role, I want to search and inspect active cases so that I can review complaints without seeing deleted data. | Case visibility | US01 | AC-UC04 |
| US05 | As a Customer/CSKH/Admin, I want to submit a complaint via App so that issues are recorded consistently. | Complete case data | US01, categories | AC-UC05 |
| US06 | As a CSKH/Admin, I want to submit a complaint via Call so that phone-reported issues are recorded. | Complete case data | US01 | AC-UC06 |
| US07 | As the system, I want to ingest email complaints so that inbound mail becomes a case. | Automated intake | US01, IMAP | AC-UC07 |
| US08 | As a Driver, I want to submit a statement so that my side is recorded. | Fair process | US04 | AC-UC08 |
| US09 | As any role, I want to attach evidence so that supporting files are linked to a case. | Traceability | US04 | AC-UC09 |
| US10 | As a CSKH/Admin, I want to import Excel so that bulk data is accepted only when every row is valid. | Safe bulk entry | US01, categories | AC-UC10 |
| US11 | As a CSKH/Admin, I want filtered reports so that I can analyze results by time and category. | Insight | US03 | AC-UC11 |
| US12 | As a CSKH/Admin, I want to export the current report so that I can share the selected view. | Portable reporting | US11 | AC-UC12 |
| US13 | As a CSKH/Admin, I want to inspect audit events so that important actions are traceable. | Accountability | US01 | AC-UC13 |
| US14 | As an Admin, I want to create, edit, enable, and disable users so that access remains controlled. | Access administration | US01 | AC-UC14 |
| US15 | As an authenticated user, I want to maintain my profile and password without changing my role. | Self-service | US01 | AC-UC15 |
| US16 | As the system, I want to run policy RAG so that relevant policies are retrieved for a case. | AI support | US01 | AC-UC16 |
| US17 | As a CSKH/Admin, I want SLA monitoring and notifications so that deadlines are tracked. | SLA compliance | US01 | AC-UC17 |

## Per-story INVEST review

`PASS` means the current design provides enough information for planning and acceptance. It does not claim that implementation or tests already exist.

| Story | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US01 | TODO | TODO | TODO | TODO | TODO | TODO | Authentication not yet implemented. |
| US02 | TODO | TODO | TODO | TODO | TODO | TODO | Logout not yet implemented. |
| US03 | TODO | TODO | TODO | TODO | TODO | TODO | Dashboard not yet implemented. |
| US04 | TODO | TODO | TODO | TODO | TODO | TODO | View cases not yet implemented. |
| US05 | TODO | TODO | TODO | TODO | TODO | TODO | Submit complaint (App) not yet implemented. |
| US06 | TODO | TODO | TODO | TODO | TODO | TODO | Submit complaint (Call) not yet implemented. |
| US07 | TODO | TODO | TODO | TODO | TODO | TODO | Ingest email complaint not yet implemented. |
| US08 | TODO | TODO | TODO | TODO | TODO | TODO | Driver statement not yet implemented. |
| US09 | TODO | TODO | TODO | TODO | TODO | TODO | Attach evidence not yet implemented. |
| US10 | TODO | TODO | TODO | TODO | TODO | TODO | Import not yet implemented. |
| US11 | TODO | TODO | TODO | TODO | TODO | TODO | Reports not yet implemented. |
| US12 | TODO | TODO | TODO | TODO | TODO | TODO | Export not yet implemented. |
| US13 | TODO | TODO | TODO | TODO | TODO | TODO | Audit not yet implemented. |
| US14 | TODO | TODO | TODO | TODO | TODO | TODO | User management not yet implemented. |
| US15 | TODO | TODO | TODO | TODO | TODO | TODO | Profile not yet implemented. |
| US16 | TODO | TODO | TODO | TODO | TODO | TODO | RAG not yet implemented. |
| US17 | TODO | TODO | TODO | TODO | TODO | TODO | SLA not yet implemented. |

## Cross-cutting acceptance constraints

- Only `ADMIN`, `CSKH`, `CUSTOMER`, and `DRIVER` roles exist.
- The API, not the frontend, enforces RBAC and record ownership.
- Active case queries exclude soft-deleted rows.
- API errors use the documented error contracts.
- Import is all-or-nothing: any invalid row prevents insertion for that batch.
- File type and size are validated server-side before parsing or storage.
- Policy RAG only uses ACTIVE policies within validity period with correct embedding model.

**Related:** Use cases · Acceptance criteria · API catalog (see project docs).
