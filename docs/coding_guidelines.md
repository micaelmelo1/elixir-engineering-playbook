# Elixir Coding Guidelines

## Purpose

This document defines Elixir-specific coding standards: naming, error handling,
idioms and tooling.

It does not redefine design principles. Those are canonical in
[docs/principles](principles).

---

# Design Principles (References)

Behaviours define contracts for external dependencies.

Design principles:
[docs/principles/solid.md](principles/solid.md)

Architecture principles:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Composition rules:
[docs/principles/composition.md](principles/composition.md)

Functional foundation:
[docs/principles/functional_programming.md](principles/functional_programming.md)

---

# General Rules

## Prefer simplicity

Choose the simplest solution that correctly solves the problem.

Avoid "clever" code.

Good code should feel boring.

---

## Be explicit

Prefer

```elixir
{:ok, payment}
{:error, :invalid_amount}
```

instead of

```elixir
raise "Invalid amount"
```

---

## Keep functions small

Functions should usually fit on one screen.

A function should communicate one idea.

---

# Naming

## Modules

Modules represent concepts.

Good

Payment

PaymentValidator

Invoice

Customer

Avoid

Manager

Handler

Helper

Utils

unless they clearly describe the responsibility.

---

## Functions

Functions represent actions.

Good

validate()

approve()

calculate_fee()

Bad

do_work()

process_data()

handle()

without proper context.

---

## Variables

Variable names should communicate intent.

Good

payment

expiration_date

retry_count

Bad

data

value

tmp

---

# Pattern Matching

Prefer pattern matching in function heads.

Prefer

```elixir
def approve(%Payment{status: :pending} = payment) do
```

instead of

```elixir
if payment.status == :pending do
```

Prefer multiple function clauses over giant case statements.

---

# Error Handling

Never use exceptions as business logic.

Prefer

```elixir
{:ok, result}
{:error, reason}
```

Reasons should be explicit.

Good

:invalid_amount
:not_found
:expired
:timeout

Avoid

:error
:false
:nil

without context.

---

# with

Prefer `with` when chaining operations returning tagged tuples.

Good

```elixir
with {:ok, payment} <- validate(params),
     {:ok, payment} <- persist(payment),
     {:ok, _} <- notify(payment) do
  {:ok, payment}
end
```

Avoid deeply nested case expressions.

---

# Structs

Prefer structs for domain entities.

Good

```elixir
%Payment{}
```

Avoid passing generic maps throughout the application.

---

# Maps

Maps are appropriate for:

External APIs

JSON

Configuration

Temporary transformations

They should not replace domain models.

---

# Configuration

Avoid

Application.get_env()

inside domain logic.
Inject dependencies instead.

---

# Dates and Time

Avoid

DateTime.utc_now()

inside business logic.
Inject clocks when deterministic behavior is required.

---

# UUIDs

Avoid generating IDs inside domain functions.
Inject ID generators whenever possible.

---

# Atoms

Never call

String.to_atom()

with external input.

Use

String.to_existing_atom()

only when appropriate.
Prefer explicit mappings.

---

# Enum vs Stream

Use Enum for small collections.
Use Stream for large or lazy pipelines.
Avoid premature optimization.

---

# Recursion

Prefer Enum unless recursion improves readability.
Tail recursion should only be used when necessary.

---

# Documentation

Every public module should contain:

@moduledoc

Every public function should contain:

@doc

Every public API should define:

@spec

Documentation should explain WHY.
Not WHAT.

---

# Logging

Logs should be structured.

Never log:

Passwords
Tokens
PII
Secrets
Large payloads

---

# Telemetry

Business operations should emit telemetry events.
Telemetry is preferred over custom instrumentation.

---

# Formatting

Always use
mix format
Never manually format code differently.

---

# Static Analysis

Always run
mix compile --warnings-as-errors
Use Credo.
Use Dialyzer when appropriate.

---

# Code Duplication

Avoid duplication.
Extract common behavior only after duplication becomes evident.
Avoid premature abstractions.

---

# Comments

Prefer expressive code.
Comments should explain WHY.
Never explain obvious code.

Bad

```elixir
# increment counter
counter + 1
```

Good

```elixir
# Retry attempts are limited by BACEN requirements.
```

---

# Macros

Avoid macros unless absolutely necessary.
Prefer functions.
Metaprogramming should be rare.

---

# Processes

Do not create processes without a reason.
GenServer is not a service object.
Only use processes when state or concurrency is required.

---

# Security

Validate all external input.
Never trust client data.
Never expose sensitive information.
Avoid atom leaks.

---

# Performance

Measure before optimizing.
Prefer readability first.
Optimize only proven bottlenecks.

---

# Definition of Done

Code is only complete when:

✓ Formatted
✓ Compiles without warnings
✓ Tests pass
✓ Public APIs documented
✓ Reviewed
✓ No unnecessary complexity
✓ No obvious duplication
✓ Idiomatic Elixir
✓ Production-ready

---

# Related Documents

- [domain.md](domain.md)
- [architecture.md](architecture.md)
- [testing.md](testing.md)
- [phoenix.md](phoenix.md)
