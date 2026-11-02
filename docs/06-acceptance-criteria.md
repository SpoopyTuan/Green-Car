# Acceptance criteria — Green Car

## UC01 Login
- Valid credentials return session; invalid show error.
- Inactive account cannot log in.

## UC02 Logout
- Logout clears client session and returns to login.

## UC03 View Dashboard
- Dashboard shows KPIs, status chart, recent cases, SLA indicators.
- Date range filter works.

## UC04 View cases
- Active cases listed; soft-deleted excluded.
- Filters by status, severity, channel.

## UC05 Submit complaint (App)
- Customer submits complaint with trip context; case created with source=APP.

## UC06 Submit complaint (Call)
- CSKH enters call details; case created with source=CALL.

## UC07 Ingest email complaint
- IMAP poll extracts email, finds trip, runs agent, creates case source=EMAIL.

## UC08 Driver statement
- Driver submits statement and evidence; linked to case.

## UC09 Attach evidence
- File upload links to case; metadata persisted.

## UC10 Import Excel
- All rows valid → import; any invalid row → reject all (atomic).

## UC11 View reports
- Reports filter by time and category.

## UC12 Export reports
- Export PDF/XLSX using current filters.

## UC13 View audit log
- Audit events viewable by Admin/CSKH with filters.

## UC14 Manage users
- Admin creates, edits, enables, disables users.

## UC15 Personal profile
- Update profile fields; password change requires current password.

## UC16 Policy RAG
- Retrieve relevant policies for a case.

## UC17 SLA & notifications
- Monitor SLA deadlines; send notifications.
