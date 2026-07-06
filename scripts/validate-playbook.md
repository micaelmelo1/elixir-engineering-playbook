# Playbook Validation Rules

Concepts live in one of two canonical tiers:

- **Principles** (`docs/principles/*`) — theory (what/why).
- **Engineering Framework** — operational rules (how), each owned by one doc:
  `otp.md`, `concurrency.md`, `error_handling.md`, `ecto.md`,
  `observability.md`, `performance.md`, `security.md`.

Check the repository for:

## 1. Duplicate Concepts
A concept must appear only in its canonical owner. Flag:
- DDD outside principles/ddd.md
- SOLID outside principles/solid.md
- OTP / runtime structure outside otp.md
- concurrency rules outside concurrency.md
- error taxonomy / propagation outside error_handling.md
- persistence rules outside ecto.md
- observability rules outside observability.md
- performance rules outside performance.md
- security boundaries outside security.md

## 2. Forbidden Re-definition
Application docs (domain, architecture, coding_guidelines, testing, phoenix,
review) MUST NOT define:
- principle theory
- operational rules owned by an Engineering Framework doc

They reference the canonical owner instead.

## 3. Reference Enforcement
Every conceptual mention in a non-canonical doc MUST point to its canonical
owner — a principle in `docs/principles/*` or the owning Engineering Framework
doc.