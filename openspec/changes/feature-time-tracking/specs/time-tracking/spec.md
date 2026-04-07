## ADDED Requirements

### Requirement: Employee can clock in once per day
The system SHALL allow an employee to register their clock-in time. An employee SHALL only be able to clock in once per calendar day. Clocking in a second time on the same day SHALL be rejected.

#### Scenario: Successful clock-in
- **WHEN** an employee who has not yet clocked in today submits a clock-in request
- **THEN** the system creates a time record with the current timestamp and returns HTTP 201

#### Scenario: Duplicate clock-in
- **WHEN** an employee who already has a clock-in record for today submits another clock-in
- **THEN** the system returns HTTP 409

### Requirement: Employee can clock out once per day
The system SHALL allow an employee to register their clock-out time for the same day they clocked in. Clock-out SHALL only be possible if a clock-in exists for that day.

#### Scenario: Successful clock-out
- **WHEN** an employee who has clocked in today submits a clock-out request
- **THEN** the system records the clock-out timestamp, marks the record as closed, and returns HTTP 200

#### Scenario: Clock-out without clock-in
- **WHEN** an employee without a clock-in for today submits a clock-out
- **THEN** the system returns HTTP 400

#### Scenario: Duplicate clock-out
- **WHEN** an employee whose record is already closed submits another clock-out
- **THEN** the system returns HTTP 409

### Requirement: Time record is immutable after clock-out
The system SHALL prevent employees from modifying or deleting a time record once it has been closed (clock-out registered), unless an admin has reopened it.

#### Scenario: Edit attempt on closed record
- **WHEN** an employee attempts to modify a closed time record
- **THEN** the system returns HTTP 403

### Requirement: Admin can reopen a closed time record
The system SHALL allow an admin to reopen a closed time record, enabling the employee to correct it (clock-out again) or allowing the admin to update the timestamps directly.

#### Scenario: Admin reopens a record
- **WHEN** an admin submits a reopen request for a closed time record
- **THEN** the system sets the record status back to open and returns HTTP 200

#### Scenario: Reopen already-open record
- **WHEN** an admin attempts to reopen a record that is not closed
- **THEN** the system returns HTTP 400

### Requirement: Open records are auto-flagged as INCOMPLETE at end of day
At 00:01 server time, a scheduled job SHALL detect any time records that are still OPEN from prior calendar days and transition them to INCOMPLETE status. An INCOMPLETE record means the employee never registered their clock-out for that day.

#### Scenario: Job detects unflagged open record
- **WHEN** the 00:01 job runs and finds an OPEN record whose date is prior to today
- **THEN** the system transitions it to INCOMPLETE status

#### Scenario: INCOMPLETE record does not block subsequent attendance
- **WHEN** an employee with one or more INCOMPLETE records from prior days registers clock-in or clock-out for the current day
- **THEN** the system accepts the new attendance record normally; the INCOMPLETE records do not block daily attendance

#### Scenario: Employee receives persistent warning about incomplete records
- **WHEN** an authenticated employee has one or more INCOMPLETE records
- **THEN** the system displays a persistent banner on their dashboard listing the incomplete dates and prompting them to contact the admin for resolution

#### Scenario: Admin notified of incomplete records
- **WHEN** the 00:01 job transitions one or more records to INCOMPLETE
- **THEN** in-app and email notifications are sent to all admins with the list of affected employees and dates

#### Scenario: Admin sets manual clock-out for incomplete record
- **WHEN** an admin submits a manual clock-out time for an INCOMPLETE record, with a mandatory correction note
- **THEN** the system sets the clock-out to the provided time, transitions the record to CLOSED, and logs the action in the audit trail

#### Scenario: Employee cannot set manual clock-out
- **WHEN** an employee attempts to directly modify or close an INCOMPLETE record
- **THEN** the system returns HTTP 403; only admins may set manual clock-out times for INCOMPLETE records

#### Scenario: INCOMPLETE records excluded from calculations
- **WHEN** earnings are computed for a period that contains INCOMPLETE records
- **THEN** those records are excluded from all totals until the admin resolves them

### Requirement: Employee can view their own time records
The system SHALL allow an employee to retrieve their own time records filtered by day, week, or month.

#### Scenario: Filter by day
- **WHEN** an employee requests records for a specific date
- **THEN** the system returns the time record for that date (if any) with duration and earned value

#### Scenario: Filter by week
- **WHEN** an employee requests records for a specific week
- **THEN** the system returns all records within that calendar week

#### Scenario: Filter by month
- **WHEN** an employee requests records for a specific month
- **THEN** the system returns all records within that calendar month

### Requirement: Admin can view time records for any employee
The system SHALL allow an admin to retrieve time records for any employee, using the same date filters (day, week, month).

#### Scenario: Admin views employee records
- **WHEN** an admin requests the time records for a specific employee and period
- **THEN** the system returns the records for that employee and period

### Requirement: Admin can directly correct clock-in and clock-out times for any time record
The system SHALL allow an admin to overwrite the clock-in timestamp, the clock-out timestamp, or both, for any employee's time record on any calendar date. A mandatory correction reason SHALL be required. The system SHALL store the original values before overwriting, recalculate earnings automatically, and record a full audit entry. Corrected records SHALL carry a visible correction flag visible in on-screen reports.

#### Scenario: Admin corrects both clock-in and clock-out on a closed record
- **WHEN** an admin submits corrected clock-in and clock-out times for a CLOSED record with a non-empty correction reason
- **THEN** the system stores the original timestamps, sets the new timestamps, transitions the record to CLOSED (corrected), recalculates the daily earnings, and records an audit entry containing: admin user, timestamp of correction, original values, new values, and correction reason

#### Scenario: Admin corrects only clock-in on an existing record
- **WHEN** an admin submits only a corrected clock-in time for a record, with a non-empty correction reason
- **THEN** the system updates only the clock-in timestamp and recalculates earnings; the correction is logged in the audit trail

#### Scenario: Admin corrects only clock-out (including INCOMPLETE resolution)
- **WHEN** an admin submits only a corrected clock-out time for a record that is CLOSED or INCOMPLETE
- **THEN** the system updates the clock-out timestamp, transitions INCOMPLETE records to CLOSED (corrected), recalculates earnings, and logs the correction with reason

#### Scenario: Admin creates a complete record for a date with no existing entry
- **WHEN** an admin submits both clock-in and clock-out times for an employee on a date that has no time record, with a mandatory correction reason
- **THEN** the system creates a new time record with status CLOSED (admin-created), calculates earnings, and records an audit entry marking the record as fully admin-created with the provided reason

#### Scenario: Correction reason is empty
- **WHEN** an admin submits a time correction without providing a correction reason
- **THEN** the system rejects the request with HTTP 400 and does not modify any record

#### Scenario: Corrected clock-out is earlier than corrected clock-in
- **WHEN** an admin submits a clock-out time that is earlier than or equal to the clock-in time (excluding valid overnight spans)
- **THEN** the system rejects the request with HTTP 400 and does not modify any record

#### Scenario: Corrected record flag visible in reports
- **WHEN** a time record has been modified by an admin correction
- **THEN** all on-screen report views for that record display a visual correction indicator (e.g., an icon or label) showing the record was manually adjusted

#### Scenario: Audit trail for admin correction
- **WHEN** an admin correction is applied to any record
- **THEN** the audit trail entry includes: admin user ID and name, correction timestamp, employee and date affected, original clock-in and clock-out values, new clock-in and clock-out values, and the correction reason provided by the admin
