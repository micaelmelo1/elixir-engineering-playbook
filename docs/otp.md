# OTP Engineering Model

## Purpose

This document defines how systems behave at runtime on the BEAM.

It covers supervision trees, GenServer patterns, process ownership, state
isolation, restart strategies and crash boundaries.

It describes runtime structure only. It does not redefine failure philosophy or
composition — those are canonical in the principles.

---

# References

Failure and recovery philosophy:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Process and system composition:
[docs/principles/composition.md](principles/composition.md)

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

# Process Ownership Rules

Every piece of runtime state has exactly one owning process.

- state is mutated only by its owner
- other processes interact through messages
- no shared mutable state across processes

Ownership makes concurrency reasoning local.

---

# State Isolation Model

A process holds only valid, consistent state.

- initialize state fully in `init/1`
- reject transitions that would produce invalid state
- if state becomes inconsistent, crash instead of patching it

State recovery is the supervisor's job, not the process's.

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

# Anti-patterns

Avoid:

- GenServers used as service objects
- business logic inside callbacks
- shared mutable state between processes
- unbounded process spawning without supervision
- rescuing errors that should crash
- supervision trees that mirror module structure instead of failure domains

---

# Checklist

Before adding a process ask:

- Does this need to be a process at all?
- What state does it own?
- Who supervises it?
- What is its restart strategy?
- What is isolated when it crashes?
- Is the domain logic kept outside the callbacks?

---

# Related Documents

- [concurrency.md](concurrency.md)
- [architecture.md](architecture.md)
- [observability.md](observability.md)
