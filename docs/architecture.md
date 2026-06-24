# GMM Devengos — Arquitectura Incremental v3

This document describes the **system architecture** and **module boundaries** for the GMM Devengos platform, which covers two product domains: **Payroll & Time Management** and **Operations & Field Tracking**. It is maintained **incrementally**: sections reflect what is **implemented today** and what is **planned next**, without rewriting history each sprint.

> **v3 — SaaS pivot:** the platform is now a **multi-tenant SaaS**. Multiple client companies (**tenants**) share a single deployment (one backend, one PostgreSQL cluster, one object-storage bucket) with **strict per-tenant data isolation**. The tenancy model is **Pool** (shared schema + `tenant_id` discriminator + PostgreSQL Row-Level Security), with the tenant resolved from a JWT claim. See **§10 Multi-tenancy model** and OpenSpec change `feature-multi-tenancy`.

**Authoritative product requirements:** `requirements/requirements-v5.md`

**Specification-driven workflow:** OpenSpec changes under `openspec/changes/` (see `docs/openspec-incremental-features-guide.md`). Completed increments are **archived** under `openspec/changes/archive/`; canonical capability requirements live in `openspec/specs/` (payroll, security, i18n, multi-tenancy, modular menu, and related capabilities). There are **no active** OpenSpec changes at present; new operations-domain increments should be proposed when those phases begin.

---

## 1. Vision (Stable)

The system is a **unified, multi-tenant SaaS platform** sharing a single backend REST API, JWT authentication, and mobile application across two domains. Every client company is a **tenant**; all tenant data is isolated within the shared infrastructure (see **§10**). The two product domains are:

| Domain | Purpose |
|--------|---------|
| **Payroll & Time** | Calculate employee earnings from recorded work time applying configurable Colombian payroll rules (time ranges, surcharges, caps, holidays). Provides role-based access, employee self-service (clock in/out, own reports), and administrative configuration (payroll parameters, employees, corrections, reports). |
| **Operations & Field Tracking** | Centralize tracking of internal service orders (OSI) for transport: full internal traceability, field log with evidence (photos, notes), GPS reference, and managed client-facing communication (client portal, digest). |

Both domains share authentication, the same users/roles/menu system, the same audit mechanism, the same mobile app (menu gated by role), and the same notification infrastructure. All of these are **tenant-scoped**: users, roles, audit entries, and data belong to exactly one tenant.

---

## 2. Technology Stack (As Implemented)

| Layer | Technology |
|-------|------------|
| Backend API | Java 17+, Spring Boot 3.x, Spring Web, Spring Security, Spring Data JPA |
| Persistence | PostgreSQL 15, Flyway migrations |
| API docs | OpenAPI 3 / SpringDoc |
| Auth | JWT (JJWT) with a signed `tenant_id` claim, bcrypt (strength 12), RBAC (`@PreAuthorize`) |
| Multi-tenancy | Pool model: `tenant_id` discriminator + Hibernate tenant `@Filter` + PostgreSQL Row-Level Security; request-scoped `TenantContext` |
| Frontend | React 18, TypeScript, Vite, TanStack React Query, Axios, React Bootstrap |
| Local infra | Docker Compose (PostgreSQL) |
| File storage | Blob/object storage with **per-tenant key prefix** (`{tenant_id}/…`) — **planned for operations phases** |

*Note: Early OpenSpec design documents may still reference Node.js/Prisma; the **implemented** stack is Java/Spring Boot + Flyway. When in doubt, trust this file and the codebase.*

---

## 3. Module Map and Build Order

Modules are **delivery increments**. Each later module depends on earlier ones within the same dependency chain.

