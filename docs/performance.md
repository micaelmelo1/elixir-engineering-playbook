# Performance Engineering

## Purpose

This document defines how to scale correctly in Elixir.

It covers the BEAM performance model, process limits, memory behavior,
bottleneck detection and optimization strategy.

Performance work is measurement-driven. It refines the design; it does not
justify abandoning the principles.

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

# Process Limits

Processes are cheap but not free.

- a serialized GenServer is a throughput ceiling — shard or pool when it
  saturates
- watch mailbox growth; a growing mailbox signals a bottleneck
- bound concurrency (pools, `async_stream` limits) instead of spawning without
  limit

Measure per-process reductions and message queue length to find hot processes.

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
- copying large payloads between processes
- materializing large datasets instead of streaming
- sacrificing clarity for unproven micro-gains

---

# Checklist

Before optimizing ask:

- Do I have a measurement identifying the bottleneck?
- Is a single process serializing the load?
- Is memory copying or retention the cause?
- Is the work genuinely parallelizable?
- Did I re-measure after the change?
- Did the change preserve readability and composition?

---

# Related Documents

- [concurrency.md](concurrency.md)
- [otp.md](otp.md)
- [observability.md](observability.md)
