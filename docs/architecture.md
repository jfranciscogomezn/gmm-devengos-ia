# GMM Devengos — Incremental Architecture

This document describes the **system architecture** and **module boundaries** for the GMM Devengos payroll and time-tracking application. It is maintained **incrementally**: sections reflect what is **implemented today** and what is **planned next**, without rewriting history each sprint.

**Authoritative product requirements:** `requirements/requirements-v3.md` (and Human-Usability documents under `hu/`).

**Specification-driven workflow:** OpenSpec changes under `openspec/changes/` (see `docs/openspec-incremental-features-guide.md`). The initial epic is **archived** at `openspec/changes/archive/2026-04-06-gmm-devengos-initial/`. Active implementation increments use `feature-payroll-config`, `feature-employee-config`, `feature-time-tracking`, and `feature-earnings-and-reports`.

---

## 1. Vision (Stable)

The system calculates employee **earnings (devengados)** from **recorded work time**, applying configurable payroll rules (time ranges, surcharges, caps, holidays). It provides **role-based access**, **employee self-service** (clock in/out, own reports), and **administrative configuration** (payroll parameters, employees, corrections, reports).

---

## 2. Technology Stack (As Implemented)

| Layer | Technology |
|-------|------------|
| Backend API | Java 17+, Spring Boot 3.x, Spring Web, Spring Security, Spring Data JPA |
| Persistence | PostgreSQL 15, Flyway migrations |
| API docs | OpenAPI 3 / SpringDoc |
| Auth | JWT (JJWT), bcrypt (strength 12), RBAC (`@PreAuthorize`) |
| Frontend | React 18, TypeScript, Vite, TanStack React Query, Axios, React Bootstrap |
| Local infra | Docker Compose (PostgreSQL) |

*Note: Early OpenSpec design documents may still reference Node.js/Prisma; the **implemented** stack is Java/Spring Boot + Flyway. When in doubt, trust this file and the codebase.*

---

## 3. Module Map and Build Order

Modules are **delivery increments**. Each later module **depends** on earlier ones.

```text
Phase 1 (done)     Phase 2 (planned)   Phase 3 (planned)   Phase 4 (planned)   Phase 5 (planned)
┌────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌──────────────────────────┐
│ Security       │ │ Payroll Config  │ │ Employee Config │ │ Time Tracking   │ │ Earnings + Time Reports  │
│ • Auth (JWT)   │ │ • Global params │ │ • Employee CRUD │ │ • Clock in/out  │ │ • Calculation engine     │
│ • Roles        │ │ • Holidays      │ │ • Salary + info │ │ • INCOMPLETE job│ │ • On-screen + Excel      │
│ • Users        │ │                 │ │                 │ │ • Admin correct │ │ • Filters, guards, flags │
│ • Menu options │ │                 │ │                 │ │ • Audit         │ │                          │
└───────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────┬────────┘ └────────────┬─────────────┘
        │                   │                   │                   │                       │
        └───────────────────┴───────────────────┴───────────────────┴───────────────────────┘
                                          All use shared auth + RBAC
```

| Module | Capability id (OpenSpec) | Backend prefix (typical) | Depends on |
|--------|---------------------------|---------------------------|------------|
| Security & access | `auth`, `role-management`, user management (maps from `member-management`) | `/api/v1/auth`, `/api/v1/roles`, `/api/v1/users` | — |
| Payroll configuration | `payroll-config` | `/api/v1/config/...` (TBD) | Security (ADMIN) |
| Employee configuration | `employee-config` | `/api/v1/employees` (TBD) | Security (ADMIN) |
| Time tracking | `time-tracking` | `/api/v1/time-records` (TBD) | Employees, payroll config (for rules context) |
| Earnings & reports | `earnings-calculation`, `time-reports` | calculation internal + `/api/v1/reports/...` (TBD) | Time records, payroll config, employees |

---

## 4. Implemented Today (Phase 1)

**Backend**

- Flyway: `roles`, `menu_options`, `role_menu_options`, `users`, `audit_logs`; seed roles, menu options, default admin user.
- JWT login, logout (client discard), `me`, change password.
- Role CRUD, menu assignment; user CRUD, status, password reset.
- Global exception handling, CORS for dev origins, OpenAPI.

**Frontend**

- Login, auth context, protected routes, dynamic sidebar from API.
- Admin screens: roles (list, form, menu assignment), users (list, form, status, reset password).
- Profile / change password (as implemented in repo).

**Out of scope for Phase 1 (by design)**

- Payroll config entities/APIs/UI, employee master data beyond users, time records, earnings engine, Excel reports.

---

## 5. Planned Increments (Phases 2–5)

Each phase should correspond to **one OpenSpec change** (or a deliberate split if scope is too large). Details live in that change’s `proposal.md`, `design.md`, and `tasks.md`; product rules remain in `requirements/requirements-v3.md`.

1. **Payroll config** — Year-scoped parameters, holidays, validation rules aligned with requirements.
2. **Employee config** — Employee entity separate from system `User` where required; salary and personal data; admin UI.
3. **Time tracking** — Daily clock-in/out, states (OPEN, CLOSED, INCOMPLETE), scheduled job, admin incomplete resolution, admin direct correction + audit.
4. **Earnings calculation** — Pure domain/service logic + integration into record/report payloads.
5. **Time reports** — Capped/uncapped views, custom date range, INCOMPLETE guard (409), extended-hours highlighting, Excel export with Notes column.

*Mobile app* (future): shares JWT and API; employee-facing subset per requirements; admin flows web-only.

---

## 6. Cross-Cutting Concerns

| Concern | Approach |
|---------|----------|
| Auditing | `audit_logs` table; extend for time-record corrections and admin actions per requirements |
| Errors | Consistent JSON error body (`ApiResponse` / `ErrorResponse` pattern in backend) |
| Security | Stateless JWT; method-level `@PreAuthorize`; secrets via environment variables |
| i18n | Product may be Spanish; **code and technical docs remain English** per project standards |

---

## 7. Repository Layout (Relevant)

```text
gmm-devengos/
├── backend/                 # Spring Boot API
├── frontend/                # React SPA
├── docker-compose.yml
├── requirements/          # requirements-v3.md
├── openspec/
│   ├── changes/             # Active and archived OpenSpec changes
│   └── specs/               # Canonical capability specs (when synced from changes)
└── docs/
    ├── architecture.md      # This file
    └── openspec-incremental-features-guide.md
```

---

## 8. How This Document Evolves

- After each **delivered module**, update **§4 Implemented** and **§5 Planned** (move bullets from planned to implemented).
- Do **not** duplicate full API lists here; link to OpenAPI or change-specific design.
- If stack or boundaries change, update **§2** and **§3** in the same PR as the code.

---

## Revision history

| Date | Change |
|------|--------|
| 2026-04-06 | Initial incremental architecture document (Phase 1 = security module as implemented). |
| 2026-04-07 | OpenSpec epic archived; active feature changes listed (`feature-payroll-config`, etc.). |
