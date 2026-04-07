> **Traceability:** Derived from archived epic `openspec/changes/archive/2026-04-06-gmm-devengos-initial/tasks.md` (sections 9, 10, 12.6, 13.2–13.3).

## Backend — Earnings calculation

- [ ] R1.1 Unit tests for earnings calculation (time ranges, rest deduction, Sunday/holiday surcharge, cap logic)
- [ ] R1.2 Implement EarningsCalculationService: hourly rate from salary and config
- [ ] R1.3 Time-range classification (normal, daytime OT, night surcharge, nocturnal OT)
- [ ] R1.4 Rest-time deduction before classification
- [ ] R1.5 Sunday/holiday detection and surcharge application
- [ ] R1.6 Capped view (8h normal + 2h daytime OT limit)
- [ ] R1.7 Integrate earnings into time record / report DTOs as designed
- [ ] R1.8 Pass all earnings calculation tests

## Backend — Time reports

- [ ] R2.1 Tests for on-screen report endpoints (capped/uncapped) and Excel generation
- [ ] R2.2 GET /reports/time (on-screen capped; employee own / admin any; day/week/month filter)
- [ ] R2.3 GET /reports/time/uncapped (admin only)
- [ ] R2.4 GET /reports/time/export?cap=true|false (stream .xlsx)
- [ ] R2.5 Custom date range (startDate/endDate) on all report endpoints; validation; exclusivity with day/week/month
- [ ] R2.6 INCOMPLETE guard → HTTP 409 + incomplete dates list (on-screen + Excel, capped + uncapped)
- [ ] R2.7 On-screen response: highlightLevel per record; corrected flag + correctionReason
- [ ] R2.8 Excel Notes column: admin correction + extended hours text per spec
- [ ] R2.9 Tests: extended-hours levels, corrected flag in JSON and Excel, incomplete blocking, custom range
- [ ] R2.10 Pass all reports tests

## Frontend

- [ ] R3.1 Admin Excel export UI (capped and uncapped per employee)
- [ ] R3.2 Employee Excel export (own capped report with Notes column)
- [ ] R3.3 Employee time records screen: extended-hours warning/alert + correction indicator (if not completed in feature-time-tracking)
