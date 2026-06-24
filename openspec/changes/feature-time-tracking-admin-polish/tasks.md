> **Traceability:** Closes deferred task T2.1 from `openspec/changes/archive/2026-06-03-feature-time-tracking/tasks.md`. Product reference: `requirements/requirements-v5.md` Módulo 6 and Módulo 8. Frontend-only; no backend changes.

## Frontend

- [ ] A1.1 Add `showEarnings` and `uncapped` state to `AdminTimeRecordsPage`; render "Show earnings" toggle and "Uncapped view" sub-toggle in the filter row (Records tab only, disabled when no employee selected)
- [ ] A1.2 Add `earningsQuery` (React Query): calls `reportsService.getCappedReport` or `getUncappedReport` based on `uncapped` flag; enabled only when `showEarnings && selectedEmployeeId !== null && activeTab === 'records'`
- [ ] A1.3 When `showEarnings` is true, replace the plain `TimeRecord` table in Records tab with `TimeReportTable` (pass `uncapped` prop); join report records by `workDate` — the earnings response already contains the matching records
- [ ] A1.4 Handle loading/error states for `earningsQuery` independently of the existing records query (earnings columns show `—` while loading)
- [ ] A1.5 Add translation keys `time:admin.showEarnings` and `time:admin.uncappedView` to `en-US/time.json` and `es-CO/time.json`

## Tests

- [ ] A2.1 Vitest: toggling `showEarnings` to `true` enables the earnings query and renders `TimeReportTable`; toggling back disables it and reverts to the plain table
- [ ] A2.2 Vitest: `uncapped` toggle switches query key from `getCappedReport` to `getUncappedReport`
- [ ] A2.3 `npm run build` and `npm test` green

## Delivery

- [ ] A3.1 Feature branch `feat/time-admin-earnings-view` on `gmm-devengos-frontend`; PR against `feat/ui-refresh` (or `master` if ui-refresh is merged)
- [ ] A3.2 Maintainer approval and merge
