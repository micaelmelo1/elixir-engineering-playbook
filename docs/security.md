# Security Model

## Purpose

This document defines how to protect system integrity.

It covers input validation boundaries, data isolation, secrets management and
trust boundaries with external systems.

Security is enforced at boundaries. It applies the architecture and domain
principles; it does not redefine them.

---

# References

Boundaries and adapters as trust edges:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain invariants as integrity guarantees:
[docs/principles/ddd.md](principles/ddd.md)

---

# Input Validation Boundaries

Validate at two distinct layers:

- delivery layer validates request shape (types, required fields, format)
- domain validates business rules and invariants

Never trust external input. All data crossing a boundary is untrusted until
validated.

Never build atoms from external input (`String.to_atom/1`); use explicit
mappings or `String.to_existing_atom/1`.

---

# Data Isolation Rules

Sensitive data must not spread beyond where it is needed.

- keep secrets and PII out of logs, telemetry and error messages
- do not serialize internal structs directly to clients; use explicit views
- scope data access to the authenticated identity

Each bounded context owns its data and does not expose internals to others.

---

# Secrets Management

Secrets are injected configuration, never hardcoded.

- load from environment/runtime configuration at the boundary
- never commit secrets to the repository
- never log or include secrets in error payloads
- pass credentials to adapters explicitly, not through global domain access

The domain never reads secrets. Adapters receive what they need.

---

# Trust Boundaries With External Systems

Treat every external system as untrusted and unreliable.

- validate and sanitize responses before using them
- enforce timeouts and fail closed on ambiguity
- translate external errors into explicit domain errors
- do not let external payloads flow unchecked into the domain

The adapter is the trust boundary. Nothing crosses it unvalidated.

---

# Authentication and Authorization

- authentication belongs to the delivery layer; the domain receives an
  authenticated identity
- authorization decisions are explicit and centralized, not scattered
- the domain never inspects `conn` or transport details

See [phoenix.md](phoenix.md) for delivery-layer handling.

---

# Anti-patterns

Avoid:

- trusting client-supplied data
- building atoms from external input
- logging secrets, tokens or PII
- hardcoded credentials
- exposing internal structs to clients
- letting unvalidated external responses reach the domain
- scattered, implicit authorization checks

---

# Checklist

Before shipping a feature ask:

- Is request shape validated at the boundary?
- Are business invariants enforced in the domain?
- Are secrets injected, never hardcoded or logged?
- Is sensitive data kept out of logs and responses?
- Are external responses validated before use?
- Is authorization explicit and centralized?

---

# Related Documents

- [phoenix.md](phoenix.md)
- [ecto.md](ecto.md)
- [observability.md](observability.md)
