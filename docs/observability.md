# Observability Layer

## Purpose

This document defines how to understand system behavior in production.

It covers logging strategy, telemetry usage, metrics taxonomy, tracing
design, alerting principles, audit trail, error visibility and the
production debugging model.

Observability is a cross-cutting concern. It reports on the system; it does not
belong inside business rules.

Observability reports on error occurrence; it does not classify or handle
errors. Error taxonomy and propagation are canonical in
[error_handling.md](error_handling.md).

---

# References

Crashes as signals, not suppressed failures:
[docs/principles/let_it_crash.md](principles/let_it_crash.md)

Isolating cross-cutting concerns from the domain:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Error taxonomy and propagation contract:
[docs/error_handling.md](error_handling.md)

Message contracts carrying correlation metadata:
[docs/concurrency.md](concurrency.md)

Domain events as the audit source of truth:
[docs/principles/ddd.md](principles/ddd.md)

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

# Metrics Taxonomy

Choose the metric type deliberately:

- **counter** — count of occurrences (requests, errors, events processed);
  monotonic, never decreases
- **gauge** — current value of something that goes up and down (queue
  depth, active connections, process count)
- **histogram** — distribution of a measurement (request duration, payload
  size); required for latency, never averaged

For each use case, capture the RED signals at minimum:

- **Rate** — throughput (requests/events per second)
- **Errors** — failure count, tagged by the error class from
  [error_handling.md](error_handling.md#error-taxonomy)
- **Duration** — latency distribution, not just an average

## Cardinality Control

Never use an unbounded value as a metric label or tag.

Avoid:

- `payment_id`, `user_id`, `request_id` as a label
- any label whose value space grows with production traffic

Unbounded labels multiply time series and can take down the metrics
backend. Put high-cardinality identifiers in logs and traces, not in metric
labels.

---

# Tracing Design

Propagate a correlation identifier across process and service boundaries.

- generate it at the entry point (delivery layer)
- carry it through messages and async work
- include it in logs and telemetry metadata

Tracing turns isolated events into a coherent story across processes.

## Correlation Propagation Mechanics

The correlation id travels as part of the message contract, not as
incidental state:

- include it in the message payload's metadata, per
  [concurrency.md](concurrency.md#message-contract-design) — never recover
  it from process dictionary or global state
- propagate it explicitly into `Task.async` closures and background jobs;
  it does not cross automatically
- when a message crosses a node boundary, carry the correlation id with it —
  do not assume it survives serialization implicitly

If a process cannot produce the correlation id of the work it is doing, the
propagation is incomplete.

---

# Error Visibility Rules

Failures must be visible, not swallowed.

- expected errors (`{:error, reason}`) are counted and surfaced with context
- unexpected errors crash and are reported, per the let-it-crash reference
- never rescue solely to hide a failure

A crash is a signal. Make it visible; do not suppress it.

This section covers visibility only — whether a failure is surfaced. Error
classification (validation, business rule, infrastructure, programmer) and
how each propagates across layers are canonical in
[error_handling.md](error_handling.md).

---

# Audit Trail

Audit trail is a distinct concern from operational observability. Do not
conflate them.

| | Operational observability | Audit trail |
|---|---|---|
| Purpose | debug and understand system behavior | prove what happened, when, and by whom |
| Retention | short, cost-bounded | long, driven by compliance requirements |
| Sampling | may be sampled or dropped under load | never sampled, never dropped |
| Mutability | logs may rotate or be reprocessed | append-only, immutable |
| Source | log lines, metrics, traces | domain events (see [principles/ddd.md](principles/ddd.md#domain-events)) |

Rules:

- financial or compliance-relevant state changes must emit a domain event
  that is persisted as the audit source of truth — do not derive an audit
  trail from log lines
- an audit record captures who initiated the action, what changed, and when
  — never reconstruct this from operational logs after the fact
- audit storage must not be subject to the same retention or sampling
  policy as operational logs

---

# Alerting Principles

An alert exists to trigger human action. If no action follows, it is noise.

- alert on symptoms that affect the system's purpose (error rate, latency,
  saturation), not on every internal condition
- every alert must be actionable — if the response is always "check and
  ignore," delete the alert
- distinguish paging alerts (wake someone up) from informational alerts
  (dashboard only)
- avoid duplicate alerts for the same underlying cause across layers

Alert fatigue from noisy, non-actionable alerts is more dangerous than
missing telemetry — it trains responders to ignore signals.

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
- unbounded-cardinality labels on metrics (ids, timestamps, free text)
- deriving an audit trail from operational logs instead of domain events
- applying log retention/sampling policy to audit records
- alerts with no actionable response
- paging on every alert instead of distinguishing severity

---

# Checklist

Before shipping a feature ask:

- Are operations emitting telemetry?
- Are metric labels bounded in cardinality?
- Are logs structured and free of sensitive data?
- Is a correlation id propagated through message contracts, async work, and
  node boundaries?
- Does this state change need an audit event, distinct from operational
  logs?
- Are failures visible with enough context to act?
- Would a new alert here be actionable, or is it noise?
- Can this feature be debugged in production from signals alone?

---

# Related Documents

- [otp.md](otp.md)
- [concurrency.md](concurrency.md)
- [performance.md](performance.md)
- [error_handling.md](error_handling.md)
- [security.md](security.md)
- [principles/ddd.md](principles/ddd.md)
