# Hexagonal Architecture (Ports & Adapters)

## Purpose

Hexagonal Architecture defines how to structure a system so that:

- business rules are isolated from frameworks
- external systems are replaceable
- dependencies point inward
- the core domain remains stable over time

This architecture is fundamental to building maintainable Elixir systems.

---

# Core Idea

> The domain should not depend on anything external.

Everything external depends on the domain.

---

# High-Level Structure

```
        External Systems
               ↓
     -----------------------
     |   Adapters Layer    |
     -----------------------
               ↓
     -----------------------
     | Application Layer   |
     -----------------------
               ↓
     -----------------------
     |     Domain Core     |
     -----------------------
```

Dependencies always point inward.

---

# Layers

## Domain Core

The innermost layer.

Contains:

- business rules
- entities
- value objects
- policies
- invariants

Rules:

- no framework code
- no HTTP
- no database
- no external dependencies
- no process state

---

## Application Layer

Orchestrates use cases.

Responsibilities:

- coordinate domain logic
- call ports (behaviours)
- manage workflows
- define transactions (conceptually)

It does NOT contain business rules.

Example use cases:

- CreatePayment
- CancelPayment
- ApprovePayment

---

## Ports (Behaviours)

Ports define contracts.

They represent what the system needs from the outside world.

Examples:

- PaymentProvider
- NotificationService
- Repository
- Clock
- IdGenerator
- Cache

Ports belong close to the application/domain boundary.

---

## Adapters (Infrastructure)

Adapters implement ports.

They connect the system to external systems.

Examples:

- PostgreSQL repositories (Ecto)
- HTTP clients
- Redis cache
- Kafka producers
- Email providers
- External APIs

Adapters are replaceable.

---

## Delivery Layer

Examples:

- Phoenix controllers
- LiveView
- Channels
- GraphQL resolvers
- CLI commands

Responsibilities:

- translate input into use cases
- call application layer
- return response
- handle HTTP concerns

---

# Dependency Rule

All dependencies must point inward.

```
Phoenix → Application → Domain
Adapters → Ports → Domain
```

The domain must never depend on:

- Phoenix
- Ecto
- HTTP clients
- Redis
- external APIs

---

# Ports & Adapters Pattern

## Port (Behaviour)

```elixir
defmodule Payments.Boundary.PaymentProvider do
  @callback create_payment(map()) ::
    {:ok, term()} | {:error, term()}
end
```

---

## Adapter

```elixir
defmodule Payments.Infrastructure.StripePaymentProvider do
  @behaviour Payments.Boundary.PaymentProvider

  def create_payment(params) do
    HTTPClient.post("/payments", params)
  end
end
```

---

## Usage (Application Layer)

```elixir
defmodule Payments.Application.CreatePayment do
  def call(payment, provider) do
    provider.create_payment(payment)
  end
end
```

---

# Why Hexagonal Architecture

## 1. Testability

We can replace external systems with mocks easily.

---

## 2. Replaceability

We can switch:

- Stripe → Adyen
- PostgreSQL → MySQL
- HTTP → gRPC

without changing business logic.

---

## 3. Isolation

Business rules are unaffected by infrastructure changes.

---

## 4. Maintainability

Changes in external systems do not ripple into the core.

---

## 5. Clarity

Each layer has a clear responsibility.

---

# Common Mistakes

## 1. Business logic in Phoenix

Bad:

```elixir
def create(conn, params) do
  amount = params["amount"] * 1.1
end
```

---

## 2. Domain depending on Ecto

Bad:

```elixir
import Ecto.Changeset
```

inside domain logic.

---

## 3. Application layer doing business rules

Bad:

```elixir
def approve(payment) do
  if payment.amount > 1000 do
    ...
  end
end
```

---

## 4. Leaking infrastructure into domain

Bad:

- HTTP structs
- Repo queries
- Conn objects

---

# Domain Independence Rule

The domain must be usable:

- without Phoenix
- without Ecto
- without HTTP
- without database

If the domain cannot run in isolation, architecture is broken.

---

# Data Flow

```
Request
  ↓
Delivery Layer (Phoenix)
  ↓
Application Layer (Use Case)
  ↓
Domain (Business Rules)
  ↓
Ports (Behaviour calls)
  ↓
Adapters (Infrastructure)
  ↓
External Systems
```

---

# Testing Alignment

This architecture directly supports testing strategy:

- Domain → unit tests
- Application → integration tests (mocked ports)
- Adapters → contract tests
- Phoenix → request tests

See `testing.md`.

---

# State Management

State belongs only in:

- database (persistent state)
- processes (runtime state when needed)

Domain should remain stateless.

---

# Evolution Strategy

Hexagonal architecture allows:

- gradual refactoring
- replacement of infrastructure
- independent module evolution

Without rewriting the domain.

---

# Anti-patterns

Avoid:

- layered frameworks that mix concerns
- “fat” Phoenix contexts
- repositories inside domain
- direct HTTP calls anywhere except adapters
- Ecto schemas in domain
- business rules inside adapters

---

# Guiding Principle

> Everything external is replaceable.
> Only the domain is stable.