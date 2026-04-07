## 1. Project Scaffolding

- [x] 1.1 Initialize backend project — implemented with Java 17 + Spring Boot 3.x (replaces Node.js/Express); includes Maven, JaCoCo, SpringDoc OpenAPI
- [x] 1.2 Initialize frontend project (React + TypeScript + Vite)
- [x] 1.3 Configure Docker Compose for PostgreSQL
- [x] 1.4 Initialize persistence layer — implemented with Spring Data JPA + Flyway (replaces Prisma); first migration V1 created and verified
- [x] 1.5 Add shared dependencies — Spring Security (bcrypt strength 12, JWT via JJWT 0.12.6), MapStruct, Lombok; exceljs/zod replaced by Spring equivalents; excel export pending with later tasks

## 2. Database Schema

- [x] 2.1 Define JPA entities + Flyway: Role, MenuOption, role_menu_options join table (V1 migration)
- [x] 2.2 Define JPA entity: User (linked to Role; replaces Member+User split — unified as User with role and personal data) (V2 migration)
- [ ] 2.3 Define JPA entity: Employee (personal info + salary)
- [ ] 2.4 Define JPA entities: PayrollConfig, Holiday
- [ ] 2.5 Define JPA entity: TimeRecord
- [x] 2.6 Run and verify Flyway migrations — V1–V4 applied successfully on local PostgreSQL
- [x] 2.7 Seed data — V4 migration seeds 8 menu options + ADMIN/EMPLOYEE roles; DataSeeder creates default admin@gmm.com on startup

## 3. Auth Capability

- [x] 3.1 Write failing tests for login endpoint (success, wrong password, disabled account, unknown email) — AuthServiceTest + AuthControllerIT
- [x] 3.2 Implement POST /api/v1/auth/login: validate credentials, bcrypt compare, return JWT + user profile + menu options
- [x] 3.3 Implement JWT filter: JwtAuthenticationFilter (OncePerRequestFilter), JwtService, UserDetailsServiceImpl
- [x] 3.4 Implement role-based access — Spring Security @PreAuthorize at method level; SecurityConfig stateless session
- [x] 3.5 Implement POST /auth/logout (200 + client-side discard), GET /auth/me, POST /auth/change-password (with bcrypt + password pattern validation)
- [x] 3.6 All auth tests pass — unit (JwtServiceTest, AuthServiceTest) + integration (AuthControllerIT)

## 4. Role Management Capability

- [x] 4.1 Write failing tests for role CRUD and menu-option assignment — RoleServiceTest + RoleControllerIT
- [x] 4.2 Implement GET /api/v1/roles and POST /api/v1/roles (admin only; duplicate name validation)
- [x] 4.3 Implement GET /roles/:id, PUT /roles/:id, DELETE /roles/:id (rejects delete if users assigned — RoleInUseException)
- [x] 4.4 Implement GET /roles (includes menuOptions in response) and PUT /roles/:id/menu-options (replaces full set)
- [x] 4.5 All role management tests pass — unit (RoleServiceTest) + integration (RoleControllerIT)

## 5. Member Management Capability

- [x] 5.1 Write failing tests for user registration, enable/disable, update, and password validation — UserServiceTest + UserControllerIT
- [x] 5.2 Implement POST /api/v1/users (bcrypt hash strength 12; password pattern validation; duplicate email check; admin only)
- [x] 5.3 Implement GET /users and GET /users/:id (admin only)
- [x] 5.4 Implement PUT /users/:id (update personal data, role; duplicate email check; admin only)
- [x] 5.5 Implement PATCH /users/:id/status (enable/disable) and POST /users/:id/reset-password (admin only)
- [x] 5.6 All user management tests pass — unit (UserServiceTest) + integration (UserControllerIT)

## 6. Payroll Configuration Capability

- [ ] 6.1 Write failing tests for payroll config create/update/read and holiday management
- [ ] 6.2 Implement GET /config/payroll/:year and PUT /config/payroll/:year
- [ ] 6.3 Add input validation: required fields, non-negative values, HH:MM time format, non-negative percentages
- [ ] 6.4 Implement GET /config/holidays/:year, POST /config/holidays, DELETE /config/holidays/:id
- [ ] 6.5 Pass all payroll config tests

## 7. Employee Configuration Capability

- [ ] 7.1 Write failing tests for employee CRUD (personal info + salary)
- [ ] 7.2 Implement POST /employees and GET /employees
- [ ] 7.3 Implement GET /employees/:id, PUT /employees/:id
- [ ] 7.4 Pass all employee config tests

## 8. Time Tracking Capability

- [ ] 8.1 Write failing tests for clock-in, clock-out, immutability, admin reopen, and list/filter
- [ ] 8.2 Implement POST /time-records/clock-in (once per day enforcement)
- [ ] 8.3 Implement POST /time-records/clock-out (requires open record; closes it)
- [ ] 8.4 Implement PATCH /time-records/:id/reopen (admin only)
- [ ] 8.5 Implement GET /time-records (filter by employeeId, date/week/month/custom date range; employee sees own, admin sees any)
- [ ] 8.6 Implement scheduled job (00:01 server time): detect OPEN records from prior days → transition to INCOMPLETE → notify admins via in-app + email
- [ ] 8.7 Implement PATCH /time-records/:id/resolve-incomplete (admin only): accept manual clock-out time + mandatory note; close record; log in audit trail
- [ ] 8.8 Add GET /time-records/incomplete endpoint: employee sees their own INCOMPLETE records; admin sees any employee's INCOMPLETE records
- [ ] 8.9 Implement PUT /time-records/correct (admin only): accept employeeId, date, clockIn (optional), clockOut (optional), correctionReason (required); handle all record statuses; store original values; set corrected flag; recalculate earnings; write audit entry with before/after values and reason
- [ ] 8.10 Implement POST /time-records/correct (admin only): create a complete record (admin-created) for a date with no existing entry; same mandatory reason and audit requirements
- [ ] 8.11 Write tests for admin correction: both times, clock-in only, clock-out only, new record creation, empty reason (400), invalid time range (400), audit entry content, corrected flag visibility
- [ ] 8.12 Write tests for auto-flagging job, admin-only manual clock-out, employee 403 on direct resolution attempt, and persistent warning data
- [ ] 8.13 Pass all time tracking tests

