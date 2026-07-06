# Ecto Integration Model

## Purpose

This document defines how persistence is integrated without leaking into the
domain.

It covers Ecto as infrastructure, schema vs domain separation, the repository
pattern, transaction boundaries and query isolation.

It applies the architecture and domain principles to persistence. It does not
redefine them — those are canonical.

---

# References

Persistence as a replaceable adapter:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain model and boundaries:
[docs/principles/ddd.md](principles/ddd.md)

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
- keep transactions short
- convert failures into explicit domain errors before returning

Pure domain decisions happen before the transaction; the transaction only
commits the resulting effects.

---

# Query Isolation Rules

Queries live inside adapters only.

- no `Ecto.Query` in domain, application or delivery layers
- no raw `Repo` calls from controllers or contexts' business logic
- expose named, intention-revealing repository functions instead of leaking
  query fragments

This keeps storage decisions replaceable and testable.

---

# Error Translation

Infrastructure errors must not leak upward.

Convert:

- constraint violations → explicit domain errors (`{:error, :already_exists}`)
- not-found → `{:error, :not_found}`

The domain never sees an `Ecto.Changeset` or a database struct.

---

# Anti-patterns

Avoid:

- business logic in changesets or schemas
- `Ecto.Query` outside adapters
- `Repo` calls from controllers
- transactions started in controllers or domain code
- returning schemas as if they were domain entities
- leaking Ecto errors into business rules

---

# Checklist

Before adding persistence ask:

- Is Ecto confined to an adapter?
- Is the schema separate from the domain entity?
- Does the repository expose domain-oriented operations?
- Is the transaction boundary a single use case?
- Are infrastructure errors translated to domain errors?
- Could the domain run without Ecto?

---

# Related Documents

- [architecture.md](architecture.md)
- [domain.md](domain.md)
- [phoenix.md](phoenix.md)