```text
┌─────────────────────────────────────────────────────────────────────────────────┐
│              CROSS-CUTTING: Multi-tenancy (SaaS) — tenant_id + RLS               │
│  Tenants · Tenant context (JWT claim) · Row-Level Security · Platform admin      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                  CROSS-CUTTING: Security & Access (Phase 1 — DONE)              │
│  Auth (JWT) · Roles · Users · Menu options · Audit log base · RBAC              │
└──────────────────────────────────┬──────────────────────────────────────────────┘
                                   │
           ┌───────────────────────┴──────────────────────────┐
           │ PAYROLL DOMAIN                                    │ OPERATIONS DOMAIN
           │                                                   │
           ▼ Phase 2                                           ▼ Phase 6
    ┌─────────────────┐                               ┌─────────────────────────┐
    │ Payroll Config  │                               │ Operations Master Data  │
    │ • Global params │                               │ • Client master         │
    │ • Holidays      │                               │ • Vehicle catalogue     │
    └────────┬────────┘                               │ • GPS reference (MVP)   │
             │ Phase 3                                └───────────┬─────────────┘
             ▼                                                    │ Phase 6 (cont.)
    ┌─────────────────┐                               ┌───────────▼─────────────┐
    │ Employee Config │                               │  OSI Management (web)   │
    │ • Employee CRUD │                               │  • OSI CRUD             │
    │ • Salary + docs │                               │  • Vehicle assignment   │
    └────────┬────────┘                               │  • Status state machine │
             │ Phase 4                                │  • Transport documents  │
             ▼                                        └───────────┬─────────────┘
    ┌─────────────────┐                                           │ Phase 7
    │ Time Tracking   │                               ┌───────────▼─────────────┐
    │ • Clock in/out  │                               │  Field Log (bitácora)   │
    │ • INCOMPLETE job│                               │  • Event types config   │
    │ • Admin correct │                               │  • Events + evidence    │
    │ • Audit         │                               │  • Mobile offline queue │
    └────────┬────────┘                               │  • Policy A (external)  │
             │ Phase 4/5                              │  • Visibility approval  │
             ▼                                        └───────────┬─────────────┘
    ┌─────────────────┐                                           │ Phase 8
    │ Earnings +      │                               ┌───────────▼─────────────┐
    │ Time Reports    │                               │  Client Portal + Digest │
    │ • Calc engine   │                               │  • Revocable token link │
    │ • Screen + Excel│                               │  • Client timeline view │
    │ • Filters, caps │                               │  • Digest text generator│
    └─────────────────┘                               └─────────────────────────┘
```

### Module registry

| # | Module | Capability id (OpenSpec) | Backend prefix (typical) | Domain | Depends on | Phase |
|---|--------|--------------------------|--------------------------|--------|------------|-------|
| 0 | Multi-tenancy (SaaS) | `multi-tenancy` | `/api/v1/platform/...` (provider) | Cross-cutting | Security | 1.5 (**prerequisite for all data modules**) |
| 1 | Security & access | `auth`, `role-management`, `user-management` | `/api/v1/auth`, `/api/v1/roles`, `/api/v1/users` | Cross-cutting | Multi-tenancy | 1 (**done, single-tenant; tenant-scoped in Phase 1.5**) |
| 2 | Audit | `audit` | `/api/v1/audit` | Cross-cutting | Security | 1 (base done; extended in later phases) |
| 3 | Payroll configuration | `payroll-config` | `/api/v1/config/...` | Payroll | Security (ADMIN) | 2 |
| 4 | Employee configuration | `employee-config` | `/api/v1/employees` | Payroll | Security (ADMIN) | 3 |
| 5 | Time tracking | `time-tracking` | `/api/v1/time-records` | Payroll | Employees, Payroll config | 4 |
| 6 | Earnings calculation | `earnings-calculation` | internal service | Payroll | Time records, Payroll config, Employees | 4/5 |
| 7 | Time reports | `time-reports` | `/api/v1/reports/...` | Payroll | Earnings calc | 5 |
| 8 | Operations master data | `ops-master-data` | `/api/v1/ops/clients`, `/api/v1/ops/vehicles` | Operations | Security (COORDINATOR) | 6 |
| 9 | OSI management | `osi-management` | `/api/v1/osi` | Operations | Ops master data | 6 |
| 10 | Field log | `field-log` | `/api/v1/osi/{id}/events` | Operations | OSI management | 7 |
| 11 | Client portal | `client-portal` | `/api/v1/portal/...` (public, token-gated) | Operations | Field log | 8 |
| 12 | Digest | `digest` | `/api/v1/osi/{id}/digest` | Operations | Field log | 8 |
| 13 | Notifications | `notifications` | `/api/v1/notifications` | Cross-cutting | Auth, Time tracking, Field log | 4 / 7 |

---

## 4. Implemented Today (Phases 1–5 complete)

> Last updated: 2026-06-17. All payroll-domain phases are delivered and merged. Operations domain not yet started.

