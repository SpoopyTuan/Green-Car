# Green Car

Complaint management system for a transport company: receiving customer complaints, driver statements, evidence, CSKH review and response.

## 1. Requirements — INVEST, Use Cases, and Acceptance Criteria

### 1.1 Product mind map

![Mindmap](docs/mindmap.png)

### 1.2 INVEST requirements

| Criterion | Meaning in Green Car |
|---|---|
| Independent | Each story provides a clear capability; depends on others only when necessary. |
| Negotiable | UI and technical details may change as long as the main goal (case lifecycle) and permission rules remain. |
| Valuable | Each story delivers value to customers, drivers, CSKH, or the business. |
| Estimable | Roles, inputs, outputs, dependencies, and error outcomes (SLA, email sending) are documented. |
| Small | Each story covers one capability and can be split into API, agent node, persistence, UI, and test tasks. |
| Testable | Every story maps to observable acceptance criteria and expected authorization/validation/error behavior. |

```
US01–US02   Authentication and session
US03        Dashboard
US04        View cases
US05        Complaint intake (App)
US06        Complaint intake (Call)
US07        Complaint intake (Email)
US08        Driver statement
US09        Evidence & attachments
US10        Import Excel
US11–US12   Reports and export
US13        Audit log
US14        User management
US15        Personal profile
US16        Policy RAG
US17        SLA and notifications
```

### 1.3 Use cases

```mermaid
flowchart TB
    subgraph roles["Actors"]
        direction LR
        admin(["Administrator"])
        cskh(["CSKH"])
        customer(["Customer"])
        driver(["Driver"])
    end

    subgraph system["Green Car — System Boundary"]
        direction LR
        adminCases["UC01 Login<br/>UC02 Logout<br/>UC03 Dashboard<br/>UC13 Audit<br/>UC14 Users<br/>UC15 Profile"]
        cskhCases["UC01-02<br/>UC03<br/>UC11 Reports<br/>UC13 Audit<br/>UC15<br/>UC04-09 Cases<br/>UC10 Import"]
        customerCases["UC01-02<br/>UC03<br/>UC04-07 Own cases<br/>UC15"]
        driverCases["UC01-02<br/>UC03<br/>UC08-09 Own statements<br/>UC15"]
    end

    admin --> adminCases
    cskh --> cskhCases
    customer --> customerCases
    driver --> driverCases
```

For detailed actors, permissions, and business flows, see [Actors, roles, and use cases](docs/03-use-cases.md).

### 1.4 Roles and permissions

| Capability | Admin | CSKH | Customer | Driver |
|---|---|---|---|---|
| View dashboard/case | Yes | Yes | Yes (own) | Yes (own) |
| Create complaint | Yes | Yes | Yes | No |
| Edit case | Yes | Yes | Own | No |
| Soft delete | Yes | Yes | No | No |
| Import Excel | Yes | Yes | No | No |
| Reports | Yes | Yes | No | No |
| Audit log | Yes | Yes | No | No |
| User management | Yes | No | No | No |

Backend must enforce RBAC; hiding controls in frontend is UX only.

## 2. Information Architecture, Screen Hierarchy, and UI/UX

### 2.1 Information architecture

```
Public
└── Login

Authenticated
├── Dashboard
├── Cases (App/Call/Email)
├── Driver Statements
├── Evidence
├── Reports
├── Import
├── Audit
├── Users
├── Profile
└── Forbidden
```

### 2.2 Screen hierarchy
Login → authenticated shell → feature pages; interactions use modals. Unauthorized navigation → Forbidden.
Detailed screen IDs, user flows, screen states, accessibility rules, and API mappings are maintained in [Information Architecture](docs/04-information-architecture.md).

### 2.3 UI prototype

#### Login

![Login](docs/images/login.png)

#### Dashboard
![Dashboard](docs/images/dashboard.png)
## 3. C4 Architecture

### C1 — System Context

```mermaid
flowchart LR
    customer(["Customer"])
    driver(["Driver"])
    cskh(["CSKH"])
    admin(["Administrator"])
    subgraph boundary[" "]
        green["Green Car\n[Software System]\nManages complaints, evidence,\nreports, imports, users, audit"]
    end
    customer -- "submit complaints" --> green
    driver -- "submit statements" --> green
    cskh -- "review & resolve" --> green
    admin -- "manage users & data" --> green
```

### C2 — Container

