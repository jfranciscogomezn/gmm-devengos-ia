## Context

State machine: OPEN → CLOSED (employee clock-out); OPEN from prior calendar day → INCOMPLETE at scheduled time (e.g. 00:01); admin operations with audit.

## Data Model

- `time_records`: employee reference, date (or timestamp pair), clock_in, clock_out, status enum, flags for corrected, original values, correction reason; indexes for employee + date queries.

## API (Illustrative)

- `POST .../clock-in`, `POST .../clock-out` — employee (own) or as per spec.
- `GET .../time-records` — filters: employeeId, day/week/month/custom range; RBAC: employee sees own, admin sees any.
- `PATCH .../time-records/{id}/reopen` — admin.
- `PATCH .../time-records/{id}/resolve-incomplete` — admin; body: manual clock-out + mandatory note.
- `GET .../time-records/incomplete` — scoped by role.
- `PUT .../time-records/correct`, `POST .../time-records/correct` — admin; mandatory `correctionReason`; persist before/after in `audit_logs` or dedicated columns.

## Scheduler

- Spring `@Scheduled(cron = ...)` or configurable; timezone awareness; idempotent transitions.

## Frontend

- Employee: single-day clock actions; list view with date filters; persistent INCOMPLETE banner.
- Admin: full list, reopen, incomplete resolution form, correction wizard with confirmation, create record for empty date.

## Testing

- Integration tests for 403 on employee attempting admin resolve; job tests with fixed clock or test doubles; audit content assertions.

## Dependencies

- Employees table must exist; link `time_records.employee_id` with FK.
