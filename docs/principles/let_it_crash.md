# Let It Crash Principle (BEAM Philosophy)

## Purpose

“Let it crash” is a core principle of the BEAM ecosystem.

It defines how systems should behave under failure conditions.

Instead of trying to prevent all errors, we design systems that:

- fail fast
- recover automatically
- remain consistent under failure
- isolate faults

---

# Core Idea

> It is better for a process to crash than to continue in an inconsistent state.

---

# Why This Exists

Traditional systems attempt to:

- catch all errors
- recover manually
- hide failures
- continue execution at all costs

This leads to:

- corrupted state
- unpredictable behavior
- hidden bugs
- cascading failures

The BEAM takes a different approach:

> Fail fast, recover safely.

---

# Failure is Expected

In Elixir systems:

- failures are normal
- crashes are not exceptional
- recovery is automatic

We design assuming that:

- processes will fail
- networks will fail
- external APIs will fail
- databases will fail

---

# Process Isolation

Each process is isolated.

If one process crashes:

- it does NOT affect others
- supervisors handle recovery
- system stability is preserved

This is the foundation of BEAM reliability.

---

# Supervisors

Supervisors are responsible for recovery strategies.

They define:

- restart strategy
- restart limits
- process hierarchy

Common strategies:

- one_for_one
- rest_for_one
- one_for_all

---

# Crash vs Error Handling

## Bad: hiding errors

```elixir
try do
  risky_operation()
rescue
  _ -> :ok
end
```

This leads to silent failures.

---

## Good: let it crash

```elixir
risky_operation()
```

If it fails:

- process crashes
- supervisor restarts it
- system remains consistent

---

# When NOT to Crash

Not all errors should crash processes.

At the crash-decision level we distinguish two buckets:

## 1. Expected errors (business logic)

Example:

- invalid input
- insufficient balance
- validation failure

These should return:

```elixir
{:error, reason}
```

---

## 2. Unexpected errors (system failure)

Example:

- bad match
- corrupted state
- external system crash
- unexpected exception

These SHOULD crash the process.

---

The full error taxonomy — validation, business rule, infrastructure, and
programmer errors, plus how each is represented and propagated across
layers — is canonical in
[error_handling.md](../error_handling.md). This document only defines the
crash-or-return decision; it does not redefine error classification.

---

# Design Principle

> Use `{:error, reason}` for expected failures.
> Use crashes for unexpected failures.

---

# Process State Integrity

A process should only hold:

- valid state
- consistent state

If state becomes invalid:

> crash immediately

Do not attempt partial recovery inside the process.

---

# Supervisors and Recovery

Supervisors ensure:

- automatic restart
- clean state reinitialization
- fault isolation

This creates self-healing systems.

---

# Fault Isolation

One process failure should not:

- corrupt global state
- affect unrelated processes
- cascade into system-wide failure

---

# Cascading Failure Prevention

Bad design:

- shared mutable state
- global caches without isolation
- tight coupling between processes

Good design:

- isolated processes
- message passing
- independent supervision trees

---

# Let It Crash vs Defensive Programming

Defensive programming tries to prevent all failures.

BEAM philosophy:

> Assume failure and recover instead.

---

# Error Boundaries

At the crash level, the boundary is simple: domain and application layers
return controlled errors; processes crash on the unexpected; supervisors
restart.

| Layer | Strategy |
|------|--------|
| Domain | `{:error, reason}` |
| Application | controlled errors |
| Processes | crash on unexpected errors |
| Supervisors | restart |

The full layer-by-layer error propagation contract — including how adapters
wrap dependency errors and how the delivery layer maps to HTTP — is
canonical in
[error_handling.md](../error_handling.md#error-propagation-across-layers).

---

# Phoenix Context

Controllers should:

- handle expected errors gracefully
- not rescue unexpected system errors

Unexpected errors should bubble up.

---

# Example

## Good

```elixir
def process_payment(payment) do
  case validate(payment) do
    {:ok, payment} -> approve(payment)
    {:error, reason} -> {:error, reason}
  end
end
```

---

## Bad

```elixir
def process_payment(payment) do
  try do
    approve(payment)
  rescue
    _ -> {:error, :failed}
  end
end
```

This hides real system problems.

---

# When to Use `try/rescue`

Only for:

- external library inconsistencies
- truly unexpected runtime failures
- boundary protection (rare cases)

Never for business flow.

---

# System Resilience Model

BEAM systems achieve resilience through:

- supervision trees
- process isolation
- message passing
- restart strategies

Not through try/catch logic everywhere.

---

# Anti-patterns

Avoid:

- swallowing exceptions
- global error handlers that hide failures
- overly defensive code
- silent recovery logic
- retry loops inside domain logic
- ignoring crashes

---

# Observability Alignment

Crashes should be:

- logged
- traced
- monitored

But not suppressed.

A crash is a signal, not a failure of design.

---

# Testing Alignment

Let-it-crash systems are easier to test because:

- failures are explicit
- state resets are deterministic
- isolation reduces side effects

---

# Supervisory Thinking

When designing a system, always ask:

- What happens if this process crashes?
- Who restarts it?
- Is state safely recoverable?
- Is failure isolated?

---

# Guiding Principle

> Do not fear crashes.
> Design for recovery instead.