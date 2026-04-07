## Context

This is a greenfield full-stack payroll time-tracking application (`gmm-devengos`). There is no existing codebase. The system must calculate employee earnings (devengados) based on worked hours, differentiating between normal hours, daytime overtime, night surcharge hours, and nocturnal overtime, using configurable parameters per year. The stack follows project standards: Node.js + TypeScript backend (REST API), React + TypeScript frontend, relational database via Prisma ORM.

## Goals / Non-Goals

**Goals:**
- Deliver a working REST API covering all 8 capabilities: auth, role-management, member-management, payroll-config, employee-config, time-tracking, earnings-calculation, time-reports.
- Deliver a React SPA that consumes the API with role-based UI.
- Store passwords using bcrypt; secure endpoints with JWT.
- Generate downloadable Excel reports via exceljs.
- Follow TDD: unit tests for earnings calculation logic; integration tests for API endpoints.

**Non-Goals:**
- Multi-company / multi-tenant support (single company in v1).
- Payroll payslip generation or tax calculations.
- Mobile native apps.
- Real-time notifications.
- Third-party HR system integrations.

## Decisions

### 1. Backend: Node.js + Express + TypeScript
**Rationale**: Aligns with project standards and team familiarity. Express is lightweight and well-suited for a REST API of this scope.
**Alternatives considered**: NestJS (more structured but adds framework overhead for this size), Fastify (good performance but less common).

### 2. ORM: Prisma
**Rationale**: Type-safe database access, migration management, and strong TypeScript integration. Already referenced in the development guide.
**Alternatives considered**: TypeORM (heavier, less ergonomic type inference), raw SQL (too verbose, no migration tracking).

### 3. Database: PostgreSQL
**Rationale**: Relational model fits the domain well (foreign keys between roles, users, employees, time records). Matches existing Docker Compose setup in the project.

### 4. Authentication: JWT (access token, short-lived)
**Rationale**: Stateless, well-understood, easy to implement with existing libraries. No refresh token in v1 to keep scope small.
**Alternatives considered**: Session-based (requires server-side state, more complex scaling).

### 5. Password hashing: bcrypt (cost factor 12)
**Rationale**: Required explicitly in requirements. Cost factor 12 is a standard security baseline.

### 6. Excel generation: exceljs
**Rationale**: Mature library, good TypeScript support, supports streaming for large files.
**Alternatives considered**: xlsx (less actively maintained), puppeteer PDF (wrong format).

### 7. Earnings calculation: pure domain service (no DB side computation)
**Rationale**: Business logic is complex (multiple time ranges, percentages, rest deductions). Keeping it in a pure TypeScript service makes it fully unit-testable without DB dependencies.
**Alternatives considered**: DB stored procedures (hard to test, vendor lock-in).

### 8. Frontend: React + TypeScript + React Query + React Router
**Rationale**: Standard stack per frontend-standards.mdc; React Query handles server state and caching; React Router for protected routes by role.

### 9. Role-based menu: menu options stored in DB, assigned to roles
**Rationale**: Requirements explicitly state roles have assigned menu options, and access is restricted by role. A DB-driven approach allows admin configuration without redeployment.

### 10. Time record immutability: clock-out locks record; admin reopen flag
**Rationale**: Requirements state employees cannot edit records after closing. Admin reopening is modeled as a boolean flag (`reopened`) on the record, which sets it back to editable.

## Risks / Trade-offs

- **Earnings calculation complexity** → Multiple overlapping time ranges (night surcharge starts before daytime overtime ends in some configs). Mitigation: Write exhaustive unit tests per range; validate range boundaries on config save.
- **Configurable time ranges vs. fixed fiscal year** → Config is per-year but a time record spans one day. If config changes mid-year, historical records could be miscalculated. Mitigation: Snapshot the relevant config at calculation time or link records to the config version active on that date.
- **Single role per user** → Simplifies access control but limits flexibility. Accepted as v1 constraint per requirements.
- **No refresh token** → Sessions expire with the access token. Users must re-login. Acceptable for internal tool with short sessions.
- **Excel reports for large date ranges** → Large employee datasets could be slow. Mitigation: Stream the Excel file; add pagination or date-range limits as a safeguard.

## Migration Plan

1. Initialize PostgreSQL via Docker Compose.
2. Run `prisma migrate deploy` to apply schema.
3. Seed initial admin user and default role.
4. Deploy backend API.
5. Deploy frontend (served via static hosting or dev server in v1).

**Rollback**: Drop and recreate schema from migration history. No data migration needed (greenfield).

## Open Questions

- Should the system support multiple companies in a future version? (Impacts schema now if yes.)
- What is the exact non-billable rest deduction rule — fixed minutes per day or configurable per shift?
- Should the JWT expiry time be configurable or hardcoded in v1?
