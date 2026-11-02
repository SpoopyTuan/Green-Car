# Business rules — Green Car

## Roles
- Only roles ADMIN, CSKH, CUSTOMER, DRIVER exist.
- ADMIN manages users and all data.
- CSKH reviews cases and approves responses.
- CUSTOMER submits complaints.
- DRIVER submits statements.

## Case lifecycle
- Case statuses: Intake → Categorization → Initial handling → Driver statement → Evidence → CSKH review → Resolution (RESOLVED) → Close (CLOSED).
- Soft-deleted cases excluded from active views.

## Permissions
- Backend enforces RBAC; frontend hiding is UX only.
- Employees/Customers can edit only their own records.
- Only ADMIN and CSKH can soft-delete.

## Import
- Import is all-or-nothing: any invalid row rejects the whole batch.

## Files
- File type and size validated server-side before parsing/storage.

## Policy RAG
- Only ACTIVE policies within validity period with correct embedding model are used.

## AI / HITL
- AI does not send final result without CSKH approval (HITL).
- Every claim must cite source; no policy fabrication.
- Conflicts distinguished as confirmed / conflicting / missing.

## Audit
- All create, update, delete, login, logout, import, export actions are audited.

## SLA
- SLA deadlines tracked per case; notifications sent on breach.
