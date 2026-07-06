# Elixir Engineering Playbook — Bootstrap

Read this first when consuming this playbook from another project (via a
sibling repo path, git submodule/subtree, or any other link).

## What this is

A two-tier canonical documentation set for building Elixir/Phoenix systems on
the BEAM at production-grade (financial-system-grade) depth.

## Load order

1. `README.md` — the doc index and the two-tier canonical model:
   - **Principles** (`docs/principles/*`) — theory (what/why).
   - **Engineering Framework** (`docs/otp.md`, `docs/concurrency.md`,
     `docs/error_handling.md`, `docs/observability.md`,
     `docs/performance.md`, `docs/security.md`, `docs/ecto.md`) —
     operational rules (how), canonical to the layer each doc names.
2. `.agent/contract.md` — the no-duplication / canonical-ownership rules that
   govern this playbook's own docs.
3. The specific principle or Engineering Framework doc relevant to the task
   at hand — read on demand, not all up front.

## How a consuming project should treat this playbook

- A consuming project's own docs may add project-specific detail (which
  adapter implements a port, which provider is integrated, which bounded
  context owns what) but must not redefine a concept already canonical here
  — OTP process design, error taxonomy, concurrency safety, persistence
  boundary rules, security boundaries, performance rules. Reference the
  canonical doc instead of restating it.
- If a local convention conflicts with a rule in this playbook, resolve the
  conflict explicitly — update the local convention, or raise it upstream
  here if the playbook is genuinely wrong for a real reason. Do not silently
  diverge.
- `scripts/validate-playbook.md` describes how to audit this playbook's own
  docs for duplication and drift. The same three checks (duplicate concepts,
  forbidden re-definition, reference enforcement) apply when auditing a
  consuming project's docs against this playbook.
