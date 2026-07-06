# Domain-Driven Design (Pragmatic Elixir)

## Purpose

Domain-Driven Design (DDD) is about aligning software design with business
requirements and language.

In this playbook, we use a **pragmatic and lightweight DDD approach**.

We avoid over-engineering and focus on clarity and maintainability.

---

# Core Idea

> The structure of the code should reflect the structure of the business.

---

# Ubiquitous Language

All code should use the same language as the business domain.

Examples:

Good:

- Payment
- Invoice
- Customer
- Account
- Fee
- Transfer

Bad:

- Data
- Manager
- Processor
- Helper
- Service (generic)

The language must be consistent across:

- code
- tests
- documentation
- API responses

---

# Bounded Contexts

A system should be divided into bounded contexts.

Each context represents a business capability.

Examples:

- Payments
- Accounts
- Customers
- Notifications
- Billing

---

## Rules for Contexts

Each context:

- has its own domain model
- has its own rules
- does not directly share internal models with others
- communicates via explicit interfaces

Avoid sharing structs between contexts.

---

# Context vs Module

A context is NOT a module.

A context is a **boundary of meaning**.

Example:

Payments context may contain:

- Payment
- FeeCalculator
- PaymentPolicy

Accounts context may contain:

- Account
- Balance
- LimitPolicy

They should not depend on each other's internals.

---

# Entities

Entities represent concepts with identity.

Examples:

- Payment
- Customer
- Invoice

Rules:

- have identity
- change over time
- encapsulate behavior

---

# Value Objects

Value Objects represent immutable concepts.

Examples:

- Money
- Email
- PixKey
- CPF
- Percentage

Rules:

- no identity
- immutable
- validated at creation

---

# Aggregates

Aggregates define consistency boundaries.

An aggregate:

- ensures invariants
- protects business rules
- controls state transitions

Example:

Payment Aggregate:

- cannot be approved twice
- cannot transition from cancelled → approved
- ensures valid state transitions

---

# Aggregate Root

The only entry point to an aggregate.

All modifications must go through it.

Example:

```elixir
Payment.approve(payment)
```

Not:

```elixir
payment.status = :approved
```

---

# Domain Services

Use Domain Services when:

- logic does not belong to a single entity
- multiple entities are involved
- rules are stateless

Examples:

- Fee calculation
- Fraud analysis
- Settlement rules

Keep them pure.

---

# Application Services

Application Services orchestrate use cases.

They:

- call domain logic
- coordinate repositories
- call external ports
- manage workflows

They do NOT contain business rules.

Example:

CreatePayment

CancelPayment

ApprovePayment

---

# Domain Events

Domain Events represent facts that already happened.

Examples:

- PaymentCreated
- PaymentApproved
- PaymentCancelled

Rules:

- expressed in past tense
- immutable
- represent business facts

---

# Event Usage

Events are used for:

- integration
- auditing
- async processing
- notifications

Not for core business logic execution.

---

# Invariants

Invariants are business rules that must always be true.

Examples:

- Payment cannot be negative
- Payment cannot be double-approved
- Account balance cannot go below zero

They must be enforced inside the domain.

---

# Anti-pattern: Anemic Domain Model

Avoid:

- structs with no behavior
- logic pushed into services only
- domain as simple data containers

Bad:

```elixir
%Payment{status: :pending}
```

with all logic elsewhere.

---

# Rich Domain Model (preferred)

Good:

```elixir
Payment.approve(payment)
Payment.cancel(payment)
```

Behavior lives with data.

---

# Context Isolation

Each bounded context:

- owns its data
- owns its rules
- owns its invariants

Communication must happen via:

- application services
- ports (behaviours)
- events

---

# Repositories in DDD

Repositories are interfaces to persistence.

They belong to the application/domain boundary.

They should:

- expose domain-friendly operations
- hide storage details

Good:

- get_payment(id)
- save(payment)
- find_pending()

Bad:

- select * queries
- generic CRUD without meaning

---

# Domain vs Application vs Infrastructure

## Domain

- business rules
- invariants
- entities
- value objects

## Application

- use cases
- orchestration
- workflows

## Infrastructure

- Ecto
- HTTP
- Redis
- external APIs

---

# Communication Between Contexts

Prefer:

- events
- application services
- explicit APIs

Avoid:

- shared schemas
- shared business logic
- direct cross-context calls

---

# Consistency Boundaries

Strong consistency is enforced inside an aggregate.

Weak consistency is allowed across contexts.

---

# Time and Determinism

Time must be injected when relevant.

Avoid:

```elixir
DateTime.utc_now()
```

inside domain logic.

---

# Error Modeling

Domain errors are explicit.

Examples:

- :invalid_amount
- :already_paid
- :insufficient_funds

Avoid generic error types.

---

# Testing Alignment

DDD strongly supports testing:

- entities → unit tests
- aggregates → state transition tests
- services → pure function tests
- application services → integration tests

See `testing.md`.

---

# Common Mistakes

Avoid:

- over-engineering aggregates
- too many bounded contexts too early
- sharing structs between contexts
- placing logic in controllers
- mixing application and domain logic
- treating DDD as a strict enterprise pattern

---

# Pragmatic DDD Rule

> Use DDD only where it increases clarity.
> Avoid complexity where it does not.

---

# Guiding Principle

> The domain model should be the most stable part of the system.