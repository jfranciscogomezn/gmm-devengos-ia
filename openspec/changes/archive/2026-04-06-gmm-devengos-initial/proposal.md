## Why

A company needs a system to accurately calculate employee earnings (devengados) based on actual worked hours, differentiating between normal, overtime, night-shift, and holiday time ranges. No such system exists today — payroll calculations are done manually, which is error-prone and time-consuming.

## What Changes

- Introduce a full-stack application (backend API + frontend) from scratch.
- New security module: role-based access control with user management and menu permissions.
- New configuration module: global payroll parameters (minimum salary, transport subsidy, work-hour limits, time-range boundaries, surcharge percentages, holidays) and per-employee settings (salary, personal info).
- New time-tracking module: employees clock in/out once per day; admins can reopen records; records are immutable once closed by the employee.
- New earnings calculation engine: computes devengado per employee per day based on configured time ranges, surcharge percentages, and rest-time deductions.
- New reporting module: on-screen views and downloadable Excel reports for employees and admins, with capped (8h normal + 2h daytime overtime) and uncapped (total recorded) variants.

## Capabilities

### New Capabilities

- `auth`: User authentication (login/logout) with JWT; password stored bcrypt-hashed; access restricted by role.
- `role-management`: Create, update, delete roles; assign menu options to each role; one role per user at a time.
- `member-management`: Register members with optional user account (username + password); enable or disable user access; link member to a role.
- `payroll-config`: Configure global payroll parameters per year: minimum wage, transport subsidy, monthly/daily work hours, time-range boundaries (normal, daytime overtime, night surcharge, nocturnal overtime), surcharge percentages, non-billable rest minutes, and holiday calendar.
- `employee-config`: Configure per-employee data: personal info (names, surnames, ID type/number, email, phone) and salary.
- `time-tracking`: Employee clock-in and clock-out (once per day, immutable after clock-out unless admin reopens); admin can reopen a closed record for correction. Records not closed by the employee are automatically flagged INCOMPLETE at 00:01; employees are warned on their dashboard and can continue registering subsequent days normally, but only the admin can set a manual clock-out to resolve incomplete records. Admins can also directly overwrite clock-in and/or clock-out timestamps for any record (any status) on any date using a mandatory correction reason; the original values and reason are persisted in the audit trail.
- `earnings-calculation`: Calculate devengado per time entry based on configured time ranges and surcharge percentages, deducting non-billable rest time; cap at 8h normal + 2h daytime overtime for standard views.
- `time-reports`: On-screen and Excel reports for employees (own records) and admins (any employee); capped and uncapped variants; filterable by day, week, month, or a free-form custom date range (start date / end date) for arbitrary periods such as biweekly pay periods. Report generation is blocked (HTTP 409) when the requested period contains INCOMPLETE records. On-screen reports visually highlight records with extended hours at two severity levels (warning: overtime present; alert: hours exceed normal + max extra). Admin-corrected records display a correction indicator with reason. Excel exports include a Notes column for corrections and extended-hours flags.

### Modified Capabilities

## Impact

- **New backend API**: REST endpoints for all capabilities above (Node.js / TypeScript recommended per project standards).
- **New frontend**: React SPA consuming the API.
- **New database schema**: tables for users, roles, menu options, members, employees, payroll config, holidays, time records.
- **Dependencies**: bcrypt, JWT library, Excel generation library (e.g., exceljs), ORM (e.g., Prisma).
- **No existing code affected** — greenfield project.
