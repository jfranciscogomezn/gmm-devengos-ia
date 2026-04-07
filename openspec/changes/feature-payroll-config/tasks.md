> **Traceability:** Backend tasks derived from archived epic `openspec/changes/archive/2026-04-06-gmm-devengos-initial/tasks.md` (sections 6, 2.4). Frontend task from section 12.3.

## Backend — Payroll configuration

- [ ] P1.1 Write failing tests for payroll config create/update/read and holiday management
- [ ] P1.2 Implement GET /config/payroll/:year and PUT /config/payroll/:year (admin only; align path prefix with existing `/api/v1` convention)
- [ ] P1.3 Add input validation: required fields, non-negative values, HH:MM time format, non-negative percentages
- [ ] P1.4 Implement GET /config/holidays/:year, POST /config/holidays, DELETE /config/holidays/:id
- [ ] P1.5 Flyway migration(s) for payroll config and holidays tables; JPA entities and repositories
- [ ] P1.6 Pass all payroll config tests

## Frontend — Admin

- [ ] P2.1 Payroll configuration screen: global parameters per selected year + holiday calendar (list/add/delete holidays)
