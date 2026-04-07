## Context

**Employee** entity holds personal data (names, ID type/number, email, phone, etc.) and **salary** fields as defined in requirements. Admin-only REST API following existing package layout (`controller`, `service`, `repository`, `domain/model`).

## Data Model

- Table `employees` (snake_case columns); indexes on commonly filtered columns (e.g. document number, email if unique per business rules).
- No duplication of `users` table; document in OpenAPI if a future link to `User` is planned.

## API

- `POST /api/v1/employees`, `GET /api/v1/employees`, `GET /api/v1/employees/{id}`, `PUT /api/v1/employees/{id}` — all `ADMIN` unless requirements specify otherwise.

## Validation

- Bean Validation on DTOs; salary positive where required; align with `specs/employee-config/spec.md`.

## Frontend

- `EmployeeListPage`, `EmployeeFormPage` (or equivalent names) under admin routes; React Query mutations.

## Testing

- UserService-style unit tests + `MockMvc` or `@SpringBootTest` integration tests; frontend tests per project conventions.

## Dependencies

- Phase 1 auth/RBAC.
- Payroll config optional for cross-validation (e.g. minimum wage); implement only if required by spec.
