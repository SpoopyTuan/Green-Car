# Actors, roles, use cases

> Design status: TARGET V1

Product name: **Green Car**.

## Actors / roles

| Role | Display name in UI |
|---|---|
| `ADMIN` | Administrator |
| `CSKH` | CSKH Agent |
| `CUSTOMER` | Customer |
| `DRIVER` | Driver |

## Role × feature matrix

| Feature | Admin | CSKH | Customer | Driver |
|---|---|---|---|---|
| Dashboard | ✓ | ✓ | ✓ (own) | ✓ (own) |
| View cases | ✓ | ✓ | ✓ (own) | ✓ (own) |
| Submit complaint (App) | ✓ | ✓ | ✓ | |
| Submit complaint (Call) | ✓ | ✓ | | |
| Submit complaint (Email) | system | system | | |
| Driver statement | | | | ✓ |
| Attach evidence | ✓ | ✓ | ✓ | ✓ |
| Import Excel | ✓ | ✓ | | |
| View reports | ✓ | ✓ | | |
| Export reports | ✓ | ✓ | | |
| Audit log | ✓ | ✓ | | |
| User management | ✓ | | | |
| Personal profile | ✓ | ✓ | ✓ | ✓ |

Unauthorized URL access → Forbidden screen.

## Use Case Diagram

```mermaid
flowchart LR
    subgraph left["Actors"]
        direction TB
        admin(["Administrator"])
        cskh(["CSKH"])
    end

    subgraph system["Green Car — System Boundary"]
        direction TB
        subgraph common["Access and overview"]
            direction LR
            uc01["UC01 Login"]
            uc02["UC02 Logout"]
            uc03["UC03 View Dashboard"]
            uc15["UC15 Manage Profile"]
        end
        subgraph cases["Cases"]
            direction LR
            uc04["UC04 View Cases"]
            uc05["UC05 Submit Complaint (App)"]
            uc06["UC06 Submit Complaint (Call)"]
            uc07["UC07 Ingest Email Complaint"]
            uc08["UC08 Driver Statement"]
            uc09["UC09 Attach Evidence"]
        end
        subgraph ops["Operations and governance"]
            direction LR
            uc10["UC10 Import Excel"]
            uc11["UC11 View Reports"]
            uc12["UC12 Export Reports"]
            uc13["UC13 View Audit Log"]
            uc14["UC14 Manage Users"]
            uc16["UC16 Policy RAG"]
            uc17["UC17 SLA & Notifications"]
        end
    end

    subgraph right["Actors"]
        direction TB
        customer(["Customer"])
        driver(["Driver"])
    end

    admin --> uc01 & uc02 & uc03 & uc04 & uc05 & uc06 & uc07 & uc08 & uc09 & uc10 & uc11 & uc12 & uc13 & uc14 & uc15 & uc16 & uc17
    cskh --> uc01 & uc02 & uc03 & uc04 & uc05 & uc06 & uc07 & uc08 & uc09 & uc10 & uc11 & uc12 & uc13 & uc15 & uc16 & uc17
    customer --> uc01 & uc02 & uc03 & uc04 & uc05 & uc09 & uc15
    driver --> uc01 & uc02 & uc03 & uc04 & uc08 & uc09 & uc15

    classDef actor fill:#666,color:#fff,stroke:#333
    classDef usecase fill:#1168bd,color:#fff,stroke:#0b4f9e
    class admin,cskh,customer,driver actor
    class uc01,uc02,uc03,uc04,uc05,uc06,uc07,uc08,uc09,uc10,uc11,uc12,uc13,uc14,uc15,uc16,uc17 usecase
```

## Shared rules and constraints

```mermaid
flowchart LR
    auth["Authenticate Session"]:::shared
    authorize["Check RBAC Permission"]:::shared
    own["Verify Record Ownership"]:::rule
    validate["Validate Input"]:::shared
    audit["Write Audit Log"]:::shared

    write["Add / Edit Case"] --> auth
    write --> authorize
    write --> validate

    employeeEdit["Employee Edits Record"] --> own
    employeeEdit --> write

    deleteUc["Soft-delete Case"] --> auth
    deleteUc --> authorize
    deleteUc --> audit
    importUc["Import Excel"] --> validate
    importUc --> audit

    classDef shared fill:#1168bd,color:#fff,stroke:#0b4884
    classDef rule fill:#e8f1fc,color:#123,stroke:#3a7bd5
```

Backend enforces RBAC; menus/buttons hidden when lacking permission.

## Use cases

### UC01 Login
- **Actor:** all roles
- **Preconditions:** an active account exists
- **Main flow:** enter credentials → submit login → backend verifies → return session → Dashboard
- **Permission:** public
- **Result:** authenticated session

### UC02 Logout
- **Actor:** all roles
- **Main flow:** invoke logout → clear client auth state → login
- **Permission:** authenticated
- **Result:** client session cleared

### UC03 View Dashboard
- **Actor:** all roles
- **Main flow:** choose date range → KPIs + charts + recent cases
- **Permission:** dashboard
- **Result:** dashboard dataset

### UC04 View cases
- **Actor:** all roles
- **Main flow:** list active cases + filters + detail
- **Permission:** caseRead
- **Result:** case list

### UC05 Submit complaint (App)
- **Actor:** Customer, CSKH, Admin
- **Main flow:** form → validation → create case with source=APP → audit
- **Permission:** caseCreate
- **Result:** case created

### UC06 Submit complaint (Call)
- **Actor:** CSKH, Admin
- **Main flow:** CSKH enters call details → create case with source=CALL → audit
- **Permission:** caseCreate
- **Result:** case created

### UC07 Ingest email complaint
- **Actor:** system (IMAP poll)
- **Main flow:** poll IMAP → extract → find trip → run agent → create case with source=EMAIL
- **Permission:** system
- **Result:** case created

### UC08 Driver statement
- **Actor:** Driver
- **Main flow:** submit statement + evidence for a case
- **Permission:** statementCreate
- **Result:** statement recorded

### UC09 Attach evidence
- **Actor:** all roles (own case)
- **Main flow:** upload file → link to case → audit
- **Permission:** evidenceUpload
- **Result:** evidence linked to case

### UC10 Import Excel
- **Actor:** Admin, CSKH
- **Main flow:** choose file → validate/preview → atomic import → summary
- **Permission:** importData
- **Result:** imported rows or validation errors

### UC11 View reports
- **Actor:** Admin, CSKH
- **Main flow:** filters by time/category → aggregates
- **Permission:** reportRead
- **Result:** report dataset

### UC12 Export reports
- **Actor:** Admin, CSKH
- **Main flow:** export PDF/XLSX using current filters
- **Permission:** reportRead
- **Result:** exported file

### UC13 View audit log
- **Actor:** Admin, CSKH
- **Main flow:** view audit events
- **Permission:** auditRead
- **Result:** audit list

### UC14 Manage users
- **Actor:** Admin
- **Main flow:** list, add/edit, enable/disable users
- **Permission:** userManagement
- **Result:** user list updated

### UC15 Personal profile
- **Actor:** all roles
- **Main flow:** update profile fields, change password with current password verification
- **Permission:** authenticated
- **Result:** profile updated

### UC16 Policy RAG
- **Actor:** system
- **Main flow:** retrieve relevant policies for a case
- **Permission:** system
- **Result:** relevant policy chunks

### UC17 SLA & notifications
- **Actor:** system/CSKH
- **Main flow:** monitor SLA deadlines, send notifications
- **Permission:** system/cskh
- **Result:** notifications sent / SLA tracked
