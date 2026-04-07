## ADDED Requirements

### Requirement: Employee can view their own capped time and earnings on screen
The system SHALL allow an employee to view their own time records on screen with capped hours (max 8 normal + 2 daytime overtime per day) and the corresponding earned value. Normal and overtime hours SHALL be visually distinguishable.

#### Scenario: Employee views capped daily records
- **WHEN** an employee requests their on-screen report for a period
- **THEN** the system returns records with capped hours, clearly marking overtime hours, and the earned value per record

### Requirement: Employee can download a capped Excel report
The system SHALL allow an employee to download an Excel file of their own time records for a selected period, applying the 8h normal + 2h overtime cap per day.

#### Scenario: Excel download - capped
- **WHEN** an employee requests an Excel export for a period
- **THEN** the system generates and streams a .xlsx file with one row per day, showing date, capped worked hours (normal and overtime columns), and earned value

### Requirement: Admin can view capped time and earnings for any employee on screen
The system SHALL allow an admin to view on-screen the capped time records (max 8 normal + 2 daytime overtime) for any specific employee, with visually distinguishable normal and overtime hours.

#### Scenario: Admin views capped records for employee
- **WHEN** an admin selects an employee and a period
- **THEN** the system returns that employee's capped records with hours and earned value

### Requirement: Admin can view total (uncapped) time records for any employee on screen
The system SHALL allow an admin to view on-screen the total registered hours per day for any employee, without applying the cap.

#### Scenario: Admin views uncapped records
- **WHEN** an admin requests the uncapped view for an employee and period
- **THEN** the system returns the total recorded hours per day as entered, alongside the full earned value

### Requirement: Admin can download a capped Excel report for a specific employee
The system SHALL allow an admin to download a capped Excel report (8h normal + 2h overtime cap) for any employee.

#### Scenario: Admin downloads capped Excel
- **WHEN** an admin requests a capped Excel export for a specific employee and period
- **THEN** the system generates and streams a .xlsx file with capped hours (normal and overtime columns visible) and earned value per day

### Requirement: Admin can download a total (uncapped) Excel report for a specific employee
The system SHALL allow an admin to download an uncapped Excel report showing total registered hours and full earned value for any employee.

#### Scenario: Admin downloads uncapped Excel
- **WHEN** an admin requests an uncapped Excel export for a specific employee and period
- **THEN** the system generates and streams a .xlsx file with total daily hours and earned value without applying any cap

### Requirement: Reports are filterable by day, week, month, and custom date range
All time and earnings reports (on-screen and Excel) SHALL support filtering by a specific day, a specific calendar week, a specific calendar month, or a free-form custom date range defined by a start date and an end date.

#### Scenario: Day filter
- **WHEN** a user requests a report with a day filter
- **THEN** the system returns or exports data for that single date only

#### Scenario: Week filter
- **WHEN** a user requests a report with a week filter
- **THEN** the system returns or exports data for all days within that calendar week

#### Scenario: Month filter
- **WHEN** a user requests a report with a month filter
- **THEN** the system returns or exports data for all days within that calendar month

### Requirement: Reports support a free-form custom date range filter (start date / end date)
All time and earnings reports (on-screen and Excel) SHALL support a custom date range filter accepting a `startDate` and an `endDate` in ISO-8601 format (YYYY-MM-DD), enabling payroll summaries for arbitrary periods such as biweekly pay periods or any non-standard interval.

#### Scenario: Custom date range — valid range
- **WHEN** a user requests a report with `startDate` and `endDate` where `startDate` ≤ `endDate`
- **THEN** the system returns or exports data for every day within `[startDate, endDate]` inclusive

#### Scenario: Custom date range — invalid range (start after end)
- **WHEN** a user requests a report with `startDate` > `endDate`
- **THEN** the system rejects the request with HTTP 400 and a descriptive validation error

#### Scenario: Custom date range — span exceeds limit
- **WHEN** a user requests a report with a date range spanning more than 366 days
- **THEN** the system rejects the request with HTTP 400

#### Scenario: Mutually exclusive filter modes
- **WHEN** a user sends a request with more than one filter mode active (e.g., both `week` and a custom date range)
- **THEN** the system rejects the request with HTTP 400 indicating that exactly one filter mode must be provided

