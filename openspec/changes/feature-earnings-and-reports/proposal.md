## Why

Devengado calculation from worked hours and configured ranges is the business core; reporting (on-screen + Excel) delivers capped/uncapped views, custom date ranges, INCOMPLETE guards, and extended-hours highlighting per requirements.

## What Changes

- Backend: `EarningsCalculationService` (pure logic + integration), report endpoints, Excel export stream, HTTP 409 when period contains INCOMPLETE records, `highlightLevel` and correction metadata in JSON responses, Notes column rules in Excel.
- Frontend: wire employee/admin report views, Excel download buttons, display of warning/alert highlights and correction indicators (complete employee time screen if deferred from time-tracking).
- Comprehensive tests for calculation edge cases, report guards, and export content.

## Product References

- `requirements/requirements-v3.md` — earnings, reports, custom date range, INCOMPLETE blocking, extended hours, Excel.
- Delta specs: `specs/earnings-calculation/spec.md`, `specs/time-reports/spec.md`.

## Builds On

- **feature-payroll-config** (parameters and holidays).
- **feature-employee-config** (salary for hourly rate).
- **feature-time-tracking** (time records and statuses).

## Out of Scope

- New auth or role model changes; mobile app.

## Impact

- Heavy test surface for calculation; consider performance for large exports; library for Excel (e.g. Apache POI) on Java backend.
