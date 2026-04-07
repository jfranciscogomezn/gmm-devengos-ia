## Why

Global payroll parameters and the holiday calendar must exist before earnings calculation and reporting can be implemented. This change delivers admin-only configuration APIs and UI aligned with product requirements.

## What Changes

- Backend: Flyway migrations and JPA entities for payroll configuration per calendar year and holidays.
- REST endpoints for read/update payroll config and CRUD holidays (paths under `/api/v1` — exact naming to follow existing backend conventions).
- Frontend: admin payroll configuration screen (parameters per year + holiday calendar).
- Automated tests (TDD) for validation rules and endpoints.

## Product References

- `requirements/requirements-v3.md` — payroll configuration, holidays, and parameters used by earnings logic.
- Delta spec: `specs/payroll-config/spec.md`.

## Builds On

- Phase 1 shipped: JWT authentication, `ADMIN` role, users/roles/menu options, Flyway V1–V4, audit logging pattern.
- Stack: Java 17 + Spring Boot 3.x, PostgreSQL, Flyway, React + TypeScript + Vite (see `docs/architecture.md`).

## Out of Scope (This Change)

- Employee master data, time records, earnings engine, Excel reports, mobile app.

## Impact

- New database tables and migrations after V4.
- New admin menu route (already seeded menu options may include `PAYROLL_CONFIG`; wire frontend when implemented).
