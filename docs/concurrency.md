# Concurrency Model

## Purpose

This document defines how concurrent and distributed execution is modeled
safely.

It covers message passing patterns, message contract design, ordering
semantics, async workflows, backpressure, failure propagation, and process
lifecycle design.

It builds on the runtime structure defined in [otp.md](otp.md). The boundary
is explicit:

- **otp.md** → structure of the runtime: process ownership, supervision
  trees, state lifecycle
- **this document** → behaviour of the runtime: message flow, communication
  patterns, ordering, distributed execution rules

It does not redefine failure isolation or functional principles — those are
canonical in their respective documents.

---

# References

Isolation and recovery philosophy:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Immutability and predictability:
[docs/principles/functional_programming.md](principles/functional_programming.md)

Runtime process structure:
[otp.md](otp.md)

---

# Message Passing Patterns

Processes communicate only through messages.

- request/response → synchronous call with a timeout
- fire-and-forget → asynchronous cast
- runtime signals and external events → info messages

Messages are immutable data. Never pass references to mutable state; there is
none.

Keep message contracts explicit and small. A message is a contract between
processes.

## Message Contract Design

Every inter-process message is a versioned contract.

Structure messages explicitly:

```elixir
%{
  type: :payment_requested,
  version: 1,
  payload: %{payment_id: id, amount: amount}
}
```

Rules:

- prefer maps with a `type` key over bare positional tuples at system
  boundaries
- include a `version` field when the message shape may evolve
- keep payloads minimal — pass identifiers, not full aggregates
- never embed mutable references or PIDs in the payload unless coordinating
  supervision

Evolution strategy:

- introduce a new message version alongside the existing one
- support both versions during the migration window
- remove the old version only after all producers and consumers have been
  updated

---

# Async Workflows

Model asynchronous work as supervised tasks or dedicated processes.

- use `Task.Supervisor` for concurrent, independent work
- use `Task.async_stream` for bounded parallel mapping
- always set timeouts
- decide explicitly what happens on failure: retry, drop, or escalate

Do not fire unsupervised tasks for work whose failure matters.

---

# Backpressure Strategies

A fast producer must never overwhelm a slow consumer.

Prefer:

- bounded queues and mailboxes
- demand-driven flow (`GenStage` / `Flow`) when pipelines are sustained
- explicit rate limiting at system boundaries

Unbounded buffering is a latent outage. Make limits explicit and observable.

---

# Race Condition Handling

Race conditions arise from shared state and unordered effects.

Prevent them by:

- giving each piece of state a single owning process (see [otp.md](otp.md))
- serializing conflicting operations through that owner
- making operations idempotent where ordering cannot be guaranteed

Do not rely on timing. Rely on ownership and explicit sequencing.

---

# Ordering Semantics

Not all work requires ordered execution. Decide explicitly.

Ordering matters when:

- state transitions must be sequential (e.g. payment lifecycle steps)
- financial settlement depends on causality
- downstream idempotency breaks under reordering

Ordering does not matter when:

- publishing notifications or alerts
- emitting log or analytics events
- fan-out read operations

Rules:

- when ordering matters → serialize through a single owning process
- when ordering does not matter → allow parallel execution freely
- never assume ordering across independent processes
- never assume ordering across nodes, even within the same cluster

---

# Process Lifecycle Design

For every long-lived process define:

- startup: how it initializes valid state
- steady state: which messages it accepts
- shutdown: how it releases resources cleanly
- failure: what a crash isolates and how state is rebuilt

Lifecycle is part of the design, not an afterthought.

---

# Failure Propagation

Process failures propagate through links and monitors. Choose deliberately.

| Mechanism | Behaviour | Use when |
|-----------|-----------|----------|
| `Process.link/1` | failure cascades bidirectionally | the two processes must live and die together |
| `Process.monitor/1` | failure is observed, not propagated | a process must react to another's exit without being affected |

Rules:

- never silently ignore a monitored process exit
- decide explicitly on failure: crash, compensate, or retry
- do not link processes across unrelated failure domains

Retry policy:

- retries must be bounded — define a maximum attempt count
- every retried operation must be idempotent
- do not retry at multiple layers for the same operation; the failure count
  amplifies
- in distributed flows, prefer a dead-letter queue or error handler over an
  unbounded retry loop

---

# Distribution Notes

Across nodes, treat every call as capable of failing or timing out.

- assume messages can be lost or delayed
- prefer idempotent, retryable operations
- design for partition and reconnection

Weak consistency across boundaries is acceptable; strong consistency belongs
inside a single owner.

---

# Anti-patterns

Avoid:

- shared mutable state across processes
- unbounded mailboxes and queues
- unsupervised tasks for critical work
- relying on `Process.sleep` for coordination
- ordering assumptions between independent processes
- blocking a process on a long synchronous call without a timeout
- implicit message contracts (bare tuples, undocumented shapes)
- silently ignoring monitored process exits
- linking processes across unrelated failure domains
- retrying at multiple layers without idempotency — failure amplifies
- unbounded retry loops across distributed boundaries

---

# Checklist

Before adding concurrent work ask:

- Who owns the state involved?
- Is the message contract explicit, typed, and versioned?
- Does this operation require ordered execution? If so, is it serialized
  through a single owner?
- Is there a timeout on every call?
- Is backpressure bounded?
- Is the operation idempotent if it may be retried?
- Is retry bounded? Is it applied at only one layer?
- Are process links and monitors chosen deliberately, not by default?
- What is isolated if this crashes?

---

# Related Documents

- [otp.md](otp.md)
- [performance.md](performance.md)
- [observability.md](observability.md)
- [error_handling.md](error_handling.md)
- [security.md](security.md)
- [ecto.md](ecto.md)
