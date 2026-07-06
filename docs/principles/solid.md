# SOLID Principles (Applied to Elixir)

## Purpose

SOLID principles are guidelines for writing maintainable and modular code.

In Elixir, these principles must be interpreted through a functional and compositional lens.

We do NOT apply SOLID as in object-oriented design.

We adapt it to functional programming.

---

# Core Idea

> SOLID is about managing complexity through separation of concerns,
> composability and explicit dependencies.

---

# S — Single Responsibility Principle (SRP)

A module should have one reason to change.

In Elixir:

- a module should represent one concept
- or one responsibility in the system

Good:

- PaymentValidator
- PaymentCalculator
- PaymentRepository

Bad:

- PaymentManager
- PaymentService (generic)
- Utils

---

## Important Clarification

In functional systems, responsibility is:

> a single reason for change in behavior

Not:

> a single function

---

# O — Open/Closed Principle (OCP)

Modules should be open for extension, closed for modification.

In Elixir this is achieved through:

- behaviours
- pattern matching
- function composition
- pipelines

Example:

```elixir
defprotocol PaymentProcessor do
  def process(payment)
end
```

or behaviours:

```elixir
@callback process(payment) :: {:ok, term()} | {:error, term()}
```

We extend behavior by adding new implementations, not modifying existing ones.

---

# L — Liskov Substitution Principle (LSP)

Any implementation of a behaviour must be replaceable without breaking the system.

If a module implements a behaviour:

- it must respect the contract
- it must not weaken guarantees
- it must not introduce unexpected behavior

Bad:

- implementation returning inconsistent error formats
- skipping required validations
- changing semantic meaning

Good:

- consistent return types
- predictable behavior
- contract adherence

---

# I — Interface Segregation Principle (ISP)

Prefer small, focused behaviours.

Avoid large “god interfaces”.

Bad:

```elixir
CoreProvider
  create_payment
  cancel_payment
  refund_payment
  validate_account
  send_notification
```

Good:

- PaymentProvider
- RefundProvider
- NotificationService
- AccountValidator

Each interface should represent a single capability.

---

# D — Dependency Inversion Principle (DIP)

High-level modules must not depend on low-level modules.

Both must depend on abstractions.

In Elixir:

- domain/application depends on behaviours
- infrastructure implements behaviours

Good:

```elixir
def create(payment, provider) do
  provider.create_payment(payment)
end
```

Bad:

```elixir
def create(payment) do
  Stripe.create(payment)
end
```

---

# Functional Interpretation of SOLID

SOLID in Elixir is achieved through:

- pure functions
- immutable data
- behaviours
- composition
- pipelines
- explicit dependencies

Not through class hierarchies.

---

# SRP in Functional Terms

A function or module should:

- transform data
- or coordinate processes
- but not both at the same time

Bad:

```elixir
def process(payment) do
  validate(payment)
  Repo.insert(payment)
  HTTP.post(...)
end
```

Good:

Split responsibilities:

- validate/transform
- persist
- notify

---

# OCP in Functional Terms

We extend behavior via:

- new function clauses
- new modules implementing behaviours
- composition pipelines

We avoid modifying existing stable logic when extending features.

---

# LSP in Functional Systems

LSP means:

> Any module implementing a behaviour must be interchangeable.

This ensures:

- testability
- polymorphism via behaviours
- stable contracts

---

# ISP in Functional Systems

Prefer:

- multiple small behaviours
- explicit contracts per capability

Avoid:

- mega behaviours
- “God interfaces”

---

# DIP in Functional Systems

Core rule:

> Domain defines contracts, infrastructure implements them.

Flow:

```
Domain → Behaviour → Adapter
```

Never reverse.

---

# Anti-patterns

Avoid:

- god modules
- service objects doing everything
- large behaviours
- direct dependency on infrastructure
- hidden dependencies
- tight coupling between modules

---

# Common Misinterpretations

SOLID is NOT:

- about classes (irrelevant in Elixir)
- about inheritance
- about strict layering only
- about abstraction for its own sake

---

# Correct Usage in Elixir

Use SOLID to guide:

- module boundaries
- behaviour design
- dependency direction
- system modularity

Not to enforce OO patterns.

---

# Checklist

Before creating or modifying a module:

- Does it have a single responsibility?
- Can it be replaced easily?
- Does it depend only on abstractions?
- Is it composable?
- Is it minimal in scope?
- Is it testable in isolation?

---

# Relationship to Other Principles

- functional_programming.md → defines execution model
- hexagonal_architecture.md → defines system structure
- ddd.md → defines domain modeling
- solid.md → defines code-level design rules

---

# Guiding Principle

> SOLID in Elixir is about composition, not inheritance.
> About contracts, not classes.
> About clarity, not abstraction for its own sake.