## Why

The admin time records screen (`/admin/time`) currently shows raw time data (clock-in, clock-out, duration, status) but does not surface earnings breakdown or the capped/uncapped toggle. This forces admins who want to verify earnings for a specific employee on a specific day to navigate to the separate `/reports` page. The requirement (T2.1 from `feature-time-tracking`) was deferred but it is the last functional gap in the payroll domain before the operations domain begins.

## What Changes

- Frontend only: no backend changes — the existing `GET /reports/time` and `GET /reports/time/uncapped` endpoints already support per-employee, custom date range queries.
- Extend `AdminTimeRecordsPage` to optionally load and display the earnings report alongside the time records list when an employee and date range are selected.
- Add a capped/uncapped toggle matching the `/reports` UI behavior (admin-only).
- Reuse existing `TimeReportTable` component and `reportsService` calls — no new API surface.
- Update translations for any new labels.

## Product References

- `requirements/requirements-v5.md` — Módulo 6 (§6.1.4 admin time records view) and Módulo 8 (§8 reports).
- Traceability: closes deferred task T2.1 from `openspec/changes/archive/2026-06-03-feature-time-tracking/tasks.md`.

## Builds On

- `feature-time-tracking` — `AdminTimeRecordsPage`, `timeService`, `TimeRecord` type.
- `feature-earnings-and-reports` — `reportsService`, `TimeReportTable`, `TimeReportRecord`, `highlightRowClass`.
- Stack: React 18, TypeScript, Vite, TanStack React Query, React Bootstrap (frontend only).

## Out of Scope (This Change)

- Backend endpoints: no new APIs; uses existing `/reports/time` and `/reports/time/uncapped`.
- Employee self-service view (`/my/time`): already shows capped earnings.
- Excel export from the admin time list: available on `/reports`; not duplicated here.
- Period presets (day/week/month): admin screen keeps the existing from/to date range only.
