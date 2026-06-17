## Context

`AdminTimeRecordsPage` renders a tab-based view with two tabs: **Records** and **Incomplete**. The Records tab shows raw `TimeRecord` rows (date, clock-in, clock-out, duration, status) fetched from `timeService.getByEmployee`. Earnings data lives in the reports endpoints, which return `TimeReportRecord` objects including `cappedMinutes`, `uncappedEarnings`, `cappedEarnings`, and `highlightLevel`.

The goal is to merge both data sources in the Records tab and surface a capped/uncapped toggle for the admin.

## Data Flow

### Existing (unchanged)
```
employee + from/to → timeService.getByEmployee() → TimeRecord[]
                     GET /api/v1/time-records?employeeId=&from=&to=
```

### Added
```
employee + from/to → reportsService.getCappedReport()   → TimeReportResponse   (default)
                  → reportsService.getUncappedReport()  → TimeReportResponse   (toggle on)
                     GET /api/v1/reports/time[/uncapped]?employeeId=&startDate=&endDate=
```

The two datasets are joined in the component by `workDate` (ISO date string). If earnings data is unavailable for a record (e.g. record is OPEN or INCOMPLETE), the earnings columns show `—`.

## Component Changes

### `AdminTimeRecordsPage`

1. **New state:** `showEarnings: boolean` (default `false`) and `uncapped: boolean` (default `false`, admin-only, only visible when `showEarnings === true`).

2. **New queries (React Query):**
   ```ts
   // Only fires when showEarnings && selectedEmployeeId !== null && activeTab === 'records'
   const earningsQuery = useQuery({
     queryKey: ['reports', 'time', uncapped, selectedEmployeeId, fromDate, toDate],
     queryFn: () => uncapped
       ? reportsService.getUncappedReport({ employeeId, startDate: fromDate, endDate: toDate })
       : reportsService.getCappedReport({ employeeId, startDate: fromDate, endDate: toDate }),
     enabled: showEarnings && selectedEmployeeId !== null && activeTab === 'records',
   });
   ```

3. **Table rendering (Records tab):** When `showEarnings` is true, replace the plain `TimeRecord` table with `TimeReportTable` (existing component, `uncapped` prop forwarded). Join by `workDate` — the report already contains the matching records. When earnings data is still loading, show a spinner overlay on the earnings columns only (records render immediately from the existing query).

4. **UI additions:**
   - A `Form.Switch` labelled "Show earnings" in the filter row (Records tab only).
   - When toggled on, a second `Form.Switch` labelled "Uncapped view" appears (reuses the same pattern from `ReportsPage`).
   - Both switches are disabled when no employee is selected.

## API Params Mapping

| `timeService.getByEmployee` param | `reportsService` param |
|-----------------------------------|------------------------|
| `from` (ISO date)                 | `startDate`            |
| `to` (ISO date)                   | `endDate`              |
| `employeeId`                      | `employeeId`           |

Note: the reports API ignores the `date`/`week`/`month` shortcut params when `startDate`/`endDate` are provided.

## No Backend Changes

All required endpoints are already implemented:
- `GET /reports/time` — capped, admin can pass `employeeId`
- `GET /reports/time/uncapped` — uncapped, admin only

## Translation Keys Needed

```json
// time namespace additions
"admin": {
  "showEarnings": "Show earnings",
  "uncappedView": "Uncapped view"
}
```

## Testing

- Unit/component test: toggle `showEarnings` on → earnings query fires; off → query disabled.
- Unit test: capped vs uncapped toggle switches query key and endpoint.
- Build and `npm test` green.
