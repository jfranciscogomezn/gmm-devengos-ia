## Context

Implements the **payroll-config** capability on the existing Spring Boot + Flyway stack. Follow DDD-style layering: controller → service → repository → entities; MapStruct for DTOs; `@PreAuthorize("hasRole('ADMIN')")` on mutating endpoints.

## Data Model (To Add)

- **Payroll configuration** keyed by **calendar year** (or equivalent): minimum wage, transport subsidy, monthly work hours, normal daily hours, max daily extra hours, time-range boundaries (normal / daytime OT / night / nocturnal OT), surcharge percentages, non-billable rest minutes. Align field names and validation with `requirements-v3.md` and `specs/payroll-config/spec.md`.
- **Holiday**: date, optional description, link to year or standalone date; support list/delete for admin.

## API (Illustrative)

- `GET/PUT /api/v1/config/payroll/{year}` (or project-standard path) — admin only.
- `GET /api/v1/config/holidays?year=`, `POST`, `DELETE /api/v1/config/holidays/{id}` — admin only.

## Validation

- Non-negative monetary and hour fields; HH:MM or configured format for time boundaries; percentages within allowed range; reject incomplete payloads with HTTP 400 and consistent error JSON.

## Frontend

- Single admin area: year selector, form for all parameters, holiday list with add/remove; React Query + existing Axios client pattern; reuse React Bootstrap.

## Testing

- Unit tests for service validation; integration tests with Testcontainers for controllers; frontend tests as per project frontend standards.

## Dependencies

- None beyond Phase 1 security and infrastructure.
