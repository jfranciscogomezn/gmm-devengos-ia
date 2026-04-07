# gmm-devengos-ia

Learning and **specification** material for the **GMM Devengos** project: OpenSpec changes, product requirements, architecture notes, AI agent commands, Cursor rules/skills, and team standards.

This repository **does not** contain application source code (`backend/`, `frontend/`). Use it to review **how the product is specified**, **how OpenSpec is split by feature**, and **how agents are instructed** to work (including Git branching and PR policy).

## Contents

| Path | Purpose |
|------|---------|
| `openspec/` | OpenSpec changes: archived epic + active `feature-*` increments (`proposal`, `design`, `tasks`, delta `specs`) |
| `requirements/` | Product requirements (`requirements-v3.md`) |
| `docs/` | Incremental architecture (`architecture.md`) and OpenSpec workflow guide (`openspec-incremental-features-guide.md`) |
| `ai-specs/specs/` | `base-standards.mdc`, `git-workflow.mdc`, backend/frontend/documentation standards |
| `ai-specs/.commands/` | Cursor / Claude slash-style command definitions |
| `.cursor/` | Cursor rules, skills (e.g. OpenSpec apply/archive), commands |

## OpenSpec CLI

From a machine with Node.js:

```bash
npx openspec list --json
npx openspec status --change "feature-payroll-config" --json
```

## Related repositories

Application code may live in separate Git repositories (e.g. `gmm-devengos-backend`, `gmm-devengos-frontend`). Keep this repo as the **single place** for cross-cutting specs and AI workflow if those repos do not include `openspec/` and `requirements/`.

## Entry points for humans

1. Read `docs/architecture.md` for phases and stack.
2. Read `docs/openspec-incremental-features-guide.md` for feature-by-feature OpenSpec practice.
3. Open `openspec/changes/archive/2026-04-06-gmm-devengos-initial/` for the original epic snapshot.
4. Open each `openspec/changes/feature-*/` for the next implementation increments.

See `CONTRIBUTING.md` for Git branching and pull request rules.