### Cross-cutting (✅ Done)

**Security & access (Phase 1, `stepcore-security-backend`)**
- Flyway V1–V4: `roles`, `menu_nodes`, `role_menu_nodes`, `users`, `audit_logs`; seed roles, default admin.
- JWT login (`tenantSlug` + email + password), logout, `me`, change password; bcrypt, `@PreAuthorize` RBAC.
- Role CRUD + hierarchical menu-node assignment (MODULE/GROUP/ITEM tree); user CRUD, status, password reset.
- `PLATFORM_ADMIN` + Platform Administration surface (`/api/v1/platform/**`): tenant CRUD, provisioning, plan/status lifecycle.

**Multi-tenancy (Phase 1.5, OpenSpec `feature-multi-tenancy`)**
- Flyway V5–V8: `tenants` table; `tenant_id` retrofitted onto `users`/`roles`/`audit_logs`; composite unique constraints; Row-Level Security policies; dedicated `stepcore_app` runtime role (no BYPASSRLS).
- Request-scoped `TenantContext`; Hibernate `@Filter` auto-enabled; `TenantRlsAspect` sets `app.current_tenant` per transaction.
- Plan-based `max_users` enforcement; `TenantStorageKeyResolver` for per-tenant object-storage prefix.
- Frontend: `tenantSlug` field on login, tenant info in `AuthContext`/Sidebar, Platform admin area, user-cap UX.

**Modular menu (Phase 1.5, OpenSpec `feature-modular-menu`)**
- Flyway V9–V10: `menu_nodes` MODULE/GROUP/ITEM catalogue; `role_menu_nodes`; migration from legacy `menu_options`; leaf-code continuity.
- `MenuTreeService`: builds role-specific tree + flat `permissions` list returned in `LoginResponse`.
- Frontend: collapsible sidebar with recursive MODULE/GROUP/ITEM; hierarchical tree picker in role assignment UI.

**i18n (OpenSpec `feature-i18n-web` + `feature-i18n-api-messages`)**
- Frontend: `es-CO` default, `en-US` switch, `localStorage` persistence, language switcher on login and app shell.
- Both backends: `Accept-Language` locale filter, `MessageSource` bundles, localized exception messages.

### Payroll domain (✅ Done, Phases 2–5)

**Payroll config (Phase 2, OpenSpec `feature-payroll-config` + `business-payroll-config-service`)**
- `stepcore-business-backend`: standalone Spring Boot service sharing the same PostgreSQL database; own Flyway history; stateless JWT resource-server; Payroll config CRUD per year + holiday calendar; full RLS tenant isolation.
- Frontend: `/admin/config` — year selector, parameters form, holiday calendar.

**Employee config (Phase 3, OpenSpec `feature-employee-config`)**
- Backend: `employees` table (personal info, salary); tenant-scoped; full CRUD.
- Frontend: `/admin/employees` — list, create, update.

**Time tracking (Phase 4, OpenSpec `feature-time-tracking`)**
- Backend: `time_records` (OPEN/CLOSED/INCOMPLETE); clock-in/out; scheduled INCOMPLETE job; admin reopen, resolve-incomplete, direct correction + audit trail; tenant-scoped with RLS.
- Frontend: `/my/time` (employee), `/admin/time` (admin: custom date range, reopen, correct, resolve); INCOMPLETE banner on dashboard.
- _Gap (deferred):_ admin time list lacks capped/uncapped earnings toggle (covered in `/reports`); partial test coverage on auto-flagging job edge cases.

**Earnings calculation (Phase 4/5, OpenSpec `feature-earnings-and-reports`)**
- Backend: `EarningsCalculationService` — hourly rate, time-range classification, rest deduction, Sunday/holiday surcharge, capped view; computed on read (no persisted earnings table); integrated into time-record and report DTOs.

**Time reports (Phase 5, OpenSpec `feature-earnings-and-reports`)**
- Backend: `GET /reports/time` (capped, admin + employee), `GET /reports/time/uncapped` (admin), Excel export capped/uncapped; custom date range; INCOMPLETE guard (HTTP 409); `highlightLevel` + `corrected` flags; Notes column in Excel.
- Frontend: `/reports` (admin), `/my/reports` (employee); period filters; uncapped toggle; Excel export.