## 9. Earnings Calculation Capability

- [ ] 9.1 Write unit tests for earnings calculation service (all time range scenarios, rest deduction, Sunday/holiday surcharge, cap logic)
- [ ] 9.2 Implement EarningsCalculationService: compute hourly rate from salary and config
- [ ] 9.3 Implement time-range classification logic (normal, daytime OT, night surcharge, nocturnal OT)
- [ ] 9.4 Implement rest-time deduction before classification
- [ ] 9.5 Implement Sunday/holiday detection and surcharge application
- [ ] 9.6 Implement capped view (8h normal + 2h daytime OT limit)
- [ ] 9.7 Integrate earnings calculation into time record responses
- [ ] 9.8 Pass all earnings calculation tests

## 10. Time Reports Capability

- [ ] 10.1 Write tests for on-screen report endpoints (capped and uncapped) and Excel generation
- [ ] 10.2 Implement GET /reports/time (on-screen capped; employee own / admin any employee; day/week/month filter)
- [ ] 10.3 Implement GET /reports/time/uncapped (admin only; total hours without cap)
- [ ] 10.4 Implement GET /reports/time/export?cap=true|false (streams Excel .xlsx file; employee own capped / admin capped or uncapped)
- [ ] 10.5 Add custom date range filter (startDate / endDate) to all report endpoints; validate startDate ≤ endDate and max 366-day span; enforce mutual exclusivity with day/week/month filter modes
- [ ] 10.6 Add INCOMPLETE-record guard to all report endpoints: if any record in the requested period is INCOMPLETE, return HTTP 409 with list of incomplete dates; applies to on-screen and Excel, capped and uncapped
- [ ] 10.7 Add extended-hours flag to on-screen report response: per record, include highlightLevel field (none | warning | alert) based on billable hours vs. payroll config thresholds; include corrected flag and correctionReason when applicable
- [ ] 10.8 Add Notes column to Excel export: "Admin correction: [reason]" for corrected records; "Extended hours" for alert-level records
- [ ] 10.9 Write tests for extended-hours highlighting: normal day (no flag), overtime day (warning), excessive day (alert); corrected record flag in response and Excel; alert threshold edge cases
- [ ] 10.10 Write tests for incomplete-record blocking and custom date range filter
- [ ] 10.11 Pass all reports tests

## 11. Frontend - Foundation

- [x] 11.1 Set up React Router v6 with protected routes — ProtectedRoute HOC redirects to /login if no token; admin-only guard for admin routes
- [x] 11.2 Set up TanStack React Query v5 client and Axios service layer (auth.service.ts, roles.service.ts, users.service.ts); JWT request interceptor; 401 response interceptor clears token and redirects
- [x] 11.3 Implement login page (LoginPage.tsx) and AuthContext (token, user profile, role, menuOptions stored in localStorage)
- [x] 11.4 Implement role-based navigation — Sidebar renders dynamically from menuOptions returned by login response

## 12. Frontend - Admin Screens

- [x] 12.1 Role management screen — RoleListPage (list, delete), RoleFormPage (create/edit), MenuOptionAssignPage (assign menu options to role)
- [x] 12.2 User management screen — UserListPage (list, toggle status, reset password, delete), UserFormPage (create/edit with password strength indicator)
- [ ] 12.3 Payroll configuration screen (global params per year + holiday calendar)
- [ ] 12.4 Employee configuration screen (list, create, update employees with personal info and salary)
- [ ] 12.5 Admin time records screen (view capped records for any employee with visual distinction; uncapped view toggle; day/week/month/custom date range filter)
- [ ] 12.6 Admin Excel export (download capped and uncapped reports per employee)
- [ ] 12.7 Admin reopen time record UI
- [ ] 12.8 Admin incomplete-record resolution screen: list all INCOMPLETE records across all employees; form to set manual clock-out + mandatory note per record
- [ ] 12.9 Admin direct time correction screen: select employee + date; form to edit clock-in and/or clock-out with mandatory correction reason; display original values alongside new inputs; confirmation step showing diff before saving
- [ ] 12.10 Admin create record screen: for dates with no existing entry; same mandatory reason and confirmation flow

## 13. Frontend - Employee Screens

- [ ] 13.1 Clock-in / clock-out screen (single action per day; display current day status)
- [ ] 13.2 Employee time records screen (own records; day/week/month/custom date range filter; capped view with visual distinction for overtime; warning/alert highlight for extended hours; correction indicator on corrected records)
- [ ] 13.3 Employee Excel export (download own capped report with Notes column)
- [ ] 13.4 Employee dashboard: persistent INCOMPLETE-records banner listing affected dates with instruction to contact admin; banner disappears when all records are resolved
