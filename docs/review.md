# Code Review System

## Purpose

This document defines how the playbook is enforced during real development.

It covers the PR checklist, architecture validation, anti-pattern detection and
cross-module consistency.

Review enforces the principles and layer docs. It does not restate them; it
points to them.

---

# References

Architecture validation:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain modeling:
[docs/principles/ddd.md](principles/ddd.md)

Design rules and functional model:
[docs/principles/solid.md](principles/solid.md) ·
[docs/principles/functional_programming.md](principles/functional_programming.md) ·
[docs/principles/composition.md](principles/composition.md)

---

# PR Checklist

Every pull request should confirm:

- new behavior has tests (happy path, failures, edge cases) — see
  [testing.md](testing.md)
- `mix format`, `mix compile --warnings-as-errors` and `mix test` pass
- public APIs are documented (`@moduledoc`, `@doc`, `@spec`)
- naming and idioms follow [coding_guidelines.md](coding_guidelines.md)
- no duplicated concept definitions (principles remain the single source)

---

# Architecture Validation Rules

Verify dependencies point inward:

- domain has no Phoenix, Ecto, HTTP or process concerns
- persistence stays behind adapters — see [ecto.md](ecto.md)
- delivery stays thin — see [phoenix.md](phoenix.md)
- external dependencies sit behind behaviours
- runtime processes map to failure domains — see [otp.md](otp.md)

If a change makes the domain depend on infrastructure, reject it.

---

# Anti-pattern Detection

Watch for:

- business logic in controllers, schemas, changesets or GenServer callbacks
- `Ecto.Query` or `Repo` outside adapters
- god modules and generic `Manager`/`Utils` modules
- shared mutable state across processes
- rescuing errors that should crash
- secrets or PII in logs — see [security.md](security.md) and
  [observability.md](observability.md)

Each layer doc's Anti-patterns section is the reference list.

---

# Consistency Rules Across Modules

Across the codebase, confirm:

- the same concept is named the same way (ubiquitous language)
- error tuples use explicit, consistent reasons
- boundaries validate consistently (shape at delivery, rules in domain)
- telemetry and logging follow one convention — see [observability.md](observability.md)

Consistency prevents architectural drift as the system grows.

---

# Review Mindset

A review protects the system, not the author.

Ask:

- Does this respect layer boundaries?
- Is the concept defined once, in the principles?
- Is it tested, readable and idiomatic?
- Would this scale and fail safely?

Prefer clarity and correctness over cleverness.

---

# Checklist

Before approving a PR:

- Tests cover new behavior and failures
- Quality gates pass with no warnings
- Layer boundaries are respected
- No duplicated principle definitions
- No known anti-patterns introduced
- Naming and errors are consistent

---

# Related Documents

- [testing.md](testing.md)
- [coding_guidelines.md](coding_guidelines.md)
- [architecture.md](architecture.md)
