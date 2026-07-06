# OTP Engineering Model

## Purpose

This document defines how systems behave at runtime on the BEAM.

It covers supervision trees, GenServer patterns, process design patterns,
process ownership, state lifecycle, restart strategies and crash boundaries.

It describes runtime structure only. It does not redefine failure philosophy,
composition or layering — those are canonical in the principles.

---

# References

Failure and recovery philosophy:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Process and system composition:
[docs/principles/composition.md](principles/composition.md)

Layering (domain, application, adapters):
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

---

# When to Use a Process

Use a process to model concurrency, ownership of runtime state, or a failure
boundary.

Do not use a process to organize code. Modules and functions organize code.

A GenServer is a runtime abstraction, not a service object.

---

# Supervision Trees

Design the supervision tree before writing processes.

For each process, decide:

- who starts it
- who restarts it
- what state it owns
- what happens when it crashes

The tree expresses the failure model of the system.

---

# Restart Strategies in Practice

Choose a strategy from the child's dependency relationships:

- `one_for_one` — children are independent
- `rest_for_one` — later children depend on earlier ones
- `one_for_all` — children share a lifecycle and must restart together

Set `max_restarts` and `max_seconds` to match the failure domain. A restart
storm should escalate, not loop silently.

Strategy definitions and rationale: see the references above.

---

# GenServer Patterns

A GenServer should:

- own a single, well-defined piece of runtime state
- expose a small, intention-revealing client API
- keep callbacks thin

Keep business rules out of GenServer callbacks. Callbacks coordinate; the domain
decides. Delegate decisions to pure functions and store only the result.

Prefer `handle_call` for request/response, `handle_cast` for fire-and-forget,
`handle_info` for runtime signals.

---

# GenServer Role Clarification

A GenServer is NOT:

- a service object
- a domain orchestrator
- an application use case

A GenServer is:

> a runtime state container and message handler.

## Correct Layering

The layers are defined in
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md).
Their runtime placement is:

- Domain → pure business logic
- Application → use case orchestration
- GenServer → runtime execution and state ownership (a runtime adapter)
- Phoenix → delivery layer

A GenServer belongs to the runtime/infrastructure side. It does not implement a
use case; it executes one that the application layer coordinates.

## Rule

Never put business decisions inside GenServer callbacks.

GenServers execute decisions; they do not define them.

---

# Process Ownership Rules

Every piece of runtime state has exactly one owning process.

- state is mutated only by its owner
- other processes interact through messages
- no shared mutable state across processes

Ownership also implies:

- no external mutation of a process's state
- no shared ETS unless explicitly designed as a cache layer
- no cross-process state mutation

Ownership makes concurrency reasoning local.

---

# State Isolation Model

A process holds only valid, consistent state.

- initialize state fully in `init/1`
- reject transitions that would produce invalid state
- if state becomes inconsistent, crash instead of patching it

State recovery is the supervisor's job, not the process's.

---

# State Lifecycle Model

## Initialization

State must be fully valid at the end of `init/1`.

No partial or lazy initialization.

## Evolution

State changes only through explicit message handling.

Each transition must be deterministic and produce valid state.

## Crash Recovery

If state becomes invalid:

- crash immediately
- do not attempt repair inside the process

Recovery happens through supervisor restart (see
[docs/principles/let_it_crash.md](principles/let_it_crash.md)).

## Reset Behavior

After a restart:

- state is rebuilt from the source of truth
- never rely on previous in-memory state

---

# Failure Domains (Crash Boundaries)

A crash boundary is the set of processes that fail together.

Design boundaries so that:

- a crash is isolated to the smallest useful scope
- unrelated work is unaffected
- recovery reinitializes clean state

Group processes that must stay consistent together; separate those that must
fail independently.

---

# Process Design Patterns

## Process-per-Entity

Use one process per business entity when:

- state must be isolated
- updates must be serialized
- concurrency conflicts are likely

Examples:

- one process per Payment
- one process per Account

These are long-lived processes that own an entity's runtime state.

## Process-per-Request

Use short-lived processes when:

- computation is isolated
- no long-lived state is required

Examples:

- background calculation
- async orchestration

The process exists for the duration of the work and then exits.

## Registry Pattern

Use a registry when processes must be discovered dynamically.

- each process is addressable via a deterministic key
- avoid global state lookup

A registry maps business identity to a runtime process.

## Process Aggregation vs Decomposition

Do not aggregate unrelated responsibilities in a single process.

Split processes by:

- lifecycle (long-lived vs short-lived)
- ownership (which state it holds)
- failure domain (what must fail together)

If a process serves more than one of these, decompose it.

---

# Anti-patterns

Avoid:

- GenServers used as service objects or application use cases
- business logic inside callbacks
- shared mutable state between processes
- shared ETS treated as writable state instead of a designed cache
- aggregating unrelated responsibilities in one process
- relying on in-memory state surviving a restart
- unbounded process spawning without supervision
- rescuing errors that should crash
- supervision trees that mirror module structure instead of failure domains

---

# Checklist

Before adding a process ask:

- Does this need to be a process at all?
- Which design pattern fits (per-entity, per-request, registry)?
- Is it long-lived or short-lived?
- What state does it own?
- Who supervises it?
- What is its restart strategy?
- What is isolated when it crashes?
- Is state rebuilt from the source of truth after a restart?
- Is the domain logic kept outside the callbacks?
- Is the use case orchestrated by the application layer, not the GenServer?

---

# Related Documents

- [concurrency.md](concurrency.md)
- [architecture.md](architecture.md)
- [observability.md](observability.md)
- [error_handling.md](error_handling.md)
- [performance.md](performance.md)
