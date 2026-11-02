# Data model — Green Car

> Design status: TARGET V1

PostgreSQL 16 + pgvector. 29 tables. Money in USD. Soft delete via `deleted_at`.

## Core tables

### app_users
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| email | varchar | unique |
| password_hash | varchar | PBKDF2 |
| full_name | varchar | |
| role | enum | ADMIN, CSKH, CUSTOMER, DRIVER |
| is_active | boolean | |
| created_at | timestamptz | |
| deleted_at | timestamptz | soft delete |

### cases
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| source | varchar | APP, CALL, EMAIL |
| status | varchar | Intake, Categorization, ... |
| severity | varchar | |
| created_by | bigint FK | app_users |
| import_batch_id | bigint FK | import_batches |
| created_at | timestamptz | |
| deleted_at | timestamptz | |

### driver_statements
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| case_id | bigint FK | cases |
| driver_id | bigint FK | app_users |
| statement | text | |
| created_at | timestamptz | |

### evidence
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| case_id | bigint FK | cases |
| uploaded_by | bigint FK | app_users |
| file_path | varchar | |
| created_at | timestamptz | |

### attachments
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| case_id | bigint FK | cases |
| income_id | bigint FK | legacy column; no incomes table in V1 |
| expense_id | bigint FK | legacy column; no expenses table in V1 |
| uploaded_by | bigint FK | app_users |

### import_batches
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| import_type | varchar | |
| status | varchar | |
| imported_by | bigint FK | app_users |
| created_at | timestamptz | |

### audit_logs
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| actor_user_id | bigint FK | app_users |
| action | varchar | |
| module | varchar | |
| created_at | timestamptz | |

### policy_versions
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | varchar | |
| status | varchar | ACTIVE/INACTIVE |
| effective_from | timestamptz | |

### policy_chunks
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| policy_version_id | bigint FK | policy_versions |
| content | text | |
| embedding | vector | pgvector |

### notifications
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | bigint FK | app_users |
| message | text | |
| read | boolean | |
| created_at | timestamptz | |

### sla
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| case_id | bigint FK | cases |
| deadline | timestamptz | |
| status | varchar | |

## Relationships
- app_users 1..* cases
- cases 1..* driver_statements
- cases 1..* evidence
- cases 1..* attachments
- import_batches 1..* cases
- policy_versions 1..* policy_chunks
- app_users 1..* audit_logs

## ER diagram
See README section 4.1.
