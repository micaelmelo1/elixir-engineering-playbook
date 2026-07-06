# Elixir Engineering Playbook

A single-source-of-truth engineering handbook for building high-quality Elixir
and Phoenix applications.

Core concepts are defined once, in the principles. Every other document
references them rather than redefining them.

---

## Principles

Canonical definitions. Each concept lives in exactly one file.

- [functional_programming.md](docs/principles/functional_programming.md)
- [hexagonal_architecture.md](docs/principles/hexagonal_architecture.md)
- [ddd.md](docs/principles/ddd.md)
- [solid.md](docs/principles/solid.md)
- [let_it_crash.md](docs/principles/let_it_crash.md)
- [composition.md](docs/principles/composition.md)

---

## Core Docs

Applied guidance. These reference the principles and Engineering Framework docs
instead of duplicating them.

- [domain.md](docs/domain.md)
- [architecture.md](docs/architecture.md)
- [coding_guidelines.md](docs/coding_guidelines.md)
- [testing.md](docs/testing.md)
- [phoenix.md](docs/phoenix.md)
- [review.md](docs/review.md) — enforcing the playbook in development

---

## Engineering Framework

Layer-specific execution guides — the canonical source for the operational
rules of their layer. They apply the principles; they do not redefine them.

- [otp.md](docs/otp.md) — runtime and supervision model
- [concurrency.md](docs/concurrency.md) — safe concurrent execution
- [error_handling.md](docs/error_handling.md) — error taxonomy, propagation, and boundaries
- [ecto.md](docs/ecto.md) — persistence without leaking into the domain
- [observability.md](docs/observability.md) — understanding production behavior
- [performance.md](docs/performance.md) — scaling correctly on the BEAM
- [security.md](docs/security.md) — protecting system integrity
