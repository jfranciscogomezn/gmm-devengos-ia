## Purpose
Record an immutable audit trail when administrators correct or manage employee time records.

## Requirements

### Requirement: Admin time mutations are recorded in the centralized audit log
The system SHALL append a row to `audit_logs` for each admin operation that reopens, resolves an INCOMPLETE record, corrects timestamps, or creates an admin time record. Entries MUST be tenant-scoped and MUST include the acting admin, affected time record id, and correction metadata.

#### Scenario: Resolve INCOMPLETE writes audit entry
- **WHEN** an admin resolves an INCOMPLETE time record with a manual clock-out and mandatory note
- **THEN** the system inserts an audit log row with action `TIME_RECORD_RESOLVE_INCOMPLETE`, entity type `TIME_RECORD`, before/after clock values, and the note in details

#### Scenario: Correct timestamps writes audit entry
- **WHEN** an admin corrects clock-in and/or clock-out on an existing record with a non-empty correction reason
- **THEN** the system inserts an audit log row with action `TIME_RECORD_CORRECT`, original and new timestamps, and the correction reason

#### Scenario: Admin-created record writes audit entry
- **WHEN** an admin creates a complete time record for a date with no prior entry
- **THEN** the system inserts an audit log row with action `TIME_RECORD_CREATE`, new clock values, and the mandatory reason

#### Scenario: Reopen writes audit entry
- **WHEN** an admin reopens a closed time record
- **THEN** the system inserts an audit log row with action `TIME_RECORD_REOPEN` referencing the record id and prior closed timestamps

### Requirement: Audit entries reference the acting admin user
The system SHALL resolve the admin's `users.id` from the JWT subject email within the current tenant when persisting audit rows.

#### Scenario: Known admin user
- **WHEN** the actor email matches an enabled user in the current tenant
- **THEN** the audit row includes that user's id as `user_id`

#### Scenario: Unknown admin user
- **WHEN** no matching user row exists
- **THEN** the audit row is still persisted with `user_id` null and the actor email stored in details

### Requirement: Admins can list time-record audit history
The system SHALL expose a read API for admins to retrieve recent audit entries for time records in their tenant.

#### Scenario: List recent entries
- **WHEN** an admin with `TIME_RECORDS_ADMIN` calls the audit list endpoint
- **THEN** the system returns HTTP 200 with the most recent time-record audit entries for the tenant (newest first, bounded limit)

#### Scenario: Filter by date range
- **WHEN** an admin calls the audit list endpoint with `from` and `to` date parameters
- **THEN** the system returns only entries whose `created_at` falls within that inclusive date range

#### Scenario: Filter by employee
- **WHEN** an admin calls the audit list endpoint with `employeeId`
- **THEN** the system returns only entries where the affected time record's `employeeId` appears in the stored before or after snapshot

#### Scenario: Filter by actor user
- **WHEN** an admin calls the audit list endpoint with `userId`
- **THEN** the system returns only entries performed by that admin user

#### Scenario: Employee forbidden
- **WHEN** a user without `TIME_RECORDS_ADMIN` calls the audit list endpoint
- **THEN** the system returns HTTP 403

### Requirement: Audit data remains tenant-isolated
The system MUST NOT allow a tenant to read or write audit rows belonging to another tenant.

#### Scenario: Cross-tenant isolation
- **WHEN** tenant A's admin requests audit history
- **THEN** only audit rows with tenant A's id are returned

### Requirement: Admins can browse time-record audit history in the web UI
The system SHALL provide an admin screen to list time-record audit entries with filters for date range, employee, and acting user.

#### Scenario: Open audit history page
- **WHEN** an admin with access to the time audit menu opens `/admin/time/audit`
- **THEN** the UI loads recent audit entries for the current tenant and shows action, timestamp, actor, employee, and before/after summary

#### Scenario: Apply filters in UI
- **WHEN** the admin selects a date range, employee, and/or user filter and refreshes the list
- **THEN** the UI requests the audit API with matching query parameters and displays the filtered results

#### Scenario: Unauthorized user
- **WHEN** a user without `TIME_RECORDS_ADMIN` navigates to `/admin/time/audit`
- **THEN** the UI redirects away from the audit screen
