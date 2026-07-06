# Composition in Elixir

## Purpose

Composition is the core mechanism used to build systems in Elixir.

Instead of inheritance or large abstractions, we build systems by combining
small, independent units.

Everything in Elixir is composable:

- functions
- modules
- behaviours
- pipelines
- processes
- systems

---

# Core Idea

> Complex systems are built by composing simple parts.

---

# Function Composition

The most fundamental form of composition.

Example:

```elixir
payment
|> validate()
|> calculate_fee()
|> persist()
|> notify()
```

Each function:

- has a single responsibility
- receives input
- returns output
- does not mutate state

---

# Composition Over Abstraction

Prefer composition instead of deep abstractions.

Bad:

- large generic "Service" modules
- deep inheritance-like hierarchies (simulated via delegation)
- over-engineered frameworks

Good:

- small focused modules
- explicit pipelines
- clear data flow

---

# Data Composition

Data is composed through transformations.

Example:

```elixir
payment = %Payment{}
payment = Payment.validate(payment)
payment = Payment.approve(payment)
```

Each step produces a new version of the data.

---

# Module Composition

Modules should be composed like building blocks.

Each module should:

- expose a small API
- have a single responsibility
- depend on abstractions, not implementations

Example:

- Payment.Validator
- Payment.Calculator
- Payment.Repository

These modules compose into a system.

---

# Behaviour Composition

Behaviours allow polymorphic composition.

Example:

```elixir
def process(payment, provider) do
  provider.create(payment)
end
```

Different implementations can be swapped without changing logic.

---

# Pipeline Composition

Pipelines are the primary control flow mechanism.

Prefer:

```elixir
payment
|> step_one()
|> step_two()
|> step_three()
```

Over:

- nested case statements
- deeply imperative flows
- procedural chaining

---

# Process Composition (OTP)

In BEAM, systems are composed of processes.

Each process:

- has a single responsibility
- communicates via messages
- is isolated
- can fail independently

Processes compose into supervision trees.

---

# System Composition

A full system is composed of:

```
Domain (pure functions)
+ Application (use cases)
+ Ports (behaviours)
+ Adapters (infrastructure)
+ Delivery (Phoenix)
+ Processes (OTP)
```

Each layer is independently composable.

---

# Composition vs Coupling

Composition reduces coupling.

Coupling happens when:

- modules depend on concrete implementations
- logic is centralized in one place
- responsibilities are mixed

Composition ensures:

- replaceability
- testability
- isolation

---

# Anti-patterns

Avoid:

- God modules
- Service objects doing everything
- Deep inheritance-like delegation chains
- Hidden dependencies
- Over-abstraction
- Centralized business logic

---

# Functional Composition vs Object Composition

In OOP:

- composition often means object graphs

In Elixir:

- composition means function + data pipelines

No state mutation is required.

---

# Composition in Hexagonal Architecture

Each layer composes the next:

```
Phoenix → Application → Domain → Ports → Adapters
```

Each layer:

- receives input
- transforms data
- delegates responsibility downward

---

# Composition in DDD

In DDD:

- entities compose value objects
- aggregates compose entities
- domains compose services
- bounded contexts compose systems

---

# Composition and Testing

Composition enables:

- isolated unit tests
- mockable dependencies
- predictable flows
- deterministic behavior

Each composed unit is testable independently.

---

# Composition and Let It Crash

Composition works with failure isolation:

- each process is independent
- failure does not break composition
- supervisors maintain system integrity

---

# Design Rule

If something is hard to compose:

- it is too large
- it has too many responsibilities
- it needs to be split

---

# Practical Heuristics

When designing a module ask:

- Can this be split into smaller units?
- Can I reuse this piece independently?
- Does this depend on concrete implementations?
- Can I test it in isolation?
- Can I replace it easily?

If answer is “no”, refactor.

---

# Guiding Principle

> Systems should be composed, not constructed.
> Complexity should emerge from simple parts, not be centralized.   