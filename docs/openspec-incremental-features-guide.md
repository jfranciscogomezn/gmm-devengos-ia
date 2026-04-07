# OpenSpec: Greenfield Context + Incremental Feature Delivery

This guide explains how to use **OpenSpec** when the **product vision** spans many modules, but **delivery** happens **one feature at a time**. It targets scenarios where:

- You created (or will create) a **large initial change** with full capability list, design, and a long `tasks.md`.
- Part of the work is **already implemented** (e.g. security + roles).
- You want **each future module** to have its **own** `proposal.md`, `design.md`, and `tasks.md`, while **avoiding** duplicated or divergent product requirements.

**Related documents:** `docs/architecture.md`, `requirements/requirements-v3.md`.

---

## 1. Concepts: Three Layers

| Layer | Purpose | Single source of truth |
|-------|---------|-------------------------|
| **Product requirements** | What the business needs, acceptance criteria, HU | `requirements/requirements-v3.md` (and HU docs) |
| **System architecture** | Stack, module order, dependencies, what is shipped | `docs/architecture.md` (update after each phase) |
| **OpenSpec change** | How we deliver **one** increment: scope, technical design slice, executable tasks | `openspec/changes/<change-name>/` |

**Rule of thumb:** Product requirements stay **one place**. OpenSpec changes **reference** them (by section or capability name); they do not replace them.

---

## 2. Why One Change Per Feature

Benefits:

- **Clear context** for AI and humans: open only the change folder for “payroll-config”.
- **Task lists stay small** and completable; `npx openspec list` shows meaningful progress per change.
- **Archive** completed work without losing history; active work is not mixed with finished modules.
- **Dependencies** are explicit in each `proposal.md` (“Builds on: Phase 1 security APIs, Flyway V1–V4”).

---

## 3. Standard Structure of a Feature Change

Create a directory:

`openspec/changes/feature-<module>-<short-id>/`

Recommended files:

| File | Scope |
|------|--------|
| `.openspec.yaml` | `schema: spec-driven` (or your team schema); `created: YYYY-MM-DD` |
| `proposal.md` | **Why** this increment, **what** is in/out, **links** to `requirements-v3.md` sections, **dependencies** on prior code/migrations |
| `design.md` | **Only** what this feature adds: new tables, endpoints, schedulers, integration points, non-goals for *this* change |
| `tasks.md` | Checklist **only** for this module (numbered, testable tasks); mark done as you implement |
| `specs/<capability>/spec.md` | Delta spec for that capability (optional but recommended if you sync to `openspec/specs/`)

**Anti-pattern:** Copying the entire `requirements-v3.md` into the change. Use **references** + short excerpts if needed for offline context.

---

## 4. The Initial “Epic” Change (`gmm-devengos-initial`)

Typical lifecycle:

1. **While active:** It holds the **first** full picture (proposal, design, cross-cutting specs, big `tasks.md`).
2. **When Phase 1 is done:** Update `tasks.md` checkboxes to match reality (as you already did for security).
3. **When you stop using it for day-to-day work:** **Archive** it with OpenSpec archive workflow (moves to `openspec/changes/archive/YYYY-MM-DD-gmm-devengos-initial/`). All files, including `tasks.md`, are preserved **as-is**.

**Current repo state:** The epic is archived as `openspec/changes/archive/2026-04-06-gmm-devengos-initial/`. Active incremental changes live under `openspec/changes/feature-*`.

After archive:

- The epic remains the **historical** record of the original plan and early design.
- **New work** uses **new** changes (`feature-payroll-config`, etc.).

---

## 5. Extracting Tasks from a Large `tasks.md`

When splitting work:

