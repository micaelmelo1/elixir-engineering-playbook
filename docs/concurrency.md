# Concurrency Model

## Purpose

This document defines how concurrent and distributed execution is modeled
safely.

It covers message passing patterns, async workflows, backpressure, race
condition handling and process lifecycle design.

It builds on the runtime structure defined in [otp.md](otp.md). It does not
redefine failure or functional principles — those are canonical.

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

# Process Lifecycle Design

For every long-lived process define:

- startup: how it initializes valid state
- steady state: which messages it accepts
- shutdown: how it releases resources cleanly
- failure: what a crash isolates and how state is rebuilt

Lifecycle is part of the design, not an afterthought.

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

---

# Checklist

Before adding concurrent work ask:

- Who owns the state involved?
- Is the message contract explicit?
- Is there a timeout?
- Is backpressure bounded?
- Is the operation idempotent if it may be retried?
- What is isolated if this crashes?

---

# Related Documents

- [otp.md](otp.md)
- [performance.md](performance.md)
- [observability.md](observability.md)
