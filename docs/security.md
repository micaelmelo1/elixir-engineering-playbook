# Security Model

## Purpose

This document defines how to protect system integrity.

It covers input validation boundaries, data isolation, secrets management,
trust boundaries with external systems, authentication and authorization, and
rate limiting / abuse boundaries.

Security is enforced at boundaries. It applies the architecture and domain
principles; it does not redefine them. It does not classify *what an error is* —
error taxonomy and boundary translation are canonical in
[error_handling.md](error_handling.md); security-relevant audit records are
canonical in [observability.md](observability.md#audit-trail).

---

# References

Boundaries and adapters as trust edges:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain invariants as integrity guarantees:
[docs/principles/ddd.md](principles/ddd.md)

Error classification and boundary translation:
[docs/error_handling.md](error_handling.md)

Security-relevant audit records:
[docs/observability.md](observability.md#audit-trail)

Backpressure vs rate limiting boundary:
[docs/concurrency.md](concurrency.md#backpressure-strategies)

---

# Input Validation Boundaries

Validate at two distinct layers:

- delivery layer validates request shape (types, required fields, format)
- domain validates business rules and invariants

Never trust external input. All data crossing a boundary is untrusted until
validated.

Never build atoms from external input (`String.to_atom/1`); use explicit
mappings or `String.to_existing_atom/1`.

A failed validation is a validation error, not a crash. This section defines
*where* validation happens; how the resulting error is represented and
propagated is canonical in
[error_handling.md](error_handling.md#error-taxonomy).

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

The domain never reads secrets. Adapters receive what they need. The "never
log secrets" rule is enforced at the observability boundary — see
[observability.md](observability.md#logging-strategy).

## Runtime Exposure

A secret held in process state, ETS, or a closure can leak through the runtime,
not only through logs:

- a crash report or error `Logger` line can serialize a GenServer's `state`,
  including any credential it holds — keep raw secrets out of long-lived
  process state; hold a handle or fetch on demand
- runtime introspection (`:observer`, `Process.info/2`, `:sys.get_state/1`)
  exposes process state to anyone with a shell on the node
- wrap a secret in an opaque struct that redacts under `Inspect`, so it cannot
  be accidentally printed, logged, or dumped

## Rotation

Secrets are rotated, not permanent.

- read secrets through an indirection (runtime config / provider); never
  capture a secret value into module attributes at compile time
- a rotated secret must take effect without a code deploy

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

Authentication answers *who is calling*; authorization answers *may this
identity perform this action on this resource*. Keep them separate.

- authentication belongs to the delivery layer; the domain receives an
  already-authenticated identity, never a `conn` or transport detail
- authorization decisions are explicit and centralized, not scattered across
  controllers
- default deny: the absence of an explicit grant is a denial, never a
  fallthrough allow
- enforce authorization at the layer that performs the operation, not only at
  the edge — a controller-only check is bypassed by a second entry point
  (another controller, a background job, an internal call). For financial
  state changes, the application/domain operation re-checks the identity's
  right to act
- scope every query and command to the authenticated identity (account /
  tenant ownership); never trust a client-supplied identity or account id

An authorization failure is a distinct, non-retryable error class (see
[error_handling.md](error_handling.md#retryable-vs-non-retryable-classification))
and a security-relevant event — authentication failures, authorization
denials, and privilege changes must emit an audit record
([observability.md](observability.md#audit-trail)).

See [phoenix.md](phoenix.md#authorization) for delivery-layer handling.

---

# Rate Limiting and Abuse Boundaries

Rate limiting protects the system from *abusive or excessive* use. It is
distinct from backpressure, which protects the system from *legitimate load it
cannot yet absorb* (see
[concurrency.md](concurrency.md#backpressure-strategies)). A production system
needs both.

- enforce limits at the delivery boundary, keyed by a meaningful identity
  (authenticated principal, API key, source) — not by IP alone, which is
  spoofable and shared
- fail closed and return an explicit, retryable signal (HTTP `429`) rather
  than silently dropping or crashing
- protect expensive and security-sensitive operations first — authentication
  attempts, payment initiation, anything that touches money or external cost
- for financial operations, require an idempotency key so a retried or replayed
  request cannot double-execute; replay protection is a security boundary, not
  only a reliability one (retry mechanics:
  [concurrency.md](concurrency.md#failure-propagation))
- a limiter is enforced state — a distributed limiter must share state across
  nodes or degrade to a safe per-node bound, never to "no limit"

Abuse boundaries are part of the trust edge, not an afterthought bolted on
under load.

---

# Anti-patterns

Avoid:

- trusting client-supplied data or a client-supplied identity / account id
- building atoms from external input
- logging secrets, tokens or PII
- hardcoded credentials, or secrets captured at compile time (no rotation
  without a deploy)
- holding raw secrets in long-lived process state where a crash dump or
  introspection can expose them
- exposing internal structs to clients
- letting unvalidated external responses reach the domain
- scattered, implicit authorization checks
- authorization enforced only at the edge, or default-allow fallthrough
- money-touching endpoints with no rate limit or no idempotency key (replay /
  double-execution)

---

# Checklist

Before shipping a feature ask:

- Is request shape validated at the boundary?
- Are business invariants enforced in the domain?
- Are secrets injected, never hardcoded or logged, kept out of process state /
  crash dumps, and rotatable without a deploy?
- Is sensitive data kept out of logs and responses?
- Are external responses validated before use?
- Is authorization default-deny, explicit, and enforced at the layer that
  performs the operation?
- Are money-touching endpoints rate limited and idempotent against replay?
- Do authentication and authorization events produce audit records?

---

# Related Documents

- [phoenix.md](phoenix.md)
- [ecto.md](ecto.md)
- [observability.md](observability.md)
- [error_handling.md](error_handling.md)
- [concurrency.md](concurrency.md)