**Time-record audit trail (OpenSpec `feature-time-record-audit-trail`)**
- Backend: writes to `audit_logs` on admin corrections; filtered read API; tenant-scoped.
- Frontend: `/admin/time/audit` — audit history page.

**UI refresh (feat/ui-refresh, PR [gmm-devengos-frontend #12](https://github.com/jfranciscogomezn/gmm-devengos-frontend/pull/12) — pending merge)**
- Warm Enterprise visual refresh: login sidebar branding (centered, hex logo, gold skyline), updated dashboard shell, sidebar, footer, UI primitives, shared `theme.css`.

### Operations domain (⏳ Not started)

- Modules 9–15 of `requirements/requirements-v5.md` (OSI, field log, client portal, digest, GPS, notifications).
- No OpenSpec changes, no code.

---

## 5. Planned Increments

Each phase corresponds to **one or more OpenSpec changes**. Details live in each change's `proposal.md`, `design.md`, and `tasks.md`. Product rules live in `requirements/requirements-v5.md`.

### Payroll polish (optional, before operations)

- **Admin time list polish** — capped/uncapped earnings view in `/admin/time`, closing gap T2.1 from `feature-time-tracking`. Scope: frontend-only + tests. OpenSpec change: `feature-time-tracking-admin-polish`.

### Operations domain (Phases 6–8, next up)

6. **Operations master data + OSI (Phase 6, P0 MVP)** — Client master with inline create; vehicle catalogue with plate normalization and inline create; OSI CRUD (web only); vehicle assignment with state machine per (OSI, vehicle) unit; transport documents; GPS reference (manual URL/ID); operations dashboard. Aligned with requirements `§9`, `§10`, `§14`.
7. **Field log + Mobile operations (Phase 7, P0/P1)** — Configurable event types with visibility; event creation (web + mobile); photo/file attachments with size limits; policy A for external operators; client visibility approval by OSI owner; offline queue with `Idempotency-Key`; load measurement form; operational notifications (push, in-app, email). Aligned with requirements `§11`, `§15`.
8. **Client portal + Digest (Phase 8, P1)** — Revocable token link; public client timeline (only `CLIENTE`/approved events); coordinator preview; digest text generator with editable template; access log. Aligned with requirements `§12`, `§13`.

### Future roadmap (Phase 9+)

- GPS API integration (provider API or periodic ingestion enriching the field log).
- Plan vs real analytics and reporting.
- Automatic SMTP digest delivery.
- ZIP evidence export aligned to OneDrive folder structure.
- HC validation bloqueos and commercial role extended permissions.

*Mobile app* shares JWT and API; employee-facing payroll subset per `requirements §3.1.6`; operations subset for `OPERADOR_CAMPO` per same section; admin and coordinator flows web-only.

---

## 6. Cross-Cutting Concerns

| Concern | Approach |
|---------|----------|
| **Multi-tenancy** | Pool model: every tenant-owned table carries a non-null `tenant_id`. The tenant is resolved from a signed JWT claim into a request-scoped `TenantContext`. The `menu_options` feature catalogue and the `tenants` table itself are global. |
| **Tenant isolation** | Defense in depth: (1) Hibernate tenant `@Filter` auto-applies `tenant_id = :currentTenant`; (2) PostgreSQL **Row-Level Security** policies (`USING (tenant_id = current_setting('app.current_tenant'))`) guarantee isolation even if an application query omits the filter. The application connects with a non-`BYPASSRLS` DB role. |
| **Tenant lifecycle** | `PLATFORM_ADMIN` (provider) manages tenants via `/api/v1/platform/**` (create/provision/suspend/activate, plan + user cap). Provisioning seeds default roles, menu assignments, and an initial tenant admin. Suspended tenants cannot log in. |
| **Per-tenant uniqueness** | Previously global unique keys become composite with `tenant_id` (e.g. `users (tenant_id, email)`, `roles (tenant_id, name)`); the same email may exist in different tenants. |
| **Auditing** | `audit_logs` table extended incrementally by phase; covers both payroll and operations entity types; immutable entries; **tenant-scoped** (`tenant_id` on every entry). |
| **Errors** | Consistent JSON error body (`ApiResponse` / `ErrorResponse` pattern in backend). |
| **Security** | Stateless JWT carrying the `tenant_id` claim; method-level `@PreAuthorize`; tenant isolation enforced by RLS (not by RBAC); secrets via environment variables; operations portal uses long-lived revocable token (no user auth), still scoped to its tenant. |
| **i18n** | Product UI may be Spanish; **all code and technical docs remain English** per project standards. |
| **Domain isolation** | Operations field log events **must never** read or write payroll time records, earnings, or employee salary. These are separate aggregate boundaries enforced at service layer. |
| **Role expansion** | Operations roles (`COORDINADOR_OPERACIONES`, `OPERADOR_CAMPO`, `VISUALIZADOR_OPERACIONES`, `COMERCIAL`, `HC_VALIDADOR`) are seeded via Flyway alongside existing payroll roles. Menu options for both domains are configured independently. |
| **Employee ≠ Operator** | `employee` flag (payroll domain) and `OPERADOR_CAMPO` role (operations domain) are independent user configuration dimensions. A user can be one, both, or neither. The operations log never modifies payroll clock entries. |
| **Mobile (shared binary)** | Single mobile app; menu and feature set gated by role per `requirements §3.1.6`. |
| **Notifications** | Single notification infrastructure (push, in-app, email); triggers from both domains (incomplete time records → admin; new OSI assignment → operator, etc.). |
| **Offline** | Mobile offline queue used for both time-record clock actions and field-log events; different data types, same pattern. |
| **File storage** | Blob/object storage as legal source of truth for operational attachments, partitioned by **per-tenant key prefix** (`{tenant_id}/…`) derived from `TenantContext`, never from client input. OneDrive mirror via export only. |

---

## 7. System Context Diagram

```text
                  ┌───────────────────────────────────────────────┐
                  │          Web Browser (full platform)           │
                  │  Admin / Coordinator / Reports / OSI / Audit   │
                  └────────────────────┬──────────────────────────┘
                                       │
                                       │  REST + JWT
┌──────────────────────┐               │               ┌────────────────────────┐
│   Mobile App         │───────────────┼───────────────│  Client Tracking Portal │
│  Employee: clock     │               │               │  (token, public, read)  │
│  Operator: OSI log   │               │               └────────────────────────┘
│  (role-gated menu)   │               │
└──────────────────────┘               ▼
                              ┌────────────────────┐
                              │   Backend (single)  │
                              │  Spring Boot 3.x    │
                              │  Modules:           │
                              │  • Auth / RBAC      │
                              │  • Payroll (v3)     │
                              │  • Operations (v4+) │
                              │  • Audit            │
                              │  • Notifications    │
                              └──────────┬──────────┘
                                         │
               ┌─────────────────────────┴──────────────────────────┐
               │                                                     │
     ┌─────────▼─────────┐                               ┌──────────▼──────────┐
     │   PostgreSQL 15    │                               │   Blob / S3 storage  │
     │   (Flyway mgmt,    │                               │   (operational files,│
     │    RLS by tenant)  │                               │   {tenant_id}/ prefix)│
     └────────────────────┘                               └─────────────────────┘
```

> **Tenant dimension:** every authenticated request carries a `tenant_id` (JWT claim). All reads/writes to PostgreSQL are confined to that tenant by Row-Level Security, and all storage objects live under the tenant's key prefix. The provider operates a separate **Platform Administration** surface (`/api/v1/platform/**`, `PLATFORM_ADMIN`) that runs outside tenant scope to manage tenants.

---

## 8. Repository Layout (Relevant)

```text
gmm-devengos/
├── backend/                    # Spring Boot API
├── frontend/                   # React SPA
├── docker-compose.yml
├── requirements/
│   ├── requirements-v5.md      # Authoritative unified requirements (v3 + v4 consolidated)
│   ├── requirements-v3.md      # Archived: payroll domain (superseded by v5)
│   ├── requirements-v4-operations-field-tracking-supplement.md  # Archived: operations supplement (superseded by v5)
│   └── diagrams/
│       └── osi-vehicle-state-machine.mmd   # State machine for (OSI, vehicle) unit
├── openspec/
│   ├── changes/                # Active and archived OpenSpec changes
│   └── specs/                  # Canonical capability specs
└── docs/
    ├── architecture.md         # This file (v2 — unified two-domain architecture)
    └── openspec-incremental-features-guide.md
```

---

## 9. How This Document Evolves

- After each **delivered phase**, update **§4 Implemented** and **§5 Planned** (move bullets from planned to implemented).
- Do **not** duplicate full API lists here; link to OpenAPI or change-specific design documents.
- If the tech stack or domain boundaries change, update **§2** and **§3** in the same PR as the code.
- The `requirements/requirements-v5.md` document is the single authoritative source for all product rules; this architecture document references it but does not duplicate functional requirements.

---

## 10. Multi-tenancy Model (SaaS)

The platform is a **multi-tenant SaaS** using the **Pool** model: all tenants share one database, one schema, and one object-storage bucket; isolation is logical, enforced on every row.

| Aspect | Decision |
|--------|----------|
| **Tenancy model** | Pool — shared DB + shared schema + `tenant_id` discriminator. (Bridge/schema-per-tenant and Silo/DB-per-tenant are explicit non-goals; can be added later for premium clients without changing the app contract.) |
| **Tenant identity** | `tenants` table (global): `id` (BIGINT identity; reserved `1`=platform, `2`=legacy), `name`, `slug` (unique), `plan` (`STANDARD`/`PREMIUM`), `max_users`, `status` (`PROVISIONING`/`ACTIVE`/`SUSPENDED`). |
| **Discriminator** | `tenant_id BIGINT NOT NULL` (FK → `tenants`) on every tenant-owned table. Global tables: `menu_options`, `tenants`, `flyway_schema_history`. |
| **Tenant resolution** | Signed `tenant_id` (+ `tenant_slug`, `tenant_plan`) claim in the JWT, set at login from the user's tenant. A `TenantContextFilter` (after JWT auth) populates a request-scoped `TenantContext`. Client-supplied tenant values are never trusted. |
| **Isolation — layer 1** | Hibernate `@FilterDef`/`@Filter` ("tenantFilter") auto-applies `tenant_id = :currentTenant`; `@PrePersist` stamps `tenant_id` from `TenantContext`. |
| **Isolation — layer 2** | PostgreSQL Row-Level Security on every tenant-owned table; `SET LOCAL app.current_tenant` per transaction; app uses a non-`BYPASSRLS` role so RLS is authoritative. |
| **Uniqueness** | Composite with tenant: `users (tenant_id, email)`, `roles (tenant_id, name)`, etc. |
| **Provider plane** | `PLATFORM_ADMIN` manages tenants via `/api/v1/platform/**` (create/provision/suspend/activate, plan + cap). Runs outside tenant scope. |
| **Provisioning** | Creating a tenant seeds its default roles, menu assignments, and initial `ADMIN` user, transactionally. |
| **Plan limits** | User creation enforces the tenant `max_users`; exceeding it → HTTP 409 `USER_LIMIT_REACHED`. |
| **Storage** | Object keys prefixed `{tenant_id}/…` within the shared bucket. |
| **Migration** | Flyway (after V4) creates `tenants`, backfills existing Phase-1 data into a default (legacy) tenant, adds `tenant_id` + composite uniques + RLS. |

Authoritative delivery detail: OpenSpec change **`feature-multi-tenancy`** (`proposal.md`, `design.md`, `tasks.md`, `specs/multi-tenancy/spec.md`). Product rules: `requirements/requirements-v5.md` **Módulo 0**.

---

## Revision History

| Date | Change |
|------|--------|
| 2026-04-06 | Initial incremental architecture document (Phase 1 = security module as implemented). |
| 2026-04-07 | OpenSpec epic archived; active payroll feature changes listed. |
| 2026-05-24 | **v2 — Unified two-domain architecture.** Added Operations & Field Tracking domain (OSI, field log, client portal, GPS, digest) alongside existing Payroll & Time domain. Updated module map (Phases 1–8+), cross-cutting concerns, system context diagram, and repository layout. Requirements authority updated to `requirements-v5.md`. |
| 2026-05-29 | **v3 — Multi-tenant SaaS pivot.** Added Pool multi-tenancy (Module 0 / Phase 1.5): `tenant_id` + Hibernate filter + PostgreSQL Row-Level Security, JWT tenant claim, platform administration, provisioning, plan-based user caps, per-tenant storage prefixes. Updated vision, stack, module map/registry, cross-cutting concerns, system context diagram; added §10. OpenSpec: `feature-multi-tenancy`. |
