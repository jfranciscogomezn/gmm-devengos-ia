## Purpose
Configure tenant-scoped global payroll parameters and holiday calendars per calendar year for earnings calculation.

## Requirements

### Requirement: Admin can configure global payroll parameters for a year
The system SHALL allow an `ADMIN` to create or update the payroll configuration for a given calendar year within their tenant. The configuration SHALL include all parameters needed for earnings calculation. Configuration is unique per `(tenant_id, year)`.

#### Scenario: Create configuration for a new year
- **WHEN** an admin submits payroll parameters for a year with no existing configuration in their tenant
- **THEN** the system creates the configuration and returns HTTP 201

#### Scenario: Update configuration for an existing year
- **WHEN** an admin submits payroll parameters for a year that already has a configuration in their tenant
- **THEN** the system updates the configuration and returns HTTP 200

#### Scenario: Retrieve configuration for a year
- **WHEN** an admin requests the payroll config for a specific year in their tenant
- **THEN** the system returns HTTP 200 with the full configuration object

#### Scenario: Retrieve a non-existent year
- **WHEN** an admin requests the payroll config for a year that has no configuration
- **THEN** the system returns HTTP 404

### Requirement: Payroll configuration includes salary and subsidy parameters
The configuration SHALL store the minimum monthly wage and the monthly transport subsidy for the configured year. Both values SHALL be non-negative.

#### Scenario: Missing required monetary field
- **WHEN** an admin submits a configuration without minimum wage or transport subsidy
- **THEN** the system returns HTTP 400

#### Scenario: Negative monetary value rejected
- **WHEN** an admin submits a negative minimum wage or transport subsidy
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes work-hour limits
The configuration SHALL store total monthly work hours, normal daily work hours, and maximum daily extra hours. Each SHALL be greater than zero where required and non-negative otherwise.

#### Scenario: Invalid hour value
- **WHEN** an admin submits a negative or zero value for monthly work hours
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes time-range boundaries
The configuration SHALL store start and end times (HH:MM) for the normal shift, daytime overtime, night surcharge, and nocturnal overtime ranges.

#### Scenario: All time boundaries provided
- **WHEN** an admin provides valid HH:MM values for all four time ranges
- **THEN** the system stores them and returns success

#### Scenario: Invalid time format
- **WHEN** an admin provides a time value that is not in HH:MM format
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes surcharge percentages
The configuration SHALL store non-negative percentage values for night surcharge, daytime overtime, nocturnal overtime, and Sunday/holiday surcharge.

#### Scenario: Negative percentage rejected
- **WHEN** an admin submits a negative percentage for any surcharge
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes non-billable rest minutes
The configuration SHALL store the number of daily rest minutes that are deducted from total worked time before earnings calculation. The value SHALL be non-negative.

#### Scenario: Rest minutes persisted
- **WHEN** an admin submits a configuration with non-billable rest minutes
- **THEN** the system stores the value and returns it on subsequent reads

### Requirement: Admin can manage the holiday calendar for a year
The system SHALL allow an `ADMIN` to add, list, and remove individual holiday dates for a given year within their tenant. A holiday is unique per `(tenant_id, date)`.

#### Scenario: Add a holiday
- **WHEN** an admin submits a date to the holiday list for a year
- **THEN** the system stores the holiday and returns HTTP 201

#### Scenario: Reject a duplicate holiday
- **WHEN** an admin submits a date that already exists for their tenant
- **THEN** the system returns HTTP 409

#### Scenario: Remove a holiday
- **WHEN** an admin deletes a holiday date
- **THEN** the system removes it and returns HTTP 204

#### Scenario: List holidays for a year
- **WHEN** an admin requests the holiday list for a year
- **THEN** the system returns HTTP 200 with all dates for that year in their tenant

### Requirement: Payroll configuration and holidays are tenant-isolated
All payroll configuration and holiday reads and writes SHALL be confined to the caller's tenant via Row-Level Security; `tenant_id` SHALL be stamped from the authenticated token.

#### Scenario: Tenant cannot read another tenant's configuration
- **WHEN** an admin of tenant A requests a year configured only in tenant B
- **THEN** the system returns HTTP 404

#### Scenario: Same year may exist independently per tenant
- **WHEN** tenant A and tenant B each configure the same year
- **THEN** both configurations coexist and each tenant sees only its own
