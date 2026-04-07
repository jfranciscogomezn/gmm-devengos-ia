## ADDED Requirements

### Requirement: Admin can configure global payroll parameters for a year
The system SHALL allow an admin to create or update the global payroll configuration for a given calendar year. The configuration SHALL include all parameters needed for earnings calculation.

#### Scenario: Create configuration for a new year
- **WHEN** an admin submits payroll parameters for a year with no existing configuration
- **THEN** the system creates the configuration and returns HTTP 201

#### Scenario: Update configuration for an existing year
- **WHEN** an admin submits payroll parameters for a year that already has a configuration
- **THEN** the system updates the configuration and returns HTTP 200

#### Scenario: Retrieve configuration for a year
- **WHEN** an admin or the system requests the payroll config for a specific year
- **THEN** the system returns HTTP 200 with the full configuration object

### Requirement: Payroll configuration includes salary and subsidy parameters
The configuration SHALL store: minimum monthly wage (salario mínimo) and monthly transport subsidy (auxilio de transporte) for the configured year.

#### Scenario: Missing required monetary field
- **WHEN** an admin submits a configuration without minimum wage or transport subsidy
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes work-hour limits
The configuration SHALL store: total monthly work hours, normal daily work hours (standard weekly schedule), and maximum daily extra hours.

#### Scenario: Invalid hour value
- **WHEN** an admin submits a negative or zero value for monthly work hours
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes time-range boundaries
The configuration SHALL store start and end times (HH:MM) for: normal shift, daytime overtime, night surcharge, and nocturnal overtime.

#### Scenario: All time boundaries provided
- **WHEN** an admin provides valid HH:MM values for all four time ranges
- **THEN** the system stores them and returns success

#### Scenario: Invalid time format
- **WHEN** an admin provides a time value that is not in HH:MM format
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes surcharge percentages
The configuration SHALL store percentage values for: night surcharge (recargo nocturno), daytime overtime (hora extra diurna), nocturnal overtime (hora extra nocturna), and Sunday/holiday surcharge (domingos y festivos).

#### Scenario: Negative percentage rejected
- **WHEN** an admin submits a negative percentage for any surcharge
- **THEN** the system returns HTTP 400

### Requirement: Payroll configuration includes non-billable rest minutes
The configuration SHALL store the number of daily rest minutes that SHALL be deducted from total worked time before earnings calculation.

#### Scenario: Rest minutes deducted
- **WHEN** an earnings calculation is performed for a day
- **THEN** the non-billable rest minutes are subtracted from the total worked duration before applying time-range classification

### Requirement: Admin can manage the holiday calendar for a year
The system SHALL allow an admin to add and remove individual holiday dates for a given year. Holidays affect the surcharge applied to hours worked on those dates.

#### Scenario: Add a holiday
- **WHEN** an admin submits a date to the holiday list for a year
- **THEN** the system stores the holiday and returns HTTP 201

#### Scenario: Remove a holiday
- **WHEN** an admin deletes a holiday date
- **THEN** the system removes it and returns HTTP 200

#### Scenario: List holidays for a year
- **WHEN** an admin requests the holiday list for a year
- **THEN** the system returns HTTP 200 with all dates for that year
