# Functional Programming Principles (Elixir)

## Purpose

This document defines the functional programming principles that guide all
engineering decisions in this playbook.

Elixir is a functional language built on the BEAM.

We leverage this model to build systems that are:

- predictable
- composable
- testable
- concurrent-safe
- easy to reason about

---

# Core Idea

Functional programming is not about syntax.

It is about:

> Controlling complexity through function composition and immutability.

---

# Immutability

Data is immutable.

Once created, it is never modified.

Instead, transformations return new values.

Good:

```elixir
updated_payment = approve(payment)
```

Bad:

```elixir
payment.status = :approved
```

---

# Pure Functions

A pure function:

- depends only on its inputs
- produces no side effects
- always returns the same output for the same input

Example:

```elixir
def calculate_fee(amount, rate) do
  amount * rate
end
```

Pure functions are:

- easy to test
- easy to reason about
- safe to compose

---

# Side Effects

Side effects must be isolated.

Examples:

- database writes
- HTTP calls
- file system access
- logging
- telemetry
- external APIs

Rule:

> Side effects belong at the edges of the system.

---

# Referential Transparency

A function call can be replaced by its return value without changing behavior.

Example:

```elixir
calculate_total(items)
```

can be replaced with:

```elixir
150.00
```

if inputs are known.

This property makes systems predictable.

---

# Function Composition

Systems should be built by composing small functions.

Good:

```elixir
payment
|> validate()
|> calculate_fee()
|> persist()
|> notify()
```

Each step has a single responsibility.

---

# Avoid Shared State

Shared mutable state leads to:

- race conditions
- unpredictable behavior
- hard-to-reproduce bugs

Prefer:

- passing state explicitly
- returning new state

---

# Data Transformation Pipeline

Elixir excels at pipelines.

Prefer transforming data step-by-step.

Avoid large monolithic functions.

Good:

```elixir
payment
|> validate()
|> enrich()
|> persist()
```

Bad:

```elixir
def process(payment) do
  # 100 lines of mixed logic
end
```

---

# Pattern Matching as Control Flow

Pattern matching replaces conditionals.

Good:

```elixir
def process(%Payment{status: :pending} = payment), do: approve(payment)
def process(%Payment{status: :approved}), do: {:error, :already_processed}
```

Avoid deeply nested if/else chains.

---

# Data vs Behavior

Prefer data transformations over object-like behavior.

Elixir is not object-oriented.

Do not model systems as mutable objects.

Model them as transformations over data.

---

# Avoid Hidden State

Avoid:

- global variables
- Application.get_env inside domain logic
- implicit configuration
- process state unless necessary

Make dependencies explicit.

---

# Determinism

Functions should avoid randomness unless explicitly required.

Avoid:

- DateTime.utc_now()
- System.random
- external state

Inject dependencies instead.

Example:

```elixir
def process(payment, now_fn) do
  now = now_fn.()
end
```

---

# Composition Over Inheritance

Elixir does not use inheritance.

We use composition:

- functions
- pipelines
- behaviours
- modules

Complex behavior emerges from composition.

---

# Data Structures First

Prefer modeling with:

- structs
- maps (only for external boundaries)
- tuples for results

Avoid hidden transformations.

---

# Error Handling Philosophy

Errors are data.

Prefer:

```elixir
{:ok, value}
{:error, reason}
```

Avoid:

- exceptions for control flow
- silent failures

---

# Predictability

Given the same input, the system should behave the same way.

This is critical for:

- testing
- debugging
- concurrency
- distributed systems

---

# Concurrency Compatibility

Functional code is naturally concurrency-safe.

Because:

- no shared mutable state
- explicit inputs/outputs
- isolated side effects

---

# Testing Alignment

Functional code is easier to test because:

- functions are isolated
- dependencies can be injected
- no hidden state exists

This directly supports `testing.md`.

---

# Anti-patterns

Avoid:

- mutable state inside domain logic
- hidden dependencies
- functions that do multiple unrelated things
- mixing side effects with business logic
- global configuration access
- unpredictable functions

---

# Guiding Principle

> Prefer transformations over mutations.
> Prefer composition over complexity.
> Prefer explicit data over hidden state.