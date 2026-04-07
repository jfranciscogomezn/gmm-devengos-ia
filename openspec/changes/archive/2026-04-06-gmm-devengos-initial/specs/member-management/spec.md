## ADDED Requirements

### Requirement: Admin can register a member
The system SHALL allow an admin to register a new member with personal information. Optionally, the admin MAY create a user account (username and password) for the member at registration time.

#### Scenario: Member registered without user account
- **WHEN** an admin submits member personal data without user credentials
- **THEN** the system creates the member record without login access and returns HTTP 201

#### Scenario: Member registered with user account
- **WHEN** an admin submits member personal data along with username, password, and a role
- **THEN** the system creates the member with a hashed password, assigns the role, and returns HTTP 201

#### Scenario: Duplicate email
- **WHEN** an admin registers a member with an email already in use
- **THEN** the system returns HTTP 409

### Requirement: Admin can enable or disable a member's user account
The system SHALL allow an admin to toggle the enabled/disabled status of a member's user account. Only enabled accounts SHALL be able to log in.

#### Scenario: Disable an enabled account
- **WHEN** an admin disables a member's account
- **THEN** the system marks the account as disabled and the member can no longer log in

#### Scenario: Enable a disabled account
- **WHEN** an admin enables a member's account
- **THEN** the system marks the account as enabled and the member can log in again

### Requirement: Admin can list all members
The system SHALL allow an admin to retrieve a list of all registered members with their account status and role.

#### Scenario: Members exist
- **WHEN** an admin requests the member list
- **THEN** the system returns HTTP 200 with an array of members including their enabled status and role name

### Requirement: Admin can update member information
The system SHALL allow an admin to update a member's personal data, role, and account status.

#### Scenario: Successful update
- **WHEN** an admin submits updated fields for an existing member
- **THEN** the system applies the changes and returns HTTP 200

#### Scenario: Member not found
- **WHEN** an admin attempts to update a non-existent member
- **THEN** the system returns HTTP 404

### Requirement: Password meets quality standards
The system SHALL enforce that passwords meet minimum quality requirements: at least 8 characters, at least one uppercase letter, one lowercase letter, one digit, and one special character.

#### Scenario: Weak password rejected
- **WHEN** an admin sets a password that does not meet the quality criteria
- **THEN** the system returns HTTP 400 with a message describing the requirements

#### Scenario: Strong password accepted
- **WHEN** an admin sets a password that meets all quality criteria
- **THEN** the system stores the bcrypt hash and returns success
