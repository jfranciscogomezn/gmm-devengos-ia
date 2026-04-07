## ADDED Requirements

### Requirement: Admin can configure personal information for an employee
The system SHALL allow an admin to set or update an employee's personal data: first name, last name, ID type, ID number, email address, and mobile phone number.

#### Scenario: Create employee personal info
- **WHEN** an admin submits valid personal data for a new employee
- **THEN** the system stores the data and returns HTTP 201

#### Scenario: Update employee personal info
- **WHEN** an admin submits updated personal data for an existing employee
- **THEN** the system updates the record and returns HTTP 200

#### Scenario: Duplicate ID number
- **WHEN** an admin registers an employee with an ID number already in use
- **THEN** the system returns HTTP 409

### Requirement: Admin can configure salary for an employee
The system SHALL allow an admin to set or update the monthly salary for a specific employee. The salary SHALL be used as the basis for earnings calculation.

#### Scenario: Set employee salary
- **WHEN** an admin submits a positive salary value for an employee
- **THEN** the system stores the salary and returns HTTP 200

#### Scenario: Negative salary rejected
- **WHEN** an admin submits a zero or negative salary
- **THEN** the system returns HTTP 400

### Requirement: Admin can list all employees
The system SHALL allow an admin to retrieve a list of all configured employees with their personal information and current salary.

#### Scenario: Employees exist
- **WHEN** an admin requests the employee list
- **THEN** the system returns HTTP 200 with an array of employee records

### Requirement: Admin can retrieve a single employee
The system SHALL allow an admin to retrieve the full configuration of a specific employee by ID.

#### Scenario: Employee found
- **WHEN** an admin requests an existing employee by ID
- **THEN** the system returns HTTP 200 with the employee's personal data and salary

#### Scenario: Employee not found
- **WHEN** an admin requests a non-existent employee ID
- **THEN** the system returns HTTP 404
