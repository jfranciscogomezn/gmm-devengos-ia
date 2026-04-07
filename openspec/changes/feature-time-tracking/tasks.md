> **Traceability:** Derived from archived epic `openspec/changes/archive/2026-04-06-gmm-devengos-initial/tasks.md` (sections 8, 2.5, 12.5–12.10, 13.1–13.4).

## Backend — Time tracking

- [ ] T1.1 Write failing tests for clock-in, clock-out, immutability, admin reopen, and list/filter
- [ ] T1.2 Implement POST /time-records/clock-in (once per day enforcement)
- [ ] T1.3 Implement POST /time-records/clock-out (requires open record; closes it)
- [ ] T1.4 Implement PATCH /time-records/:id/reopen (admin only)
- [ ] T1.5 Implement GET /time-records (filter by employeeId, date/week/month/custom date range; employee sees own, admin sees any)
- [ ] T1.6 Implement scheduled job (00:01 server time): OPEN records from prior days → INCOMPLETE; notify admins (in-app + email per requirements)
- [ ] T1.7 Implement PATCH /time-records/:id/resolve-incomplete (admin only): manual clock-out + mandatory note; audit trail
- [ ] T1.8 GET /time-records/incomplete — employee: own; admin: any employee
- [ ] T1.9 PUT /time-records/correct (admin only): employeeId, date, optional clockIn/clockOut, mandatory correctionReason; all statuses; original values; corrected flag; audit
- [ ] T1.10 POST /time-records/correct (admin only): create complete record for date with no entry; mandatory reason + audit
- [ ] T1.11 Tests for admin correction scenarios (both times, partial, new record, 400 cases, audit content, corrected flag)
- [ ] T1.12 Tests for auto-flagging job, employee 403 on resolve, persistent warning data
- [ ] T1.13 Flyway migration for `time_records`; pass all time tracking tests

## Frontend — Admin

- [ ] T2.1 Admin time records screen: any employee; capped view + uncapped toggle; day/week/month/custom date range
- [ ] T2.2 Admin reopen time record UI
- [ ] T2.3 Admin incomplete-record resolution: list all INCOMPLETE; form manual clock-out + mandatory note per record
- [ ] T2.4 Admin direct time correction: employee + date; edit in/out; mandatory reason; show originals; confirm diff
- [ ] T2.5 Admin create record screen for dates with no entry; same reason + confirmation flow

## Frontend — Employee

- [ ] T3.1 Clock-in / clock-out screen (one action per day; show current day status)
- [ ] T3.2 Employee time records: own records; filters; capped view; overtime distinction; placeholders for extended-hours/correction indicators (full styling may align with earnings-reports change)
- [ ] T3.3 Employee dashboard: persistent INCOMPLETE banner with dates + contact admin message until resolved
