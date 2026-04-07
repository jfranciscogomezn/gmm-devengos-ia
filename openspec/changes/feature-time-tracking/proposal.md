## Why

Core domain: employees record daily clock-in/clock-out; the system enforces business rules for OPEN/CLOSED/INCOMPLETE states, admin corrections, and auditability. This unlocks later earnings and reporting.

## What Changes

- Backend: `TimeRecord` persistence, clock-in/out endpoints, filters, scheduled job for INCOMPLETE transition, admin resolve-incomplete, admin direct correction (PUT/POST) with mandatory reason and audit trail, incomplete listing endpoint.
- Frontend: employee clock UI, own records view with filters and INCOMPLETE banner; admin screens for time records, reopen, incomplete resolution, correction, admin-created record.
- Tests covering immutability, RBAC, job behavior, and correction flows.

## Product References

- `requirements/requirements-v3.md` — time tracking, INCOMPLETE handling, admin-only manual clock-out, direct correction, audit.
- Delta spec: `specs/time-tracking/spec.md`.

## Builds On

- Phase 1: auth, roles, users, audit_logs.
- **feature-employee-config**: time records must reference an employee (or equivalent identifier per design).
- **feature-payroll-config**: optional for notifications thresholds; not strictly blocking clock-in/out.

## Out of Scope

- Earnings calculation amounts in responses (may return raw times only until **feature-earnings-and-reports**), unless minimal derived fields are required earlier — decide in implementation plan.

## Impact

- New Flyway migrations; possible Quartz/Spring `@Scheduled` for midnight job; email/in-app notification stubs per requirements.
