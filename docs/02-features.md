# Feature Catalog

> Design status: TARGET V1 with CURRENT PROTOTYPE status recorded separately

The status column describes the current prototype. Target V1 behavior is defined by the use cases, acceptance criteria, OpenAPI contract, and architecture documents.

UI product: **Green Car**.

| ID | Name | V1 | Status |
|---|---|---|---|
| F01 | Authentication | Yes | Target |
| F02 | Dashboard | Yes | Target |
| F03 | Case Management | Yes | Target |
| F04 | Complaint Intake (App/Call/Email) | Yes | Target |
| F05 | Driver Statements | Yes | Target |
| F06 | Evidence & Attachments | Yes | Target |
| F07 | Reports | Yes | Target |
| F08 | Excel Import | Yes | Target |
| F09 | Audit Log | Yes | Target |
| F10 | User Management | Yes | Target |
| F11 | Profile | Yes | Target |
| F12 | Policy RAG | Yes | Target |
| F13 | SLA & Notifications | Yes | Target |

## F01 Authentication
**Current prototype:** mock login with browser storage. **Target V1:** REST login returns Bearer JWT; backend enforces auth. Logout clears client token; V1 has no refresh token.

## F02 Dashboard
Case KPIs, status chart, recent cases, SLA indicators.

## F03 Case Management
List active cases, filters, pagination, soft delete, detail modal. Fields: channel, customer, trip, driver, category, severity, status, created_at.

## F04 Complaint Intake
- App: customer submits complaint with trip context.
- Call: CSKH enters call details.
- Email: IMAP poll → extract → find trip → run agent → create case.

## F05 Driver Statements
Driver submits statement and evidence for a case.

## F06 Evidence & Attachments
Upload files, link to case, persisted metadata.

## F07 Reports
Overview, by time, by category. Export PDF/XLSX.

## F08 Excel Import
Import case data from .xlsx/.xls with validation and preview.

## F09 Audit Log
Log create, update, soft delete, import, login events.

## F10 User Management
Admin only: list, add/edit, enable/disable users.

## F11 Profile
Account tab, security (password change), roles & permissions (read-only role).

## F12 Policy RAG
Retrieve relevant policies for a case using BM25/dense/hybrid.

## F13 SLA & Notifications
Monitor SLA deadlines, send notifications.
