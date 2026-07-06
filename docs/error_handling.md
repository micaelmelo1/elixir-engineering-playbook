# Error Handling Model

## Purpose

This document defines how errors are classified, represented, propagated
across layers, and translated at system boundaries.

It covers error taxonomy, error representation, layer-by-layer propagation
contracts, retryable vs non-retryable classification, and external error
representation.

It builds on the crash philosophy in
[principles/let_it_crash.md](principles/let_it_crash.md) and the retry
execution mechanics in [concurrency.md](concurrency.md). It does not redefine
either:

- **let_it_crash.md** → decides *when a process should crash* vs return an
  error
- **this document** → decides *what an error is*, how it is represented, and
  how it flows through layers
- **concurrency.md** → decides *how a retry executes safely* (bounded,
  idempotent, single layer)

---

# References

Crash philosophy:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Retry execution mechanics:
[docs/concurrency.md](concurrency.md)

Functional error representation:
[docs/principles/functional_programming.md](principles/functional_programming.md)

Tagged-tuple syntax and naming:
[docs/coding_guidelines.md](coding_guidelines.md)

HTTP/API error mapping:
[docs/phoenix.md](phoenix.md)

---

# Error Taxonomy

Every error belongs to exactly one class. Classification determines
representation, propagation, and retryability.

## 1. Validation Errors

Malformed or incomplete input.

- example: `:invalid_amount`, `:missing_field`
- expected, never crashes
- never retryable — the input will not change on retry

## 2. Business Rule Violations

A domain invariant was not satisfied.

- example: `:insufficient_funds`, `:payment_already_captured`
- expected, never crashes
- never retryable — the business state will not change on retry unless a
  compensating action happens first

## 3. Infrastructure / Dependency Errors

A collaborator (database, external API, another node) failed or timed out.

- example: `:timeout`, `:connection_refused`, `:service_unavailable`
- expected at the system level
- may be retryable — classify explicitly, do not assume

## 4. Programmer Errors

A bad match, a `nil` where a value was required, an unhandled clause.

- unexpected
- never represented as `{:error, reason}` — let the process crash (see
  [let_it_crash.md](principles/let_it_crash.md))

---

# Error Representation

Represent errors as tagged tuples: `{:error, reason}`.

`reason` should be:

- an atom, when the failure needs no additional context —
  `{:error, :not_found}`
- a struct, when the failure carries context needed by the caller —
  `{:error, %PaymentError{reason: :insufficient_funds, payment_id: id}}`

Rules:

- never expose a dependency's raw exception or struct across a layer
  boundary (e.g. `Ecto.Changeset`, `Postgrex.Error`, `Mint.TransportError`)
- wrap dependency errors into a domain-meaningful reason at the adapter that
  calls the dependency
- keep the reason space per context small and explicit — avoid a single
  catch-all `:error` atom

---

# Error Propagation Across Layers

| Layer | Contract |
|-------|----------|
| Domain | pure functions return `{:error, reason}`; never raises, never wraps infrastructure errors |
| Application | orchestrates domain calls via `with`; translates infrastructure errors into domain-meaningful reasons |
| Adapter (Repo, HTTP client, external SDK) | wraps dependency exceptions into `{:error, reason}` at the boundary — no dependency error crosses into the domain or application layer |
| Process / GenServer (runtime) | decides crash vs return per [let_it_crash.md](principles/let_it_crash.md); never repairs invalid state in place |
| Phoenix (delivery) | maps domain `{:error, reason}` into an HTTP status and error envelope; never leaks internal reasons verbatim |

The domain must never see a dependency's error type. The adapter that calls
the dependency is responsible for translation.

---

# Retryable vs Non-Retryable Classification

Classify retryability at the point the error is created, not at the call
site that handles it.

Typically retryable:

- network timeouts
- connection refused / connection reset
- `5xx` responses from external services
- optimistic lock conflicts

Typically non-retryable:

- validation errors
- business rule violations
- `4xx` responses from external services (except `429`)
- authorization failures

Attach retryability explicitly rather than inferring it downstream:

```elixir
{:error, %DependencyError{reason: :timeout, retryable: true}}
```

The actual retry loop — bounds, idempotency requirement, single-layer
execution — is defined in [concurrency.md](concurrency.md#failure-propagation).
This section only classifies; it does not execute.

---

# External Error Representation

At the API boundary, translate internal reasons into a stable, minimal
envelope:

```json
{
  "error": {
    "code": "invalid_amount",
    "message": "Amount must be greater than zero",
    "request_id": "..."
  }
}
```

Rules:

- never leak internal atoms, stack traces, or struct contents to an external
  client
- maintain an explicit mapping from internal reason to external code — do
  not `to_string/1` the reason directly
- the domain never knows about HTTP status codes; the delivery layer owns
  the mapping (see [phoenix.md](phoenix.md))

---

# Anti-patterns

Avoid:

- catching all exceptions and returning a generic `{:error, :failed}`
- using exceptions for control flow in domain logic
- leaking dependency-specific errors (`Ecto.Changeset`, `Postgrex.Error`)
  outside the adapter that produced them
- retrying a non-idempotent operation
- mixing validation errors and infrastructure errors in the same reason
  space
- inferring retryability at the call site instead of classifying it at the
  source
- exposing internal error reasons directly in an API response

---

# Checklist

Before returning or propagating an error ask:

- Which of the four classes does this error belong to?
- Is the representation consistent up to the boundary where it is
  translated?
- If it crosses an adapter boundary, has the dependency's native error been
  wrapped?
- Is retryability explicit, not inferred?
- Does the external representation avoid leaking internal detail?
- Should this actually be a crash instead of an error tuple?

---

# Related Documents

- [principles/let_it_crash.md](principles/let_it_crash.md)
- [concurrency.md](concurrency.md)
- [coding_guidelines.md](coding_guidelines.md)
- [phoenix.md](phoenix.md)
- [otp.md](otp.md)
- [observability.md](observability.md)
- [security.md](security.md)
- [ecto.md](ecto.md)
- [review.md](review.md)
