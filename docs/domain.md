# Domain Modeling Guidelines

## Purpose

The domain is the heart of the application.

It contains the business rules, policies, invariants and concepts that define
the problem being solved.

Everything else exists to support the domain.

The domain should remain independent from frameworks, databases and external
services.

---

# Core Principles

The domain should be:

- Explicit
- Predictable
- Deterministic
- Testable
- Framework-independent

Business rules should survive changes in infrastructure.

---

# Domain First

Always design the business model before writing code.

Start by identifying:

- Entities
- Value Objects
- Business Rules
- Invariants
- Policies
- Events

Avoid starting from the database schema.

Avoid starting from HTTP requests.

Avoid starting from Phoenix contexts.

---

# Domain Language

Use the business language.

Code should reflect the vocabulary used by domain experts.

Good

Payment

Invoice

Mandate

Transfer

Settlement

Account

Fee

Bad

Data

Record

Info

Object

Manager

Helper

---

# Entities

Entities have identity.

Examples

Payment

Customer

Invoice

Account

Transfer

Entities usually contain:

- identity
- state
- business behavior

Prefer structs.

Example

```elixir
defmodule Payments.Payment do
  @enforce_keys [:id, :amount, :status]

  defstruct [
    :id,
    :amount,
    :status,
    :created_at
  ]
end
```

---

# Value Objects

Value Objects do not have identity.

Equality depends only on their values.

Examples

Money

PixKey

Cpf

Email

Address

Percentage

Fee

Currency

Good Value Objects are:

Immutable

Validated

Self-contained

Example

```elixir
Money.new(100, "BRL")
```

instead of

```elixir
%{
  amount: 100,
  currency: "BRL"
}
```

---

# Business Behavior

Behavior belongs close to the data.

Prefer

```elixir
Payment.approve(payment)
```

instead of

```elixir
PaymentService.approve(payment)
```

whenever the behavior naturally belongs to the entity.

---

# Domain Services

Use a Domain Service only when behavior does not belong to a single entity.

Examples

FeeCalculator

SettlementPolicy

FraudAnalysis

ExchangeRate

TaxCalculator

Domain Services should remain pure.

---

# Application Services

Application Services orchestrate the domain.

Responsibilities include:

Calling repositories

Calling providers

Publishing events

Managing transactions

They should contain very little business logic.

---

# Invariants

Protect business rules.

An invalid entity should never exist.

Example

A payment cannot be approved twice.

A Pix key cannot be empty.

A Money amount cannot be negative.

Validate these rules inside the domain.

---

# Validation

Prefer validation close to the domain.

Return explicit errors.

Good

```elixir
{:error, :invalid_amount}
```

Avoid

```elixir
raise "Invalid amount"
```

---

# State Transitions

Represent state transitions explicitly.

Example

Pending

↓

Approved

↓

Settled

↓

Cancelled

Every transition should be validated.

Avoid assigning arbitrary values.

---

# Domain Events

Important business facts should become events.

Examples

PaymentCreated

PaymentApproved

PaymentCancelled

InvoiceIssued

Events describe something that already happened.

Use past tense.

---

# Repositories

Repositories are interfaces.

They belong to the domain or application layer.

Repositories should expose business-oriented operations.

Good

save(payment)

find(id)

find_pending()

Avoid generic CRUD-only interfaces whenever richer domain operations improve
clarity.

---

# External Services

The domain never calls HTTP clients.

The domain never talks to Redis.

The domain never knows about Ecto.

Use behaviours.

---

# Side Effects

Domain code should avoid side effects.

Bad

```elixir
def approve(payment) do
  Repo.update(...)
end
```

Good

```elixir
def approve(payment) do
  {:ok, %{payment | status: :approved}}
end
```

Persistence happens elsewhere.

---

# Error Modeling

Errors are part of the domain.

Prefer explicit domain errors.

Examples

:not_found

:expired

:already_paid

:already_cancelled

:insufficient_balance

Avoid generic errors.

---

# Domain Purity

Avoid inside the domain:

Application.get_env()

Logger

Repo

HTTP clients

Redis

Kafka

Phoenix

Conn

Socket

LiveView

Controllers

---

# Time

Time is a dependency.

Avoid

```elixir
DateTime.utc_now()
```

inside business rules.

Inject time when deterministic behavior is required.

---

# Identifiers

Generate identifiers outside the domain whenever possible.

This improves deterministic testing.

---

# Money

Never represent money using float.

Prefer:

Decimal

or a dedicated Money value object.

---

# Idempotency

Domain operations should be idempotent whenever applicable.

Calling an operation twice should not corrupt business state.

---

# Business Rules

Business rules should be easy to discover.

Avoid spreading one rule across multiple modules.

Keep related rules together.

---

# Pattern Matching

Use pattern matching to model business rules.

Prefer multiple function clauses over deeply nested conditionals.

---

# Testing

The domain should contain the largest number of unit tests.

Pure domain logic should be fast to test.

The domain should not require:

Database

Phoenix

HTTP

Redis

External APIs

to execute tests.

---

# Anti-patterns

Avoid:

Anemic domain models

God services

Business logic in controllers

Business logic in Ecto schemas

Business logic in migrations

Business logic inside GenServers

Database-driven design

Framework-driven design

---

# Checklist

Before introducing new domain logic ask:

- Does this belong to the domain?
- Is it pure?
- Can it be tested in isolation?
- Does it model business language?
- Does it protect invariants?
- Does it expose explicit errors?
- Does it avoid infrastructure concerns?
- Is it deterministic?

---

# Related Documents

- philosophy.md
- architecture.md
- coding_guidelines.md
- testing.md
- ecto.md