## Context

**Earnings**: classify worked intervals using payroll config (normal, daytime OT, night, nocturnal OT), apply rest deduction, Sunday/holiday surcharges, and capped view (8h normal + 2h daytime OT cap) for standard displays.

**Reports**: REST JSON for UI; separate uncapped admin endpoint; `GET .../export` streaming `.xlsx` with `cap=true|false`.

## Calculation Module

- Stateless service methods; unit-tested without Spring where possible; inputs: time record(s), employee salary, payroll config for year, holiday set.
- Integrate computed amounts into time-record DTOs or report aggregates as per API design.

## Report API

- Filters: day | week | month | **custom** `startDate`/`endDate` (mutually exclusive, max span e.g. 366 days).
- **409 Conflict** with body listing incomplete dates if any INCOMPLETE in range.
- Response fields: `highlightLevel` (`none` | `warning` | `alert`) from config thresholds; `corrected`, `correctionReason` when applicable.

## Excel

- Notes column: admin correction text; extended hours marker for alert-level rows.

## Frontend

- Admin: uncapped toggle, export capped/uncapped, employee picker.
- Employee: own capped view, export; complete highlight UI on time records list if not finished in time-tracking change.

## Testing

- Parameterized tests for time-range classification; golden cases for caps; integration tests for 409 guard; snapshot or file-based checks for Excel content where practical.

## Dependencies

- All prior feature migrations applied; integration test database includes sample config, employee, and time records.
