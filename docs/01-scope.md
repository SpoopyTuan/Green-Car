# Scope V1

> Design status: TARGET V1
> 
> Implementation status: CURRENT PROTOTYPE ONLY

Product name in the interface: **Green Car**.

## Problem

A transport company needs a simple place to:

- record customer complaints;
- collect driver statements and evidence;
- let CSKH review and respond;
- track SLA and audit actions.

This is not an ERP, not a full accounting system, and does not manage inventory or fleet operations.

Complaint records: channel (App/Call/Email), customer, trip, driver, category, severity, status, evidence, statements, policy references.

Users: full name, email, phone, role, active/disabled status.

## Target V1 goals

The target V1 supports:

- Login through the REST API with backend authentication and authorization.
- Dashboard (KPIs, case status, recent cases).
- Case management (list, detail, status tracking).
- Complaint intake via App, Call, and Email (IMAP poll).
- Driver statements and evidence attachments.
- Reports & analytics.
- Excel data import with validation, preview, and persisted batch results.
- Attachment upload and persisted metadata.
- Persisted activity/audit log.
- User management (Admin).
- Profile: account, password change, and permission matrix.
- Backend-enforced roles and permissions.
- Policy RAG and SLA monitoring.
- PostgreSQL persistence aligned with the approved API contract.

## Non-goals

V1 does **not** include:

- Inventory or fleet management.
- Payroll or HR.
- Payment gateways.
- Multi-factor authentication or external identity providers.
- Direct integrations with third-party accounting or marketplace APIs.

## Current prototype

Component | Reality
--- | ---
Frontend | MOCK DATA. No API fetch, no SQL.
Database | Schema designed. Not connected to the app.
Auth | Mock authentication.
Excel Import | UI + mock flow. File contents not read.
Attachments | Select local file, store name/type/size. No upload.

No executable backend baseline exists yet. Frontend integration, production-grade import, attachment, audit, and operational hardening remain to be built.

## Target V1 architecture

```
React frontend
    → REST/HTTPS/JSON
FastAPI backend
    → PostgreSQL + pgvector
    → file storage for imports and attachments
    → Gemini/LangGraph for AI agent
```

The current mock frontend is intentional and remains available for UI development and demonstration. It is not the target authentication, authorization, persistence, import, or attachment architecture.
