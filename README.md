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
US04–US06   Complaint intake (App/Call/Email)
US07–US09   Driver statements and evidence
US10        Import Excel
US11–US12   Reports and export
US13        Audit log
US14        User management
US15        Personal profile and password
US16        Policy RAG and evaluation
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
        adminCases["Admin: UC01 Login · UC02 Logout · UC03 Dashboard · UC13 Audit · UC14 Users · UC15 Profile"]
        cskhCases["CSKH: UC01-02 · UC03 · UC11 Reports · UC13 Audit · UC15 · UC04-09 Cases · UC10 Import"]
        customerCases["Customer: UC01-02 · UC03 · UC04-07 Own cases · UC15"]
        driverCases["Driver: UC01-02 · UC03 · UC08-09 Own statements · UC15"]
    end
    admin --> adminCases
    cskh --> cskhCases
    customer --> customerCases
    driver --> driverCases
```

Roles: Admin, CSKH, Customer, Driver. Flow: App/Call/Email → case → statement → evidence → CSKH review → send result → CLOSED.

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
    app_users ||--o{ notifications : user
    cases ||--o{ sla : case

    app_users {
        bigint id PK
        varchar email
        varchar password_hash
        varchar full_name
        varchar role
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
        bigint income_id FK
        bigint expense_id FK
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
    notifications {
        bigint id PK
        bigint user_id FK
        text message
        boolean read
        timestamptz created_at
    }
    sla {
        bigint id PK
        bigint case_id FK
        timestamptz deadline
        varchar status
    }
```

See the [detailed database diagram](docs/DATABASE.md), [data model](docs/05-data-model.md),

### 4.2 OpenAPI 3.0 API documentation




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

### Case lifecycle sequence

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

- Class diagrams: Auth, Case, Driver Statement, Evidence, Import, Reporting, Audit, RAG.
- Sequence diagrams: intake, categorization, handling, driver complaint, evidence extraction, CSKH review, send response, audit.

## Documentation
Documentation lives in the `docs/` folder of the project. See the Green Car repository for full details.

## Project Progress (early stage)
- [x] Define scope, goals, and roles
- [x] Write feature list and use cases
- [ ] Write user stories following INVEST
- [ ] Write acceptance criteria and business rules
- [ ] Design information architecture and screen permissions
- [ ] Design C4 and arc42 documentation
- [ ] Design PostgreSQL schema, DBML, and relationships
- [ ] Write API documentation in OpenAPI 3.0
- [ ] Design React frontend folder structure
- [ ] Design three-tier backend folder structure
- [ ] Design class diagrams and sequence diagrams
- [ ] Create traceability between requirements, screens, API, and database
- [ ] Build the three-tier backend
- [ ] Build authentication, user management, and personal profile API
- [ ] Implement case CRUD API
- [ ] Complete dashboard, reports, import, attachments, and audit API
- [ ] Write unit tests and API integration tests
- [ ] Connect React frontend to the API
- [ ] Implement Excel import with row-level preview and PDF/XLSX report export
- [ ] Record audit events for create, update, delete, login, logout, import, and export
- [ ] Add frontend tests with Vitest
- [ ] Complete PostgreSQL migrations and integration tests against a real database
- [ ] Add durable file storage for imports and attachments
- [ ] Standardize JWT, rate limiting, authorization, and security tests