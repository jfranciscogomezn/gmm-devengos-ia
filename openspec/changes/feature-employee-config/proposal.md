## Why

Employees (workforce master data: personal information and salary) are required before time records can be attributed to a person and earnings calculated. System `User` accounts (Phase 1) remain separate from this domain unless explicitly linked later per requirements.

## What Changes

- Backend: Flyway + JPA for `Employee` (or equivalent) with personal fields and salary; admin CRUD API.
- Frontend: admin employee configuration screens (list, create, update).
- Tests per TDD.

## Product References

- `requirements/requirements-v3.md` — employee configuration.
- Delta spec: `specs/employee-config/spec.md`.

## Builds On

- Phase 1 security (ADMIN-only endpoints).
- Recommended order: implement after or in parallel with **feature-payroll-config** only if requirements demand salary validation against config; otherwise can follow payroll-config.

## Out of Scope

- Time tracking, earnings calculation, reports, user account provisioning for every employee (unless requirements specify linking User ↔ Employee in this change).

## Impact

- New Flyway version(s) after payroll-config migrations if sequential numbering is used; coordinate migration order with team.
