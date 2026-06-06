## Purpose
Bootstrap the standalone business microservice with stateless JWT authentication, tenant isolation, and shared-database deployment.

## Requirements

### Requirement: Standalone business backend on the shared database
The `stepcore-business-backend` service SHALL run as a standalone process, separate from the security backend, while connecting to the **shared** PostgreSQL database (`stepcore_security`). It SHALL own its own tables and SHALL apply its migrations through a dedicated Flyway history table so the two services' migration streams never collide. It SHALL NOT query the security backend's user/role tables for authorization.

#### Scenario: Service starts against the shared database
- **WHEN** the business service boots pointing at the shared database
- **THEN** Flyway applies the business migrations using its own history table and the service exposes its API, without modifying the security service's tables or history

#### Scenario: Tenant identity owned by the business service
- **WHEN** a business table needs the tenant identity
- **THEN** it stores `tenant_id BIGINT NOT NULL`, stamped from the JWT, with **no** foreign key into the security service's `tenants` table, preserving the ownership boundary and keeping the business migrations independently runnable

### Requirement: Stateless JWT authentication for resource endpoints
The service SHALL authenticate requests by validating a Bearer JWT signed by the security service with the shared HMAC secret. It SHALL reject requests with a missing, malformed, expired, or wrongly-signed token with HTTP 401, and SHALL NOT maintain server-side sessions.

#### Scenario: Valid token is accepted
- **WHEN** a request presents a Bearer token signed by the security service with a non-expired expiry
- **THEN** the service establishes the security context from the token claims and processes the request

#### Scenario: Invalid signature is rejected
- **WHEN** a request presents a token signed with a different secret
- **THEN** the service returns HTTP 401 and does not process the request

#### Scenario: Missing token on a protected endpoint
- **WHEN** a request to a protected endpoint omits the Authorization header
- **THEN** the service returns HTTP 401

### Requirement: Authorities derived from the token roles claim
The service SHALL grant Spring Security authorities from the JWT `roles` claim (mapped as `ROLE_<role>`) so that role-restricted endpoints can be authorized without querying the security database.

#### Scenario: Admin role authorizes a protected operation
- **WHEN** a token carries the `ADMIN` role and the caller invokes an `ADMIN`-only endpoint
- **THEN** the request is authorized

#### Scenario: Missing required role is forbidden
- **WHEN** a token lacks the `ADMIN` role and the caller invokes an `ADMIN`-only endpoint
- **THEN** the service returns HTTP 403

### Requirement: Tenant isolation enforced by Row-Level Security
Every tenant-owned table SHALL carry `tenant_id BIGINT NOT NULL` and a PostgreSQL Row-Level Security policy keyed on `current_setting('app.current_tenant')`. The runtime database role SHALL be a non-owner role without `BYPASSRLS`. The service SHALL set the tenant GUC from the authenticated token's `tenant_id` for the active transaction and SHALL stamp `tenant_id` on persisted rows from the token, never from request input.

#### Scenario: Reads are confined to the caller's tenant
- **WHEN** a user of tenant A queries a tenant-owned resource
- **THEN** only rows with `tenant_id = A` are returned, even for find-by-id access paths

#### Scenario: Cross-tenant access is denied
- **WHEN** a user of tenant A requests a resource owned by tenant B by id
- **THEN** the service responds HTTP 404 (the row is invisible under RLS)

#### Scenario: Unset tenant context matches no rows
- **WHEN** a query runs without an established tenant context
- **THEN** RLS yields zero rows (deny by default)
