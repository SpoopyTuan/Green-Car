# Traceability — Green Car

> Design status: TARGET V1
>
> Test status: 0 automated tests executed; named verification IDs remain PLANNED unless explicitly mapped to executable evidence

Each US maps to: screen, API operation, DB table, C4 component, module, verification. See use-cases.md, features.md, 04-information-architecture.md, and openapi.yaml.

## Functional traceability

| Requirement | Use Case | Acceptance Criteria | Screen | C4 Component | API operationId | Database | Target module / UML | Verification |
|---|---|---|---|---|---|---|---|---|
| US01 | UC01 | AC-UC01 | SCR-01 | Identity & Access | `login` | `app_users`, `audit_logs` | `features/auth`; Authentication; S01 | `INT-AUTH-001`, `SEC-AUTH-001` PLANNED |
| US02 | UC02 | AC-UC02 | Shell → SCR-01 | Identity & Access | `logout` | No revocation store | `features/auth`; Authentication; S02 | `INT-AUTH-002`, `SEC-JWT-001` PLANNED |
| US03 | UC03 | AC-UC03 | SCR-02 | Dashboard & Reporting | `getDashboard` | active cases views | Dashboard; S03 | `INT-DASH-001`, `PERF-REPORT-001` PLANNED |
| US04 | UC04 | AC-UC04 | SCR-10/11 | Case Service | `listCases`, `getCase` | `cases` | Cases; read pattern | `INT-CASE-001` PLANNED |
| US05 | UC05 | AC-UC05 | SCR-12 | Case Service | `createCase` | `cases`, `audit_logs` | Cases; create | `UNIT-APP-CASE-001`, `INT-CASE-002` PLANNED |
| US06 | UC06 | AC-UC06 | SCR-12 | Case Service | `createCase` | `cases`, `audit_logs` | Cases; create | `INT-CASE-003` PLANNED |
| US07 | UC07 | AC-UC07 | SCR-10 | Case Service | `createCase` | `cases`, `audit_logs`, `import_batches` | Cases; create/system | `INT-CASE-004` PLANNED |
| US08 | UC08 | AC-UC08 | SCR-20 | Driver Statement | `createStatement` | `driver_statements`, `audit_logs` | Driver Statement; create | `INT-STATEMENT-001` PLANNED |
| US09 | UC09 | AC-UC09 | SCR-30 | Evidence | `uploadAttachment` | `evidence`, `audit_logs` | Evidence; create | `INT-EVIDENCE-001` PLANNED |
| US10 | UC10 | AC-UC10 | SCR-50 | Excel Import | `previewImport`, `createImport`, `listImports`, `getImport` | `import_batches`, `audit_logs` | Imports; S04–S05 | `INT-IMPORT-001..003` PLANNED |
| US11 | UC11 | AC-UC11 | SCR-40 | Dashboard & Reporting | `getReport` | reporting views | Reports; read | `INT-REPORT-001`, `PERF-REPORT-001` PLANNED |
| US12 | UC12 | AC-UC12 | SCR-40 | Dashboard & Reporting | `exportReport` | reporting views, `audit_logs` | Reports/exporters | `INT-REPORT-002`, `SEC-EXPORT-001` PLANNED |
| US13 | UC13 | AC-UC13 | SCR-60 | Audit Log | `listAuditLogs` | `audit_logs` | Audit; read | `INT-AUDIT-001`, `SEC-AUDIT-002` PLANNED |
| US14 | UC14 | AC-UC14 | SCR-70 | User & Profile | `listUsers`, `createUser`, `updateUser`, `updateUserStatus` | `app_users`, `audit_logs` | Users | `INT-RBAC-ADMIN-001`, `INT-USERS-001` PLANNED |
| US15 | UC15 | AC-UC15 | SCR-80 | User & Profile | `getProfile`, `updateProfile`, `changePassword` | `app_users`, `audit_logs` | Profile | `INT-PROFILE-001`, `INT-AUTH-003` PLANNED |
| US16 | UC16 | AC-UC16 | System retrieval | Policy RAG | rag retrieval | `policy_versions`, `policy_chunks` | RAG; S11 | `INT-RAG-001` PLANNED |
| US17 | UC17 | AC-UC17 | System/CSKH SLA monitoring | SLA & Notifications | sla monitoring | `sla`, `notifications`, `audit_logs` | SLA; S12 | `INT-SLA-001`, `INT-NOTIF-001` PLANNED |

## Quality/NFR traceability

| Requirement | C4 Component | API / Module | Implementation Mechanism | Verification |
|---|---|---|---|---|
| NFR-PERF-001/002 | HTTP API, Persistence | CRUD modules | paginated parameterized queries; bounded writes and audit transaction | `PERF-CRUD-001/002` PLANNED |
| NFR-PERF-003/004 | Dashboard & Reporting | `getDashboard`, `getReport`, `exportReport` | active indexed PostgreSQL views; bounded synchronous export | `PERF-REPORT-001`, `PERF-EXPORT-001` PLANNED |
| NFR-CAP-001/002 | Backend, PostgreSQL | all | modular monolith and representative indexed dataset | `PERF-CAP-001`, `PERF-CONC-001` PLANNED |
| NFR-IMPORT-001/002 | Excel Import, Persistence | import operations | signature/type checks; authoritative validators; one transaction | `INT-IMPORT-001..003`, `PERF-IMPORT-001` PLANNED |
| NFR-SEC-001 | Identity & Access | login/logout | 30-minute JWT; client discard; no refresh/revocation store | `INT-AUTH-001/002`, `SEC-JWT-001` PLANNED |
| NFR-SEC-002/003 | Identity & Access | login/changePassword | minimum 8 characters; 5 failures/15 min account lockout | `INT-AUTH-002`, `SEC-AUTH-001` PLANNED |
| NFR-SEC-004 | Identity & Access | protected operations | deny-by-default RBAC and ownership policies | `INT-RBAC-*`, `ARCH-009` PLANNED |
| NFR-SEC-005 | Audit Log, HTTP API | cross-cutting | structured redaction; CREATE/UPDATE/SOFT_DELETE/RESTORE; correlation ID | `SEC-LOG-001`, `SEC-AUDIT-001` PLANNED |
| NFR-MAINT-001/002 | all backend components | solution/CI | dependency rules; coverage/static/contract gates | CI coverage PLANNED |
| NFR-REL-001 | Backend, PostgreSQL | deployment | health monitoring and monthly availability calculation | `OPS-AVAIL-001` PLANNED |
| NFR-BCP-001 | PostgreSQL | operations | daily backup, 30-day retention, RPO 24 h, RTO 4 h | `OPS-RESTORE-001` PLANNED |
| NFR-OBS-001 | HTTP API | middleware/health | structured logs, trace ID, health endpoints, secret exclusion | `INT-OBS-001`, `SEC-LOG-001` PLANNED |
| NFR-ACC-001 | React Web | all screens | labels, non-color status, keyboard/focus checks | `A11Y-001` PLANNED |

## Conformance

Every OpenAPI operation is represented above through its owning user story. No planned identifier is evidence of a passing test merely because the aggregate suite passes. Named tests must be mapped before changing an individual verification ID to `PASS`. Remaining work is listed in [Project Progress](../../README.md#project-progress).
