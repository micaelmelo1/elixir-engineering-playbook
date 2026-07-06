# Observability Layer

## Purpose

This document defines how to understand system behavior in production.

It covers logging strategy, telemetry usage, tracing design, error visibility
and the production debugging model.

Observability is a cross-cutting concern. It reports on the system; it does not
belong inside business rules.

---

# References

Crashes as signals, not suppressed failures:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Isolating cross-cutting concerns from the domain:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

---

# Logging Strategy

Logs should be structured and intentional.

- attach context (request id, correlation id, entity id)
- log at meaningful boundaries, not inside tight loops
- use levels deliberately: `debug`, `info`, `warning`, `error`

Never log secrets, tokens, credentials, PII or large payloads.

Logging happens at the edges. The domain returns explicit results; the boundary
decides what to log.

---

# Telemetry Usage

Emit `:telemetry` events for meaningful business and system operations.

- start/stop/exception spans for use cases
- measurements for durations, counts and sizes
- stable event names and metadata contracts

Telemetry is preferred over ad-hoc instrumentation. Attach handlers (metrics,
logs) outside the code that emits the events.

---

# Tracing Design

Propagate a correlation identifier across process and service boundaries.

- generate it at the entry point (delivery layer)
- carry it through messages and async work
- include it in logs and telemetry metadata

Tracing turns isolated events into a coherent story across processes.

---

# Error Visibility Rules

Failures must be visible, not swallowed.

- expected errors (`{:error, reason}`) are counted and surfaced with context
- unexpected errors crash and are reported, per the let-it-crash reference
- never rescue solely to hide a failure

A crash is a signal. Make it visible; do not suppress it.

---

# Production Debugging Model

Design so that production questions can be answered without a debugger.

- can you trace a single request end to end?
- can you see error rates and latencies per use case?
- can you correlate a crash with the input that caused it?

If a question cannot be answered from logs, telemetry and traces, the
instrumentation is incomplete.

---

# Anti-patterns

Avoid:

- logging inside domain/business logic
- unstructured, context-free log lines
- logging secrets or PII
- swallowing errors to keep dashboards green
- per-call custom instrumentation instead of telemetry
- traces that break across process boundaries

---

# Checklist

Before shipping a feature ask:

- Are operations emitting telemetry?
- Are logs structured and free of sensitive data?
- Is a correlation id propagated across boundaries?
- Are failures visible with enough context to act?
- Can this feature be debugged in production from signals alone?

---

# Related Documents

- [otp.md](otp.md)
- [concurrency.md](concurrency.md)
- [performance.md](performance.md)
