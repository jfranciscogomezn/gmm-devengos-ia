## Purpose
Provide hierarchical, role-based navigation loaded dynamically at login from a modular menu catalogue.

## Requirements

### Requirement: Menu catalogue is a hierarchical tree
The system SHALL store the application menu as a global hierarchical catalogue in `menu_nodes` with exactly three node types: `MODULE` (top-level product area), `GROUP` (subsection within a module), and `ITEM` (navigable screen). Each node SHALL have a unique `code`, a display `label`, a sibling `sort_order`, and an `enabled` flag. Only `ITEM` nodes SHALL carry a non-null `route`.

#### Scenario: Module is a root node
- **WHEN** the catalogue contains a node with `node_type = MODULE`
- **THEN** that node's `parent_id` is null and its `route` is null

#### Scenario: Item is a leaf with a route
- **WHEN** the catalogue contains a node with `node_type = ITEM`
- **THEN** that node has a non-null `route` and is assignable to roles as a permission

#### Scenario: Group has no route
- **WHEN** the catalogue contains a node with `node_type = GROUP`
- **THEN** that node's `route` is null and it serves only as a structural container

### Requirement: Roles are granted menu access at leaf nodes only
The system SHALL associate roles with menu access through `role_menu_nodes`, referencing only `ITEM` nodes. Assignments to `MODULE` or `GROUP` nodes SHALL be rejected.

#### Scenario: Assign item to role
- **WHEN** an administrator assigns an `ITEM` node to a role
- **THEN** the assignment is persisted in `role_menu_nodes`

#### Scenario: Reject non-item assignment
- **WHEN** an administrator attempts to assign a `MODULE` or `GROUP` node to a role
- **THEN** the system rejects the request with HTTP 400

### Requirement: Login returns a role-specific menu tree
The system SHALL, upon successful authentication, return a nested menu tree containing only `MODULE`, `GROUP`, and `ITEM` nodes that are reachable from at least one `ITEM` assigned to the user's role. Nodes with `enabled = false` SHALL be excluded. The tree SHALL be ordered by `sort_order` at each level.

#### Scenario: Admin sees modular payroll section
- **WHEN** a user with role `ADMIN` logs in and has `EMPLOYEE_CONFIG` and `PAYROLL_CONFIG` assigned
- **THEN** the login response includes a `Payroll` module containing a `Configuration` group with those two items as children

#### Scenario: Employee sees limited modules
- **WHEN** a user with role `EMPLOYEE` logs in with only `MY_TIME` and `MY_PROFILE` assigned
- **THEN** the login response includes only modules/groups leading to those two items and does not include Security or Payroll administration items

#### Scenario: Empty group is omitted
- **WHEN** a role has no assigned items under a group's subtree
- **THEN** that group does not appear in the login menu tree

### Requirement: Login exposes flat permissions for authorization
The system SHALL include a flat `permissions` array in the login response listing the `code` of every assigned, enabled `ITEM` node for the user's role. The same codes SHALL be embedded in the JWT `permissions` claim.

#### Scenario: Permissions match assigned leaves
- **WHEN** a user logs in with items `EMPLOYEE_CONFIG` and `USER_MANAGEMENT` assigned
- **THEN** the login response `permissions` array contains exactly those codes and the JWT carries the same list

#### Scenario: Structural nodes are not permissions
- **WHEN** a user logs in
- **THEN** neither the `permissions` array nor the JWT contains `MODULE` or `GROUP` codes

### Requirement: Platform administrator menu is catalogue-driven
The system SHALL include platform administration entries in the same `menu_nodes` catalogue. Users with role `PLATFORM_ADMIN` SHALL receive the platform module subtree according to role assignment, consistent with tenant users.

#### Scenario: Platform admin receives tenants item
- **WHEN** a `PLATFORM_ADMIN` user logs in with the platform tenants item assigned
- **THEN** the menu tree includes a navigable item for tenant administration

### Requirement: Tenant provisioning seeds default menu assignments
The system SHALL, when provisioning a new tenant, assign the default administrator role the same baseline set of menu item leaves as the legacy tenant administrator (all tenant-relevant items, excluding platform-only items).

#### Scenario: New tenant admin can access security and payroll config
- **WHEN** a new tenant is provisioned
- **THEN** its seeded `ADMIN` role includes item leaves for role management, user management, payroll configuration, employee configuration, time records administration, and reports

### Requirement: Frontend renders the modular menu tree
The frontend SHALL render the login menu tree with collapsible `MODULE` and `GROUP` sections and navigable links for `ITEM` nodes. Route protection SHALL continue to use leaf permission codes.

#### Scenario: Sidebar groups payroll configuration items
- **WHEN** an administrator views the sidebar after login
- **THEN** payroll configuration and employee configuration appear under the Payroll module, not as a flat unordered list

#### Scenario: Route requires leaf permission
- **WHEN** a user without `EMPLOYEE_CONFIG` navigates to `/admin/employees`
- **THEN** the frontend denies access according to the user's permission set

### Requirement: Role management UI supports tree assignment
The system SHALL provide an administration UI to assign menu item leaves to a role using a hierarchical tree control that mirrors the catalogue structure.

#### Scenario: Select all items in a module
- **WHEN** an administrator selects a module checkbox in the role menu assignment screen
- **THEN** all enabled item leaves under that module are selected for assignment

#### Scenario: Persisted assignment is leaf-only
- **WHEN** an administrator saves role menu changes
- **THEN** only item node ids are sent to and stored by the backend

### Requirement: Login response menu payload shape
The system SHALL return the authenticated user's navigation menu as a nested `menu` tree in the login response instead of a flat `menuOptions` list.

#### Scenario: Login response contains menu tree
- **WHEN** a user authenticates successfully
- **THEN** the login response body includes a `menu` array of nested nodes and does not include the legacy flat `menuOptions` field
