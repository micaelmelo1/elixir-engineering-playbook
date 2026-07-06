# Ecto Integration Model

## Purpose

This document defines how persistence is integrated without leaking into the
domain.

It covers Ecto as infrastructure, schema vs domain separation, the repository
pattern, transaction boundaries, concurrent write safety, query isolation,
query performance and migration safety.

It applies the architecture and domain principles to persistence. It does not
redefine them — those are canonical. Error taxonomy and the adapter-boundary
wrapping contract are canonical in
[error_handling.md](error_handling.md#error-propagation-across-layers); this
document applies that contract to Ecto/Postgrex specifically.

---

# References

Persistence as a replaceable adapter:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain model and boundaries:
[docs/principles/ddd.md](principles/ddd.md)

Error taxonomy and adapter-boundary wrapping contract:
[docs/error_handling.md](error_handling.md)

Race condition handling and process ownership:
[docs/concurrency.md](concurrency.md#race-condition-handling)

Query cost and memory behavior:
[docs/performance.md](performance.md#memory-behavior)

Data isolation and PII handling:
[docs/security.md](security.md#data-isolation-rules)

---

# Ecto Is Infrastructure Only

Ecto lives in the infrastructure layer, behind a port.

The domain never imports `Ecto`, never builds changesets, never runs queries.

If Ecto disappeared, the domain would still compile and its tests would still
pass.

---

# Schema vs Domain Separation

Ecto schemas are persistence records, not domain entities.

- schemas map rows to structs
- domain entities carry business behavior and invariants

Translate at the boundary:

- schema → domain struct when reading
- domain struct → schema/params when writing

Do not put business rules in schemas or in changesets. Changesets validate
storage shape; the domain validates business rules.

---

# Repository Pattern Usage

Repositories are ports exposed to the application layer.

They should:

- expose domain-oriented operations (`find_pending`, `save`)
- return domain structs or explicit errors
- hide queries, schemas and `Repo` behind the adapter

The application depends on the repository behaviour; the Ecto implementation is
one adapter among possible others.

---

# Transaction Boundaries

Transactions belong in the infrastructure/application boundary, never in the
domain and never in controllers.

- open a transaction around a single use case's writes
- keep transactions short — a long transaction holds row/table locks that
  block other writers
- convert failures into explicit domain errors before returning

Pure domain decisions happen before the transaction; the transaction only
commits the resulting effects.

Compose multi-step writes with `Ecto.Multi` rather than nested `Repo.transaction`
calls:

- each step is named, so a failure identifies exactly which step failed
- a failed step aborts the whole multi and rolls back — no step partially
  commits
- keep non-database side effects (publishing events, calling external
  services) out of the multi; run them after the transaction commits, since a
  rollback cannot undo them

---

# Concurrent Write Safety

Process ownership ([otp.md](otp.md#process-ownership-rules),
[concurrency.md](concurrency.md#race-condition-handling)) serializes access to
in-memory state on one node. A database row is shared, persisted state that
multiple nodes and processes can write concurrently — process ownership does
not reach it. The database itself must enforce the safety.

- use a unique constraint (not a prior `SELECT` check) to prevent duplicate
  rows — a check-then-insert has a race window; the constraint is the actual
  guarantee, checked with `unique_constraint/3` in the changeset
- use optimistic locking (`Ecto.Changeset.optimistic_lock/3`) when a row is
  read, decided on, and later written by a use case — the write fails if
  another writer changed the row in between, instead of silently overwriting
  it
- use a row-level lock (`SELECT ... FOR UPDATE` inside a transaction) only when
  the operation must serialize against concurrent readers-then-writers of the
  same row, and keep the locked transaction short
- for money movement specifically, a unique constraint on the idempotency key
  is the concurrency guarantee against duplicate processing (see
  [security.md](security.md#rate-limiting-and-abuse-boundaries) for the
  request-level idempotency requirement this backs)

Never rely on "we checked before we wrote" without a database-enforced
constraint — the check and the write are not atomic.

---

# Query Isolation Rules

Queries live inside adapters only.

- no `Ecto.Query` in domain, application or delivery layers
- no raw `Repo` calls from controllers or contexts' business logic
- expose named, intention-revealing repository functions instead of leaking
  query fragments

This keeps storage decisions replaceable and testable.

## Query Cost

A repository function's shape hides its cost from the caller — make sure it
does not hide an unbounded query:

- preload associations explicitly and intentionally; an unpreloaded
  association accessed in a loop is an N+1 query, invisible until load
- stream large result sets (`Repo.stream/2` inside a transaction) instead of
  loading them into memory — the general memory rule is canonical in
  [performance.md](performance.md#memory-behavior)

---

# Error Translation

Infrastructure errors must not leak upward. This is the Ecto/Postgrex instance
of the adapter-boundary wrapping contract that is canonical in
[error_handling.md](error_handling.md#error-propagation-across-layers) — this
section only names the Ecto-specific mapping, it does not redefine the
contract.

Convert:

- a unique/check constraint violation → an explicit domain reason matched by
  constraint name (`unique_constraint(:email, name: :users_email_index)` →
  `{:error, :already_exists}`), not a generic `{:error, :changeset_invalid}`
- `Ecto.NoResultsError` / `nil` from `Repo.get` → `{:error, :not_found}`
- `Postgrex.Error` (e.g. connection/timeout) → an infrastructure error, per
  [error_handling.md](error_handling.md#error-taxonomy)

The domain never sees an `Ecto.Changeset`, a `Postgrex.Error`, or any other
database struct.

---

# Migration Safety

A migration runs against a live production database while the old and new
application code may both be running (rolling deploy). A migration that
assumes instantaneous, exclusive access to the table is a production incident.

- make migrations additive and backward-compatible: add a column as nullable
  or with a default before code depends on it; deploy the code; only then
  remove or tighten the old shape in a later migration
- never rename or drop a column in the same deploy that removes the code using
  it — the old code (still running during rollout) will break
- avoid an operation that takes a long-held lock on a hot table (e.g. adding a
  `NOT NULL` column with a default on Postgres versions that rewrite the
  table, adding an index without `CONCURRENTLY`)
- backfill existing rows in small batches out of the request path, not as a
  single migration statement that locks the table for its duration
- for a financial system, a broken migration mid-deploy must not corrupt or
  block payment writes — rehearse destructive or long-running migrations
  against a production-sized copy first

---

# Anti-patterns

Avoid:

- business logic in changesets or schemas
- `Ecto.Query` outside adapters
- `Repo` calls from controllers
- transactions started in controllers or domain code
- returning schemas as if they were domain entities
- leaking Ecto errors into business rules
- check-then-insert without a database constraint enforcing it
- reading and later writing a row with no optimistic lock or row-level lock
  against concurrent writers
- non-database side effects (events, external calls) executed inside an
  `Ecto.Multi` step
- an unpreloaded association accessed in a loop (N+1)
- a migration that renames/drops a column the currently-running code still uses
- a long-locking migration run against a hot production table without
  `CONCURRENTLY` or batching

---

# Checklist

Before adding persistence ask:

- Is Ecto confined to an adapter?
- Is the schema separate from the domain entity?
- Does the repository expose domain-oriented operations?
- Is the transaction boundary a single use case?
- Are infrastructure errors translated to domain errors, matched by constraint
  name rather than a generic reason?
- Could the domain run without Ecto?
- Is concurrent write safety enforced by the database (unique constraint,
  optimistic lock), not just checked-then-assumed in application code?
- Are non-database side effects kept out of the `Ecto.Multi`/transaction?
- Would this query N+1 under a realistic list size, and should a large result
  set stream instead of load?
- Is this migration additive/backward-compatible, and safe to run against a
  live, hot table?

---

# Related Documents

- [architecture.md](architecture.md)
- [domain.md](domain.md)
- [phoenix.md](phoenix.md)
- [error_handling.md](error_handling.md)
- [concurrency.md](concurrency.md)
- [performance.md](performance.md)
- [security.md](security.md)
