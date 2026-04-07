## ADDED Requirements

### Requirement: User can log in with valid credentials
The system SHALL authenticate a user by email and password. On success it SHALL return a short-lived JWT access token. The password MUST be verified against the bcrypt hash stored in the database. Only members with an active (enabled) user account SHALL be allowed to log in.

#### Scenario: Successful login
- **WHEN** a member with an enabled account submits correct email and password
- **THEN** the system returns HTTP 200 with a JWT access token and the user's role information

#### Scenario: Wrong password
- **WHEN** a member submits an incorrect password
- **THEN** the system returns HTTP 401 with an error message and does not reveal whether the email exists

#### Scenario: Disabled account
- **WHEN** a member with a disabled account attempts to log in
- **THEN** the system returns HTTP 403 indicating the account is inactive

#### Scenario: Non-existent email
- **WHEN** an email not registered in the system is submitted
- **THEN** the system returns HTTP 401 with a generic invalid-credentials message

### Requirement: Protected endpoints require a valid JWT
The system SHALL reject requests to protected endpoints that do not carry a valid, non-expired JWT in the Authorization header (Bearer scheme).

#### Scenario: Missing token
- **WHEN** a request is made to a protected endpoint without an Authorization header
- **THEN** the system returns HTTP 401

#### Scenario: Expired token
- **WHEN** a request is made with an expired JWT
- **THEN** the system returns HTTP 401

#### Scenario: Valid token
- **WHEN** a request carries a valid JWT
- **THEN** the system processes the request normally

### Requirement: Role-based access control on endpoints
The system SHALL enforce that a user can only access endpoints permitted by their assigned role's menu options.

#### Scenario: Unauthorized role access
- **WHEN** a user with role A attempts to access an endpoint not permitted for role A
- **THEN** the system returns HTTP 403

#### Scenario: Authorized role access
- **WHEN** a user with role A accesses an endpoint permitted for role A
- **THEN** the system processes the request normally

### Requirement: User can log out
The system SHALL provide a logout endpoint. After logout the client-side token SHALL be discarded. The system SHALL treat the token as invalid for the remainder of its natural expiry (stateless invalidation via short TTL is acceptable in v1).

#### Scenario: Successful logout
- **WHEN** an authenticated user calls the logout endpoint
- **THEN** the system returns HTTP 200 and the client discards the token
