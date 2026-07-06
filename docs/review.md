# Code Review System

## Purpose

This document defines how the playbook is enforced during real development.

It covers the PR checklist, architecture validation, anti-pattern detection,
cross-module consistency and review risk tiering.

Review enforces the principles and the Engineering Framework docs (`otp.md`,
`concurrency.md`, `error_handling.md`, `observability.md`, `performance.md`,
`security.md`, `ecto.md`). It does not restate them; it points to them. The
only content genuinely owned here is the review *process* — what to check and
when a change needs more scrutiny than a routine PR.

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
- no duplicated concept definitions (principles and Engineering Framework docs
  remain the single source)
- a migration is additive and backward-compatible, per
  [ecto.md](ecto.md#migration-safety)
- a change that reads-then-writes a shared row has database-enforced
  concurrent-write safety, per [ecto.md](ecto.md#concurrent-write-safety)

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
- authorization enforced only at the edge, or a missing rate limit on a
  money-touching endpoint — see
  [security.md](security.md#rate-limiting-and-abuse-boundaries)
- a compliance-relevant state change with no audit event — see
  [observability.md](observability.md#audit-trail)

Each layer doc's Anti-patterns section is the reference list.

---

# Consistency Rules Across Modules

Across the codebase, confirm:

- the same concept is named the same way (ubiquitous language — see
  [principles/ddd.md](principles/ddd.md#ubiquitous-language))
- error tuples use explicit, consistent reasons, per
  [error_handling.md](error_handling.md#error-representation)
- boundaries validate consistently (shape at delivery, rules in domain)
- telemetry and logging follow one convention — see [observability.md](observability.md)

Consistency prevents architectural drift as the system grows.

---

# Review Risk Tiering

Not every PR carries the same risk. A routine PR gets a routine review; the
following categories need a second, more deliberate pass — and, for a
financial system, a second reviewer with context in the affected area:

- **money-touching changes** — anything that creates, moves, or reconciles
  funds; verify idempotency and concurrent-write safety explicitly
  ([ecto.md](ecto.md#concurrent-write-safety),
  [security.md](security.md#rate-limiting-and-abuse-boundaries))
- **migrations** — verify additive/backward-compatible shape and lock
  behavior against a hot table
  ([ecto.md](ecto.md#migration-safety))
- **authentication/authorization changes** — verify default-deny and that the
  check is enforced at the layer performing the operation, not only the edge
  ([security.md](security.md#authentication-and-authorization))
- **cross-context changes** — a PR that touches more than one bounded context
  is a signal the boundary may be leaking; verify context isolation
  ([principles/ddd.md](principles/ddd.md))

Escalating these to a slower, more careful review is not bureaucracy — it is
matching review effort to blast radius.

---

# Review Mindset

A review protects the system, not the author.

Ask:

- Does this respect layer boundaries?
- Is the concept defined once, in its canonical principle or Engineering
  Framework doc?
- Is it tested, readable and idiomatic?
- Would this scale and fail safely?

Prefer clarity and correctness over cleverness.

---

# Checklist

Before approving a PR:

- Tests cover new behavior and failures
- Quality gates pass with no warnings
- Layer boundaries are respected
- No duplicated principle or Engineering Framework definitions
- No known anti-patterns introduced
- Naming and errors are consistent
- Does this PR fall into a risk tier that needs a second reviewer?

---

# Related Documents

- [testing.md](testing.md)
- [coding_guidelines.md](coding_guidelines.md)
- [architecture.md](architecture.md)
- [otp.md](otp.md)
- [concurrency.md](concurrency.md)
- [error_handling.md](error_handling.md)
- [observability.md](observability.md)
- [performance.md](performance.md)
- [security.md](security.md)
- [ecto.md](ecto.md)
- [phoenix.md](phoenix.md)
