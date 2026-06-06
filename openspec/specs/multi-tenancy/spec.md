## Purpose
Isolate tenant data in a shared multi-tenant SaaS deployment using tenant-scoped JWT claims and database row-level security.

## Requirements

### Requirement: Every tenant-owned record belongs to exactly one tenant
The system SHALL associate every tenant-owned record (users, roles, role-menu assignments, audit entries, and all payroll and operations data) with exactly one tenant via a non-null `tenant_id`. The `tenant_id` SHALL be derived from the authenticated request context and never from client-provided input.

#### Scenario: New record is stamped with the caller's tenant
- **WHEN** an authenticated user creates a tenant-owned record
- **THEN** the system stores the record with `tenant_id` equal to the tenant in the caller's token, regardless of any tenant value present in the request body

#### Scenario: Global feature catalogue is not tenant-scoped
- **WHEN** the system reads the menu-option catalogue
- **THEN** the catalogue is shared across tenants and is not filtered by `tenant_id`

### Requirement: Tenant data is isolated across tenants
The system SHALL prevent any tenant from reading, updating, or deleting another tenant's data. Isolation SHALL be enforced at the database layer with Row-Level Security so that it holds even if an application query omits a tenant filter.

#### Scenario: Cross-tenant read is empty
- **WHEN** a user of tenant A requests a record that belongs to tenant B by its identifier
- **THEN** the system responds as if the record does not exist (HTTP 404), never returning tenant B data

#### Scenario: Cross-tenant write is blocked
- **WHEN** a user of tenant A attempts to update or delete a record that belongs to tenant B
- **THEN** the system does not modify tenant B data and responds HTTP 404

#### Scenario: Row-Level Security blocks a missing application filter
- **WHEN** a query runs without the application-level tenant filter under the tenant connection context
- **THEN** PostgreSQL Row-Level Security restricts the result to the current tenant's rows only

### Requirement: The tenant is resolved from the JWT
The system SHALL resolve the active tenant for each request from a signed `tenant_id` claim in the access token. The claim SHALL be set at login from the authenticated user's tenant.

#### Scenario: Tenant claim issued at login
- **WHEN** a user authenticates successfully
- **THEN** the issued access token contains the user's `tenant_id`, `tenant_slug`, and `tenant_plan` claims

#### Scenario: Suspended tenant cannot log in
- **WHEN** a user whose tenant status is `SUSPENDED` attempts to log in
- **THEN** the system rejects the login with HTTP 403 and issues no token

### Requirement: Identifier uniqueness is per tenant
The system SHALL scope previously global unique identifiers to the tenant. User email and role name SHALL be unique within a tenant but MAY repeat across different tenants.

#### Scenario: Same email in two tenants
- **WHEN** an admin of tenant A and an admin of tenant B each create a user with the same email address
- **THEN** both users are created successfully because email uniqueness is enforced per tenant

#### Scenario: Duplicate email within one tenant
- **WHEN** an admin creates a second user with an email that already exists in the same tenant
- **THEN** the system rejects the request with HTTP 409

### Requirement: The provider manages tenant lifecycle
The system SHALL provide platform administration endpoints, restricted to the `PLATFORM_ADMIN` role, to create tenants and manage their plan, user cap, and status. Tenant users SHALL NOT have access to platform administration.

#### Scenario: Create a tenant
- **WHEN** a `PLATFORM_ADMIN` submits a new tenant with name, slug, plan, and user cap
- **THEN** the system creates the tenant and provisions it, returning HTTP 201

#### Scenario: Suspend a tenant
- **WHEN** a `PLATFORM_ADMIN` sets a tenant's status to `SUSPENDED`
- **THEN** the system blocks subsequent logins for that tenant's users

#### Scenario: Tenant user denied platform access
- **WHEN** a user without `PLATFORM_ADMIN` calls a `/platform` endpoint
- **THEN** the system responds HTTP 403

### Requirement: Creating a tenant provisions its baseline configuration
The system SHALL, when a tenant is created, seed the tenant's default roles, default menu assignments, and an initial tenant administrator user, in a single transactional operation.

#### Scenario: Provisioning seeds baseline
- **WHEN** a tenant is created by a `PLATFORM_ADMIN`
- **THEN** the system creates the tenant's default roles, attaches their menu options, and creates an initial `ADMIN` user for that tenant

#### Scenario: Provisioning failure rolls back
- **WHEN** any step of provisioning fails
- **THEN** the system rolls back the whole operation and no partial tenant remains

### Requirement: User creation respects the tenant plan limit
The system SHALL enforce the tenant's maximum number of users according to its plan. Attempts to exceed the cap SHALL be rejected.

#### Scenario: Within the plan limit
- **WHEN** an admin creates a user and the tenant's active user count is below `max_users`
- **THEN** the system creates the user and returns HTTP 201

#### Scenario: Exceeding the plan limit
- **WHEN** an admin creates a user and the tenant's active user count already equals `max_users`
- **THEN** the system rejects the request with HTTP 409 and error code `USER_LIMIT_REACHED`

### Requirement: Attachments are partitioned by tenant
The system SHALL store operational attachments under a tenant-specific key prefix in the shared object storage, derived from the request context.

#### Scenario: Tenant-prefixed object key
- **WHEN** a user uploads an attachment
- **THEN** the system stores the object under a key prefixed with the caller's `tenant_id` and records the tenant-prefixed URI in audit

### Requirement: Existing data is migrated to a default tenant
The system SHALL migrate all pre-existing (Phase 1) data into a single default tenant so that current behaviour is preserved after the multi-tenancy migration.

#### Scenario: Legacy data assigned to the default tenant
- **WHEN** the multi-tenancy migration runs against a database with existing users, roles, and audit entries
- **THEN** every existing row is assigned to the default (legacy) tenant and remains accessible to that tenant's users