```mermaid
flowchart TB
    user(["Users"])
    subgraph platform["Green Car"]
        direction TB
        web["Web Frontend\n[React + Vite]"]
        backend["Backend\n[FastAPI]"]
        db[("PostgreSQL + pgvector")]
    end
    gemini[("Gemini API")]
    mail[("IMAP/SMTP")]
    tel[("Stringee Telephony")]
    user --> web
    web -- REST/JSON --> backend
    backend -- SQL --> db
    backend --> gemini
    backend --> mail
    backend --> tel
```

### C3 — Component

```mermaid
flowchart TB
    subgraph be["FastAPI Backend"]
        api["HTTP API"]
        identity["Identity & Access"]
        case["Case Service"]
        driver["Driver Statement"]
        evidence["Evidence"]
        importer["Excel Import"]
        reporting["Dashboard & Reporting"]
        audit["Audit Log"]
        rag["Policy RAG"]
        persist["Persistence"]
    end
    web["Web Frontend"]
    db[("PostgreSQL")]
    web --> api
    api --> identity
    api --> case
    api --> driver
    api --> evidence
    api --> importer
    api --> reporting
    identity --> persist
    case --> persist
    driver --> persist
    evidence --> persist
    importer --> persist
    reporting --> persist
    case -.-> audit
    importer -.-> audit
    audit --> persist
    persist --> db
    rag --> persist
```

## 4. Database Diagrams and OpenAPI 3.0 Contract

### 4.1 Database

```mermaid
erDiagram
    app_users ||--o{ cases : creates
    app_users ||--o{ driver_statements : creates
    app_users ||--o{ evidence : uploads
    app_users ||--o{ import_batches : imports
    app_users ||--o{ audit_logs : performs
    cases ||--o{ driver_statements : has
    cases ||--o{ evidence : has
    cases ||--o{ attachments : has
    import_batches ||--o{ cases : contains
    policy_versions ||--o{ policy_chunks : chunks

    app_users {
        bigint id PK
        varchar email
        varchar password_hash
        varchar full_name
        varchar phone
        user_role role
        boolean is_active
        timestamptz created_at
        timestamptz deleted_at
    }
    cases {
        bigint id PK
        varchar source
        varchar status
        varchar severity
        bigint created_by FK
        bigint import_batch_id FK
        timestamptz created_at
        timestamptz deleted_at
    }
    driver_statements {
        bigint id PK
        bigint case_id FK
        bigint driver_id FK
        text statement
        timestamptz created_at
    }
    evidence {
        bigint id PK
        bigint case_id FK
        bigint uploaded_by FK
        varchar file_path
        timestamptz created_at
    }
    attachments {
        bigint id PK
        bigint case_id FK
        bigint uploaded_by FK
    }
    import_batches {
        bigint id PK
        varchar import_type
        varchar status
        bigint imported_by FK
        timestamptz created_at
    }
    audit_logs {
        bigint id PK
        bigint actor_user_id FK
        varchar action
        varchar module
        timestamptz created_at
    }
    policy_versions {
        bigint id PK
        varchar name
        varchar status
        timestamptz effective_from
    }
    policy_chunks {
        bigint id PK
        bigint policy_version_id FK
        text content
        vector embedding
    }
```

See the [detailed database diagram](docs/DATABASE.md), [data model](docs/05-data-model.md),
and [traceability matrix](docs/traceability/matrix.md).

### 4.2 OpenAPI 3.0 API documentation

The HTTP contract is defined with OpenAPI 3.0.4 and organized into Authentication, Dashboard, Cases, Driver Statements, Evidence, Reports, Imports, Audit, Users, and Profile.

```
18 paths
26 operations
Bearer JWT security
Role-based access (ADMIN, CSKH, CUSTOMER, DRIVER)
JSON request/response contracts
Reusable request/response schemas
Application error responses
Request, success, and error examples
```

Every operation has a stable `operationId`, documented parameters, request and response schemas, success/error status codes, and target authorization rules. The contract is a design source of truth; the backend runtime has not been implemented yet.

- [OpenAPI 3.0.4 specification](docs/api/openapi.yaml)

## 5. Folder structure

### Current repository layout

```
Green Car/
├── README.md
├── .gitignore
├── docs/
│   ├── 01-scope.md
│   ├── 02-features.md
│   ├── 03-use-cases.md
│   ├── 04-information-architecture.md
│   ├── 05-data-model.md
│   ├── 06-acceptance-criteria.md
│   ├── 07-folder-structure.md
│   ├── 08-invest-requirements.md
│   ├── DATABASE.md
│   ├── requirements/
│   │   ├── business-rules.md
│   │   └── non-functional-requirements.md
│   ├── api/
│   │   └── openapi.yaml
│   ├── architecture/
│   │   └── c4/
│   │       └── README.md
│   ├── traceability/
│   │   └── matrix.md
│   ├── images/
│   │   ├── dashboard.png
│   │   └── login.png
│   ├── mindmap.png
│   └── AI Agent for complaint handling.pptx
└── src/
    ├── backend/
    └── frontend/
```