1. Open the archived (or current) epic `tasks.md`.
2. **Copy only** the sections that belong to the next module (e.g. “## 6. Payroll Configuration”).
3. Paste into the new change’s `tasks.md`.
4. **Renumber** if needed (`P-1`, `P-2`, … or `6.1` → `1.1` inside the new file).
5. Add a header line: `> Derived from: openspec/changes/archive/.../tasks.md (sections 6.x) and requirements-v3.md § ...`

This keeps **traceability** without maintaining two full backlogs.

---

## 6. Keeping Design Aligned

- **Global design** (whole system) lives in the **archived epic** `design.md` and in **`docs/architecture.md`** (incremental, current truth for stack and phases).
- **Per-feature** `design.md` should:
  - State **assumptions** (“Auth and RBAC exist; use `ADMIN` for all config endpoints”).
  - Document **new** Flyway versions (`V5__...`), entities, and API paths.
  - **Not** repeat the entire auth flow unless this feature changes it.

If the epic `design.md` still says “Node + Prisma” but the code is Java + Flyway, **do not** rewrite the archive. **Update** `docs/architecture.md` and the **new** feature `design.md` to reflect reality; optionally add a one-line note in epic archive only if you edit archived files (usually unnecessary).

---

## 7. Specs: Delta vs Canonical

If your OpenSpec workflow uses **delta specs** under `changes/<name>/specs/`:

- Each **feature change** carries deltas for **its** capability (`payroll-config`, `employee-config`, …).
- On **archive**, run your team’s **sync** step (if applicable) to merge deltas into `openspec/specs/<capability>/spec.md`.

If you **do not** use sync yet:

- Delta specs in the change folder are still valuable as **scoped** documentation.
- Plan to introduce `openspec/specs/` when the first module after the epic is archived.

---

## 8. Naming Conventions

| Item | Convention |
|------|------------|
| Change folder | `feature-payroll-config`, `feature-employee-config`, `feature-time-tracking`, `feature-earnings-and-reports` |
| Capability folders under `specs/` | Match OpenSpec proposal: `payroll-config`, `employee-config`, `time-tracking`, `earnings-calculation`, `time-reports` |
| Tasks sections | Use clear headers: `## Database`, `## API`, `## Frontend`, `## Tests` |

Use **kebab-case** for change folder names; keep **capability** names consistent with existing `openspec/changes/gmm-devengos-initial/specs/` layout.

---

## 9. Workflow Checklist: Starting a New Feature

1. [ ] Read `requirements/requirements-v3.md` for this module only.
2. [ ] Read `docs/architecture.md` — confirm phase order and dependencies.
3. [ ] Create `openspec/changes/feature-<name>/` with `.openspec.yaml`, `proposal.md`, empty `tasks.md` scaffold.
4. [ ] Write `proposal.md`: scope, links, dependencies, out-of-scope for this change.
5. [ ] Write `design.md`: schema, endpoints, schedulers, integration with existing modules.
6. [ ] Populate `tasks.md` from epic backlog slice + new items (migrations, tests, UI).
7. [ ] Add or update `specs/<capability>/spec.md` if using spec-driven deltas.
8. [ ] Run `npx openspec status --change "feature-<name>" --json` and fix missing artifacts.
9. [ ] Create a **Git feature branch**; implement with TDD; tick tasks `[x]` as you go. Open a **PR** and wait for **maintainer approval** before merge (`ai-specs/specs/git-workflow.mdc`).
10. [ ] When done: archive the change (optional sync), then update `docs/architecture.md` §4/§5.

---

## 10. Using the CLI

On Windows, if `openspec` is not on `PATH`, use:

```bash
npx openspec list --json
npx openspec status --change "feature-payroll-config" --json
npx openspec instructions apply --change "feature-payroll-config" --json
```

Use **one active change** per parallel stream of work; avoid editing two features in the same change folder.

---

## 11. Git and Multiple Repositories

If `backend` and `frontend` are **separate** Git repositories:

- Either add a **monorepo** that contains `openspec/`, `docs/`, and `requirements/`, **or**
- Duplicate **only** the minimal pointer (e.g. link in README) and keep OpenSpec in the repo that is the **system** source of truth.

Best practice: **one** repo that owns `openspec/` + `docs/` + `requirements/` so the spec and code references stay together.

---

## 12. Anti-Patterns

| Anti-pattern | Why it hurts | Instead |
|--------------|--------------|---------|
| One giant active `tasks.md` for all modules | Hard to track progress; unclear “done” per release | One `tasks.md` per change |
| Full copy of requirements in each change | Drift; conflicting edits | Reference `requirements-v3.md` by section |
| New feature proposal without “Builds on” | Broken assumptions, duplicate auth work | Explicit dependencies |
| Archiving without updating `docs/architecture.md` | Team thinks Node/Prisma is current | Update architecture doc when stack or phase changes |
| Skipping tests in `tasks.md` | Regressions in calculation/reporting | Always include test tasks per module |

---

## 13. Summary

- **Product truth:** `requirements/requirements-v3.md`.
- **Engineering phases and stack:** `docs/architecture.md` (incremental).
- **Delivery unit:** one OpenSpec **change** per feature, with its own `proposal`, `design`, and `tasks`.
- **Epic change:** archive when you pivot to per-feature changes; it remains the historical umbrella.
- **Always** link proposals to requirements and prior phases so new work **complements** what is already built.

---

## Revision history

| Date | Change |
|------|--------|
| 2026-04-06 | Initial guide for greenfield + incremental OpenSpec workflow. |
