# Elixir Coding Guidelines

## Purpose

This document defines the engineering standards for writing maintainable,
idiomatic, testable, and production-ready Elixir code.

These guidelines should be followed across all projects using this playbook.

The primary goals are:

- Readability
- Simplicity
- Maintainability
- Testability
- Explicitness
- Consistency

---

# Core Principles

Every implementation should be:

- Easy to understand
- Easy to modify
- Easy to test
- Easy to review

Always optimize for the next developer who will read the code.

Software is read much more often than it is written.

---

# General Rules

## Prefer simplicity

Choose the simplest solution that correctly solves the problem.

Avoid unnecessary abstractions.

Avoid "clever" code.

Good code should feel boring.

---

## Be explicit

Explicit code is easier to understand than implicit behavior.

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

## Keep modules focused

A module should have one clear responsibility.

Good

PaymentValidator

PaymentRepository

PaymentProcessor

PaymentNotifier

Bad

PaymentManager

Utils

Helpers

Common

---

## Keep functions small

Functions should usually fit on one screen.

Long functions should be split.

A function should communicate one idea.

---

## Prefer composition

Compose multiple small functions.

Avoid giant functions.

Prefer

validate()

↓

calculate()

↓

persist()

↓

notify()

instead of one 300-line function.

---

# Naming

## Modules

Modules represent concepts.

Good

Payment

PaymentValidator

PaymentProvider

Invoice

Customer

---

Avoid

Manager

Processor

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

cancel()

calculate_fee()

send_payment()

Bad

do_work()

process_data()

execute()

handle()

run()

without proper context.

---

## Variables

Variable names should communicate intent.

Good

payment

customer

expiration_date

retry_count

Bad

data

value

item

obj

tmp

---

# Pattern Matching

Pattern matching is one of Elixir's greatest strengths.

Prefer

```elixir
def approve(%Payment{status: :pending} = payment) do
```

instead of

```elixir
if payment.status == :pending do
```

---

Prefer multiple function clauses.

Good

```elixir
def process(%Payment{status: :pending} = payment)

def process(%Payment{status: :approved})

def process(%Payment{status: :cancelled})
```

Avoid giant case statements whenever multiple clauses improve readability.

---

# Functions

Prefer pure functions.

Good

```elixir
calculate_total(items)
```

Bad

```elixir
calculate_total_and_save(items)
```

---

Avoid hidden side effects.

A function should do what its name suggests.

---

# Side Effects

Keep side effects at the application's boundaries.

Examples:

Database

HTTP

Redis

Filesystem

Logger

Telemetry

Kafka

RabbitMQ

Email

Domain logic should remain pure whenever possible.

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

Prefer with when chaining operations returning tagged tuples.

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

# Behaviours

Every external dependency should have a behaviour.

Examples

PaymentProvider
Storage
Cache
Notification
Clock
IdGenerator
Repositories

---

# Dependency Injection

Depend on behaviours.
Not implementations.

Good

```elixir
@provider Application.compile_env(...)
```

Better

```elixir
process(payment, provider)
```

or

```elixir
process(payment, opts)
```

Domain code should not depend on runtime configuration.

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

# Refactoring

Leave the codebase better than you found it.
Improve naming.
Reduce duplication.
Simplify logic.
Increase cohesion.
Reduce coupling.

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

# Guiding Principle

Whenever there is doubt between two implementations,
choose the one that is:

- simpler;
- clearer;
- more explicit;
- easier to test;
- more idiomatic;
- easier to maintain.