# Performance Engineering

## Purpose

This document defines how to scale correctly in Elixir.

It covers the BEAM performance model, the latency/throughput axes, scheduler
behavior, process and memory limits, shared state, bottleneck detection and
optimization strategy.

Performance work is measurement-driven. It refines the design; it does not
justify abandoning the principles. This document decides *what to optimize and
how the runtime behaves under load*; *how performance is measured* (metrics,
histograms, percentiles) is canonical in
[observability.md](observability.md#metrics-taxonomy).

---

# References

Composition and small units enable local optimization:
[docs/principles/composition.md](principles/composition.md)

Immutability and predictability shape performance behavior:
[docs/principles/functional_programming.md](principles/functional_programming.md)

Runtime and concurrency context:
[otp.md](otp.md) · [concurrency.md](concurrency.md)

---

# BEAM Performance Model

The BEAM favors many small, isolated, preemptively scheduled processes.

- concurrency is cheap; parallelism scales with schedulers/cores
- work is spread across lightweight processes, not threads
- a single busy process is a bottleneck regardless of core count

Design for many cooperating processes rather than one large one.

---

# Latency vs Throughput

Throughput (work per second) and latency (time per unit of work) are distinct
axes that trade off. Optimizing one can degrade the other — batching raises
throughput but adds latency; more concurrency raises throughput but can worsen
tail latency under contention.

- decide which axis the requirement targets before optimizing
- for financial operations, tail latency (p99/p999) is a first-class SLO, not
  an average — a request that is usually fast but occasionally seconds-slow is
  a real defect
- measure latency as a distribution, never as a mean — the histogram/percentile
  discipline is canonical in
  [observability.md](observability.md#metrics-taxonomy)

Optimize the axis the requirement names, and re-check the other did not regress.

---

# Scheduler Starvation

The BEAM preempts Elixir/Erlang code by reductions, so no single process
monopolizes a scheduler. Native code is not preempted.

- a long-running NIF or BIF blocks its scheduler thread until it returns,
  starving every other process on that scheduler — keep NIFs short or run them
  on dirty schedulers
- tight CPU loops, huge regexes, and large binary/JSON work on a normal
  scheduler delay unrelated work; chunk them or move them off the request path
- `System.schedulers_online/0` bounds real parallelism — spawning more busy
  processes than schedulers adds contention, not speed

A starved scheduler shows up as latency on work that is not itself slow.

---

# Process Limits

Processes are cheap but not free.

- a serialized GenServer is a throughput ceiling — shard or pool when it
  saturates
- watch mailbox growth; a growing mailbox signals a bottleneck — the mechanism
  that bounds it, backpressure, is canonical in
  [concurrency.md](concurrency.md#backpressure-strategies)
- avoid selective `receive` against a large mailbox — matching a specific
  message scans the whole mailbox (O(n)), so a large mailbox makes every
  receive slower
- bound concurrency (pools, `async_stream` limits) instead of spawning without
  limit

Measure per-process reductions and message queue length to find hot processes.

---

# Shared State and ETS

A serialized process protects one owner's state but serializes every access.
When state is read-heavy and shared, ETS removes the serialization point.

- single-writer mutable state → keep it in its owning process (see
  [otp.md](otp.md#process-ownership-rules))
- shared, read-mostly state (caches, lookup tables) → ETS with
  `read_concurrency`, allowing concurrent lock-free reads instead of funneling
  every read through one process
- ETS is not free: values are copied in and out, there is no transaction across
  tables, and invalidation is manual — treat it as a designed cache layer, not
  ad-hoc shared mutable state ([otp.md](otp.md#process-ownership-rules))
- concurrent writes to ETS still need a safety strategy — see
  [concurrency.md](concurrency.md#race-condition-handling)

Reach for ETS to break a proven serialization bottleneck, not by default.

---

# Memory Behavior

Immutable data means transformations allocate.

- large messages are copied between processes — pass identifiers or minimal data
- binaries over 64 bytes are reference-counted and shared; beware leaks from
  retained large binaries
- prefer streaming over materializing large collections in memory

Understand per-process heaps and garbage collection before tuning them.

---

# Bottleneck Detection

Find the bottleneck before changing anything.

- start from observability signals (latencies, telemetry, mailbox sizes) —
  see [observability.md](observability.md)
- profile with the right tool (`:observer`, `:recon`, benchmarks)
- identify the single limiting resource: CPU, memory, a process, or I/O

Optimize the proven bottleneck, not the suspected one.

---

# Optimization Strategy

Optimize in order:

1. measure and locate the real bottleneck
2. improve the algorithm or data flow
3. reduce copying and unnecessary work
4. parallelize or shard only if the work is actually parallelizable
5. re-measure to confirm the gain

Readability first. Optimize proven bottlenecks only, and keep the composition
intact so units remain replaceable and testable.

---

# Anti-patterns

Avoid:

- optimizing without measurement
- routing all load through a single serialized process
- unbounded parallelism
- blocking a scheduler with a long NIF or tight native loop
- optimizing average latency while ignoring the tail (p99/p999)
- using ETS as ad-hoc shared mutable state instead of a designed cache
- selective `receive` against a large mailbox
- copying large payloads between processes
- materializing large datasets instead of streaming
- sacrificing clarity for unproven micro-gains

---

# Checklist

Before optimizing ask:

- Do I have a measurement identifying the bottleneck?
- Which axis am I optimizing — latency or throughput — and did the other
  regress?
- Is a single process serializing the load, and would ETS remove it?
- Could a NIF or native call be starving a scheduler?
- Is memory copying or retention the cause?
- Is the work genuinely parallelizable?
- Did I re-measure after the change?
- Did the change preserve readability and composition?

---

# Related Documents

- [concurrency.md](concurrency.md)
- [otp.md](otp.md)
- [observability.md](observability.md)
- [ecto.md](ecto.md)