### Target Green Car backend layout

```
src/
├── main.py, config.py, core.py
├── api/
├── modules/
├── agents/
├── integrations/
├── rag/
├── db/
├── services/
├── repositories/
└── security/
```

## 6. Detailed Class and Sequence Design

### 6.1 Case lifecycle sequence

```mermaid
sequenceDiagram
    participant C as Customer
    participant API as FastAPI
    participant A as AI Agent
    participant D as Driver
    participant K as CSKH
    C->>API: Submit complaint (App/Call/Email)
    API->>A: Intake + categorize
    A-->>API: Case created
    API->>D: Request statement
    D->>API: Submit statement + evidence
    API->>A: Evidence extraction
    A-->>API: Proposal
    API->>K: Review
    K->>API: Approve & send response
    API->>C: Send approved response (SMTP)
    API-->>K: CLOSED
```

### 6.2 Core class diagrams

#### Authentication

```mermaid
classDiagram
    direction TB
    class AuthController {
        +login() Response
        +logout() Response
    }
    class LoginRequest {
        +string email
        +string password
    }
    class LoginResponse {
        +string accessToken
        +string tokenType
        +DateTime expiresAt
        +UserResponse user
    }
    class UserResponse {
        +long id
        +string email
        +string fullName
        +UserRole role
    }
    class IAuthService {
        <<interface>>
        +login(email, password) TokenResult
        +logout(userId) void
    }
    class AuthService
    class TokenResult {
        +string accessToken
        +DateTime expiresAt
    }
    class AuthResult {
        +string accessToken
        +DateTime expiresAt
        +AppUser user
    }
    class AppUser {
        +long id
        +string email
        +string passwordHash
        +string fullName
        +UserRole role
        +boolean isActive
        +DateTime? deletedAt
    }
    class IUserRepository {
        <<interface>>
        +findByEmail(email) AppUser
        +updateLastLogin(userId) void
    }
    class ITokenIssuer {
        <<interface>>
        +issue(user) TokenResult
    }
    class IAuditLogRepository {
        <<interface>>
        +add(entry) void
    }
    class UserRepository
    class PasswordHasher
    class JwtTokenIssuer
    class AuditLogRepository

    AuthController --> LoginRequest
    AuthController --> LoginResponse
    AuthController --> IAuthService
    AuthService ..|> IAuthService
    AuthService --> IUserRepository
    AuthService --> IPasswordHasher
    AuthService --> ITokenIssuer
    AuthService --> IAuditLogRepository
    AuthService --> AuthResult
    AuthResult --> AppUser
    UserRepository ..|> IUserRepository
    PasswordHasher ..|> IPasswordHasher
    JwtTokenIssuer ..|> ITokenIssuer
    AuditLogRepository ..|> IAuditLogRepository
```

#### Case

```mermaid
classDiagram
    direction TB
    class CaseController {
        +listCases() CaseResponse list
        +getCase(id) CaseResponse
        +createCase(request) CaseResponse
        +updateCase(id, request) CaseResponse
        +softDeleteCase(id) void
    }
    class CaseResponse {
        +long id
        +string source
        +string status
        +string severity
        +DateTime createdAt
    }
    class ICaseService {
        <<interface>>
        +listCases(context) CaseResponse list
        +getCase(id, context) CaseResponse
        +createCase(draft, context) CaseResponse
        +updateCase(id, draft, context) CaseResponse
        +softDeleteCase(id, context) void
    }
    class CaseService
    class CasePolicy {
        +ensureCanCreate(context) void
        +ensureCanUpdate(context, case) void
        +ensureCanDelete(context, case) void
    }
    class CaseValidator {
        +validate(draft) ValidationResult
    }
    class ValidationResult {
        +boolean valid
        +list errors
    }
    class Case {
        +long id
        +string source
        +string status
        +string severity
        +long createdBy
        +DateTime createdAt
        +DateTime? deletedAt
        +softDelete(userId, time) void
    }
    class CaseDraft
    class ActorContext {
        +long userId
        +UserRole role
    }
    class ICaseRepository {
        <<interface>>
        +listCases() Case list
        +findActive(id) Case
        +add(case) void
        +update(case) void
    }
    class CaseRepository

    CaseController --> CaseResponse
    CaseController --> ICaseService
    CaseService ..|> ICaseService
    CaseService --> CasePolicy
    CaseService --> CaseValidator
    CaseService --> ICaseRepository
    CaseService --> Case
    CaseService --> CaseDraft
    CasePolicy --> ActorContext
    CasePolicy --> Case
    CaseValidator --> CaseDraft
    CaseValidator --> ValidationResult
    CaseRepository ..|> ICaseRepository
```

