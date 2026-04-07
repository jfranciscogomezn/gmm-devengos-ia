## ADDED Requirements

### Requirement: Admin can create a role
The system SHALL allow an admin to create a new role with a unique name.

#### Scenario: Successful role creation
- **WHEN** an admin submits a unique role name
- **THEN** the system creates the role and returns HTTP 201 with the created role

#### Scenario: Duplicate role name
- **WHEN** an admin submits a role name that already exists
- **THEN** the system returns HTTP 409 with a conflict error

### Requirement: Admin can list all roles
The system SHALL allow an admin to retrieve all existing roles.

#### Scenario: Roles exist
- **WHEN** an admin requests the list of roles
- **THEN** the system returns HTTP 200 with an array of all roles

### Requirement: Admin can update a role
The system SHALL allow an admin to update a role's name.

#### Scenario: Successful update
- **WHEN** an admin submits a new unique name for an existing role
- **THEN** the system updates the role and returns HTTP 200

#### Scenario: Role not found
- **WHEN** an admin attempts to update a role that does not exist
- **THEN** the system returns HTTP 404

### Requirement: Admin can delete a role
The system SHALL allow an admin to delete a role that has no members assigned to it.

#### Scenario: Successful deletion
- **WHEN** an admin deletes a role with no assigned members
- **THEN** the system removes the role and returns HTTP 200

#### Scenario: Role has assigned members
- **WHEN** an admin attempts to delete a role that has one or more members assigned
- **THEN** the system returns HTTP 409 with a descriptive error

### Requirement: Admin can assign menu options to a role
The system SHALL allow an admin to assign a set of menu options to a role, replacing the current assignment.

#### Scenario: Successful assignment
- **WHEN** an admin submits a list of menu option IDs for a role
- **THEN** the system replaces the role's menu options with the provided list and returns HTTP 200

#### Scenario: Invalid menu option ID
- **WHEN** an admin includes a menu option ID that does not exist
- **THEN** the system returns HTTP 400

### Requirement: Admin can retrieve menu options for a role
The system SHALL return the list of menu options currently assigned to a given role.

#### Scenario: Role has menu options
- **WHEN** an admin requests the menu options of an existing role
- **THEN** the system returns HTTP 200 with the list of assigned menu options

### Requirement: A user can only hold one role at a time
The system SHALL enforce that each member is assigned exactly one role. Assigning a new role to a member SHALL replace the previous one.

#### Scenario: Role reassignment
- **WHEN** an admin assigns a different role to a member who already has a role
- **THEN** the system replaces the previous role with the new one and the member retains only the new role
