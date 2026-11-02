# Data Diagram

> Design status: TARGET V1
>
> Runtime status: NOT CONNECTED

PostgreSQL schema `green_car` for **Green Car**.

The interface uses **mock data** aligned with this model (tables, complaint cases, driver statements, evidence, attachments, soft delete, currency). The interface **does not connect** to PostgreSQL.

Design files: [05-data-model.md](05-data-model.md)

## Relationships (tables)

`app_users` is on the left. `cases` is central. `driver_statements`, `evidence`, `attachments`, and `sla` branch from `cases`. `import_batches` feeds `cases`. `policy_versions` feeds `policy_chunks`. `app_users` feeds `audit_logs` and `notifications`.

```mermaid
%%{init: {"flowchart": {"nodeSpacing": 30, "rankSpacing": 70, "padding": 12}}}%%
flowchart LR
  U[app_users]

  subgraph groups [" "]
    direction TB

    subgraph cases [Cases]
      direction LR
      CS[cases]
      DS[driver_statements]
      EV[evidence]
      AT[attachments]
      SL[sla]
    end

    subgraph shared [Shared]
      direction LR
      IB[import_batches]
      AL[audit_logs]
      NT[notifications]
    end

    subgraph policy [Policy RAG]
      direction LR
      PV[policy_versions]
      PC[policy_chunks]
    end
  end

  U --> cases
  cases --> driver_statements
  cases --> evidence
  cases --> attachments
  cases --> sla
  import_batches --> cases
  policy_versions --> policy_chunks
  U --> audit_logs
  U --> notifications
```

## Enums

| Enum | Values | UI |
|---|---|---|
| `user_role` | ADMIN, CSKH, CUSTOMER, DRIVER | Administrator, CSKH, Customer, Driver |
| `case_source` | APP, CALL, EMAIL | App, Call, Email |
| `case_status` | Intake, Categorization, In Progress, Resolved, Closed | Case status |
| `case_severity` | Low, Medium, High, Critical | Severity |
| `import_status` | PENDING, PROCESSING, COMPLETED, FAILED | Import-batch status |
| `audit_action` | CREATE, UPDATE, SOFT_DELETE, RESTORE, LOGIN, EXPORT, IMPORT | Audit log action |

## Views (report aggregates)

Views are **not stored tables**. They read active case records and aggregate by day / month / category / severity. In V1, `currency_code` is always `USD`; no foreign currency conversion.

```mermaid
flowchart LR
  C[cases]
  C --> V1[vw_case_active]
  C --> V2[vw_case_daily]
  C --> V3[vw_case_monthly]
  C --> V4[vw_case_by_severity]
  C --> V5[vw_case_by_status]
  C --> V6[vw_sla_breach]
```

## Detailed ER diagram (main columns)

```mermaid
erDiagram
  app_users {
    bigint id PK
    varchar email
    varchar password_hash
    varchar full_name
    varchar phone
    user_role role
    boolean is_active
    timestamptz created_at
    timestamptz updated_at
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
    timestamptz updated_at
    timestamptz deleted_at
  }

  driver_statements {
    bigint id PK
    bigint case_id FK
    bigint driver_id FK
    text statement
    timestamptz created_at
    timestamptz updated_at
  }

  evidence {
    bigint id PK
    bigint case_id FK
    bigint uploaded_by FK
    varchar file_path
    varchar original_name
    bigint file_size
    varchar mime_type
    timestamptz created_at
  }

  attachments {
    bigint id PK
    bigint case_id FK
    bigint income_id FK
    bigint expense_id FK
    varchar original_name
    bigint file_size
    varchar mime_type
    bigint uploaded_by FK
    timestamptz created_at
  }

  import_batches {
    bigint id PK
    varchar import_type
    varchar status
    varchar original_file_name
    bigint row_count
    bigint imported_by FK
    timestamptz created_at
    timestamptz updated_at
  }

  audit_logs {
    bigint id PK
    bigint actor_user_id FK
    varchar action
    varchar module
    varchar table_name
    text detail_before
    text detail_after
    varchar request_id
    timestamptz created_at
  }

  policy_versions {
    bigint id PK
    varchar name
    varchar status
    timestamptz effective_from
    timestamptz effective_to
  }

  policy_chunks {
    bigint id PK
    bigint policy_version_id FK
    text content
    integer chunk_index
    vector embedding
    timestamptz created_at
  }

  notifications {
    bigint id PK
    bigint user_id FK
    text message
    varchar channel
    boolean read
    timestamptz read_at
    timestamptz created_at
  }

  sla {
    bigint id PK
    bigint case_id FK
    timestamptz deadline
    varchar status
    timestamptz breached_at
    timestamptz created_at
    timestamptz updated_at
  }

  app_users ||--o{ cases : created_by
  app_users ||--o{ driver_statements : driver
  app_users ||--o{ evidence : uploaded_by
  app_users ||--o{ attachments : uploaded_by
  app_users ||--o{ import_batches : imported_by
  app_users ||--o{ audit_logs : actor
  app_users ||--o{ notifications : user

  cases ||--o{ driver_statements : case
  cases ||--o{ evidence : case
  cases ||--o{ attachments : case
  cases ||--o{ sla : case

  import_batches ||--o{ cases : batch

  policy_versions ||--o{ policy_chunks : version
```

Each attachment references exactly one parent through the SQL check constraint, while one case may own zero or many attachment rows.

## Audit security constraint

Audit data must never contain passwords, `password_hash`, JWTs, refresh tokens, or secrets. The current generic audit trigger serializes whole rows and therefore requires redaction for `app_users` before database implementation.

**Status:** REQUIRED BEFORE DATABASE IMPLEMENTATION. Audit records must include actor, entity type/id, timestamp, redacted before/after, and request/correlation ID when available. Soft-delete transitions map to `SOFT_DELETE`, and the inverse maps to `RESTORE`.