#### Driver Statement & Evidence

```mermaid
classDiagram
    direction TB
    class StatementController {
        +createStatement(request) StatementResponse
    }
    class EvidenceController {
        +uploadEvidence(request) EvidenceResponse
    }
    class DriverStatement {
        +long id
        +long caseId
        +long driverId
        +string statement
        +DateTime createdAt
    }
    class Evidence {
        +long id
        +long caseId
        +long uploadedBy
        +string filePath
        +DateTime createdAt
    }
    class IStatementRepository {
        <<interface>>
        +add(statement) void
    }
    class IEvidenceRepository {
        <<interface>>
        +add(evidence) void
    }

    StatementController --> DriverStatement
    EvidenceController --> Evidence
```

### 6.3 Sequence diagrams

- **Intake** — Customer submits complaint → API → AI Agent categorizes → Case created
- **Categorization** — AI Agent classifies group, looks up trip, assesses severity
- **Handling** — Driver submits statement + evidence → AI aggregates & cross-checks → AI proposes handling direction
- **Driver complaint** — Driver submits statement → linked to case → CSKH notified
- **Evidence extraction** — AI extracts evidence details → confidence score → flagged for review if low
- **CSKH review** — CSKH reviews AI proposal → approves, edits, or rejects → sends response
- **Send response** — Approved response sent via SMTP to customer → case marked CLOSED
- **Audit** — Every create/update/delete/login/import/export written to audit_logs

### 6.4 Remaining design

Class diagrams for Reporting, Import, Audit, and RAG modules, plus additional sequence diagrams for import validation, report generation, and notification flows, are planned but not yet documented.


## Documentation

Documentation lives in the `docs/` folder of the project.

- Requirements: [Scope](docs/01-scope.md), [Features](docs/02-features.md), [INVEST backlog](docs/08-invest-requirements.md), [Acceptance criteria](docs/06-acceptance-criteria.md)
- Requirements catalogs: [Business rules](docs/requirements/business-rules.md), [Non-functional requirements](docs/requirements/non-functional-requirements.md)
- UI/UX: [Information architecture](docs/04-information-architecture.md)
- Database: [Data model](docs/05-data-model.md), [ER diagram](docs/DATABASE.md)
- Folder structure: [React and three-tier backend](docs/07-folder-structure.md)
- API contract: [OpenAPI 3.0.4 specification](docs/api/openapi.yaml)
- Traceability: [Master traceability matrix](docs/traceability/matrix.md)

## Project Progress
- [x] Define scope, goals, and roles
- [x] Write feature list and use cases
- [x] Write user stories following INVEST
- [x] Write acceptance criteria and business rules
- [x] Write non-functional requirements
- [x] Design information architecture and screen permissions
- [x] Design C4 System Context, Container, Component diagrams
- [x] Design PostgreSQL schema, tables, and relationships
- [x] Write API documentation in OpenAPI 3.0
- [x] Design backend folder structure
- [ ] Design React frontend folder structure
- [x] Design class diagrams and sequence diagrams
- [x] Create traceability between requirements, screens, API, and database
- [ ] Build the three-tier backend
- [ ] Build authentication, user management, and personal profile API
- [ ] Implement case CRUD API
- [ ] Implement AI agent / LangGraph integration (intake, categorization, evidence extraction)
- [ ] Implement Policy RAG (US16)
- [ ] Implement SLA monitoring and notifications (US17)
- [ ] Implement IMAP email complaint ingestion (US07)
- [ ] Implement Stringee Telephony integration for call complaints
- [ ] Complete dashboard, reports, import, attachments, and audit API
- [ ] Write unit tests and API integration tests
- [ ] Connect React frontend to the API
- [ ] Implement Excel import with row-level preview and PDF/XLSX report export
- [ ] Record audit events for create, update, delete, login, logout, import, and export
- [ ] Add frontend tests with Vitest
- [ ] Complete PostgreSQL migrations and integration tests against a real database
- [ ] Add durable file storage for imports and attachments
- [ ] Standardize JWT, rate limiting, authorization, and security tests