#### Scenario: Employee applies custom date range to own records
- **WHEN** an authenticated employee requests a custom date range report
- **THEN** the system returns only that employee's own records within the specified range

#### Scenario: Admin applies custom date range to any employee
- **WHEN** an admin requests a custom date range report for a specific employee
- **THEN** the system returns that employee's records within the specified range

### Requirement: Report generation is blocked when the requested period contains INCOMPLETE records
All time and earnings reports (on-screen and Excel, capped and uncapped) SHALL be blocked from generation if the requested period contains one or more time records in INCOMPLETE status. The system SHALL inform the user which dates are incomplete and require admin resolution before the report can be produced.

#### Scenario: On-screen report blocked by incomplete record
- **WHEN** a user requests an on-screen report for a period that contains at least one INCOMPLETE record for the subject employee
- **THEN** the system returns HTTP 409 with an error body listing the incomplete dates; no report data is returned

#### Scenario: Excel export blocked by incomplete record
- **WHEN** a user requests an Excel export for a period that contains at least one INCOMPLETE record for the subject employee
- **THEN** the system returns HTTP 409 with an error body listing the incomplete dates; no file is generated

#### Scenario: Block applies to both capped and uncapped variants
- **WHEN** an INCOMPLETE record exists within the requested period
- **THEN** both Report A (capped) and Report B (uncapped) are blocked equally

#### Scenario: Block applies to admin reports for any employee
- **WHEN** an admin requests a report for a specific employee whose period contains INCOMPLETE records
- **THEN** the system blocks the report and lists the incomplete dates; the admin must resolve them before the report can be generated

#### Scenario: Report succeeds once all incomplete records are resolved
- **WHEN** all INCOMPLETE records within the requested period have been resolved by the admin (manual clock-out set)
- **THEN** the system allows the report to be generated and includes those now-closed records in the totals

#### Scenario: Frontend guidance for blocked report
- **WHEN** the frontend receives a 409 response due to incomplete records
- **THEN** it displays the list of incomplete dates with a clear message directing the employee to contact the admin, or directing the admin to the correction workflow

### Requirement: On-screen reports visually highlight records with extended hours
All on-screen time reports (Report A and Report B) SHALL visually distinguish records where the employee's billable hours exceed defined thresholds derived from the payroll configuration, helping users quickly identify unusual or potentially erroneous entries.

Two highlight levels SHALL be applied per record row:
- **Warning level** (yellow / amber): billable hours exceed the configured normal daily work hours (overtime was worked).
- **Alert level** (red / orange): billable hours exceed the configured normal daily work hours plus the configured maximum daily extra hours (unusually long day — possible data entry error).

Corrected records (modified by an admin) SHALL always display a distinct correction indicator regardless of hours.

#### Scenario: Record within normal hours — no highlight
- **WHEN** an employee's billable hours for a day are less than or equal to the configured daily normal hours
- **THEN** the report row is rendered without any special highlight

#### Scenario: Record with overtime — warning highlight
- **WHEN** an employee's billable hours exceed the configured daily normal hours but do not exceed normal hours plus maximum daily extra hours
- **THEN** the report row is rendered with a warning-level highlight (e.g., amber background or icon)

#### Scenario: Record with excessive hours — alert highlight
- **WHEN** an employee's billable hours exceed the configured daily normal hours plus the configured maximum daily extra hours
- **THEN** the report row is rendered with an alert-level highlight (e.g., red background or icon) to flag a potentially incorrect entry

#### Scenario: Highlight applied in both capped and uncapped views
- **WHEN** a record triggers a highlight threshold
- **THEN** the highlight is shown in both Report A (capped) and Report B (uncapped) on-screen views; in Report A, the highlight is based on the raw billable hours, not the capped value

#### Scenario: Admin-corrected record indicator
- **WHEN** a time record carries the admin-correction flag
- **THEN** the report row displays a visible correction indicator (icon or label) in addition to any hours-based highlight; hovering or tapping the indicator shows the correction reason and admin name

#### Scenario: Excel export does not apply visual highlights but includes correction flag column
- **WHEN** an Excel report is generated and contains corrected or extended-hours records
- **THEN** the .xlsx file includes a "Notes" column where corrected records show "Admin correction: [reason]" and records exceeding the alert threshold show "Extended hours"
