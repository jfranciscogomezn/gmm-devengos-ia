## ADDED Requirements

### Requirement: System calculates daily earned value per time record
The system SHALL compute the devengado (earned monetary value) for each time record by applying the employee's hourly rate to the classified hours, using the payroll configuration active for that calendar year.

#### Scenario: Normal-hours day
- **WHEN** an employee works only within the normal shift time range
- **THEN** the system computes earned value as: (worked hours after rest deduction) × hourly rate × 1.0

#### Scenario: Day with daytime overtime
- **WHEN** an employee's worked time extends into the daytime overtime range
- **THEN** the system applies the daytime overtime surcharge percentage to hours within that range

#### Scenario: Day with night surcharge hours
- **WHEN** an employee's worked time falls within the night surcharge range
- **THEN** the system applies the night surcharge percentage to hours within that range

#### Scenario: Day with nocturnal overtime
- **WHEN** an employee's worked time extends into the nocturnal overtime range
- **THEN** the system applies the nocturnal overtime percentage to hours within that range

### Requirement: Non-billable rest time is deducted before classification
The system SHALL subtract the configured daily non-billable rest minutes from the total worked duration before classifying hours into time ranges.

#### Scenario: Rest deduction applied
- **WHEN** an employee works 9 hours and the rest deduction is 30 minutes
- **THEN** the system classifies 8h 30min of billable time across the applicable ranges

### Requirement: Hours on Sundays and holidays receive the configured surcharge
The system SHALL detect whether the work date is a Sunday or a date in the holiday calendar for that year, and apply the corresponding surcharge to all worked hours on that day.

#### Scenario: Sunday work
- **WHEN** a time record falls on a Sunday
- **THEN** all classified hours receive the Sunday/holiday surcharge percentage

#### Scenario: Holiday work
- **WHEN** a time record date is in the holiday calendar
- **THEN** all classified hours receive the Sunday/holiday surcharge percentage

### Requirement: Hourly rate is derived from employee's monthly salary
The system SHALL compute the employee's hourly rate as: monthly salary ÷ configured monthly work hours.

#### Scenario: Hourly rate calculation
- **WHEN** an employee has a monthly salary of X and the config defines Y monthly hours
- **THEN** the hourly rate used in earnings calculation is X ÷ Y

### Requirement: Capped view limits hours to 8 normal + 2 daytime overtime
The system SHALL provide a capped earnings view that limits counted hours to a maximum of 8 normal hours and 2 daytime overtime hours per day, regardless of total hours worked.

#### Scenario: Excess hours capped
- **WHEN** an employee works 12 hours in a day
- **THEN** the capped view computes earnings based on 8 normal + 2 overtime hours only (10 total)

#### Scenario: Under-limit day unaffected
- **WHEN** an employee works 7 hours in a day
- **THEN** the capped view computes earnings based on the full 7 hours (no cap applied)
