# Runtime Docs Depth Pass — Session Handoff

## Context

Branch: `docs/next-level-engineering-framework`
PR: [#2 — docs: next-level engineering framework (OTP → review)](https://github.com/micaelmelo1/elixir-engineering-playbook/pull/2)

The user is auditing the "Engineering Framework" layer docs
(`docs/otp.md`, `docs/concurrency.md`, `docs/error_handling.md`,
`docs/observability.md`, `docs/performance.md`, `docs/security.md`,
`docs/ecto.md`, `docs/review.md`) one at a time, pushing each to
BEAM-grade / production-grade depth for a financial (Pix-like) system. The
standard: not just "documented," but "would a senior BEAM engineer reviewing
for production-readiness find a real gap here?"

---

## Done so far (this session, in order)

1. **`docs/otp.md`** (commit `5f467fb`) — added GenServer Role Clarification
   (GenServer is a runtime state container, NOT a service object/domain
   orchestrator/use case — business decisions never live in callbacks),
   Process Design Patterns (process-per-entity, process-per-request,
   registry, aggregation vs decomposition), full State Lifecycle Model
   (initialization, evolution, crash recovery, reset from source of truth).

2. **`docs/concurrency.md`** (commit `7b582bc`) — added Message Contract
   Design (versioned map structure, evolution strategy), Ordering Semantics
   (when ordering matters vs not, single-owner serialization), Failure
   Propagation (link vs monitor decision table, bounded/idempotent retry
   policy), and an explicit OTP/concurrency boundary statement in Purpose
   (`otp.md` = runtime structure, `concurrency.md` = runtime behaviour).

3. **`docs/error_handling.md`** (new file, commit `9759bba`) — error
   handling was fragmented across `let_it_crash.md` (crash-vs-error),
   `coding_guidelines.md` (tagged-tuple syntax), `phoenix.md` (HTTP
   mapping), none deep enough alone. Created this as the canonical source:
   four-class error taxonomy (validation / business rule / infrastructure /
   programmer), representation rules (atom vs struct reason, never leak
   dependency exceptions across boundaries), layer-by-layer propagation
   contract (domain → application → adapter → process → Phoenix),
   retryable/non-retryable classification (cross-refs `concurrency.md` for
   retry execution mechanics instead of duplicating), external error
   envelope shape. Trimmed the three source files to point here instead of
   restating. Wired into `README.md` index (Engineering Framework section).

4. **`docs/observability.md`** (commit `310672d`) — added Metrics Taxonomy
   (counter/gauge/histogram selection, RED signals, cardinality-control rule
   — unbounded labels like `payment_id` can take down a metrics backend),
   Correlation Propagation Mechanics (correlation id travels inside the
   message contract per `concurrency.md`, must be threaded explicitly into
   `Task.async`/background jobs/node boundaries), Audit Trail (distinct from
   operational observability — retention/sampling/mutability differ;
   compliance-relevant state changes emit a domain event as the audit
   source of truth, never derived from logs), Alerting Principles
   (actionable-only, paging vs informational, alert fatigue as a real
   hazard). Cross-referenced `error_handling.md` instead of restating error
   classification.

5. **`docs/security.md`** (commit `f86b80c`) — added Rate Limiting and Abuse
   Boundaries (new section: distinct from backpressure — abuse vs legitimate
   load, identity-keyed limits, `429`/fail closed, idempotency key as replay/
   double-execution defense for money-touching endpoints, distributed-limiter
   state must never degrade to "no limit"), deepened Authentication and
   Authorization (separate authn/authz, default-deny, defense-in-depth —
   enforce at the layer that performs the op not only the edge, scope to
   authenticated identity, authz failures feed the audit trail), and expanded
   Secrets Management with Runtime Exposure (BEAM-specific: crash dumps,
   `:observer`/`:sys.get_state` leak process state; redact under `Inspect`)
   and Rotation. Cross-referenced instead of duplicating: error taxonomy →
   `error_handling.md`, never-log-secrets/audit → `observability.md`,
   backpressure/retry → `concurrency.md`. Added reciprocal Related-Documents
   links in those three files. Respects `.agent/contract.md` (references
   principles/other docs; defines no principle concept).

6. **`docs/performance.md`** (commit `ea8060b`) — added Scheduler Starvation
   (BEAM preempts by reductions but native code is not — long NIF/BIF or tight
   native loop blocks a scheduler thread; dirty schedulers, chunk heavy binary/
   JSON work, `schedulers_online` as the real parallelism bound), Shared State
   and ETS (read-mostly shared state → ETS with `read_concurrency` to remove
   the single-process serialization point; references `otp.md#process-ownership-rules`
   for the cache discipline and `concurrency.md#race-condition-handling` for
   write safety), Latency vs Throughput (distinct trading-off axes; tail
   latency p99/p999 as a first-class financial SLO, never an average), and a
   selective-`receive` O(n) mailbox-scan note in Process Limits. Boundary made
   explicit: performance.md owns *what to optimize / how the runtime behaves
   under load*; *how it is measured* stays canonical in `observability.md`.
   Added reciprocal `otp.md` → `performance.md` link.

Also this session (governance, commits `ec0178e` + `8df4fba`): ran
`scripts/validate-playbook.md`, collapsed duplicated rules in
`coding_guidelines.md` into references, wired missing principle links, and —
per the user's structural decision — formalized a **two-tier canonical model**:
`principles/*` = theory, Engineering Framework docs = canonical operational
rules of their layer. `README.md` updated (committed); `.agent/contract.md` and
`scripts/validate-playbook.md` updated in place but remain **untracked** (the
user has kept all governance/handoff files out of git — do not commit them
without asking). The `error_handling.md`-as-canonical tension is now RESOLVED by
this model.

7. **`docs/ecto.md`** (commit `8a9fb83`) — added Concurrent Write Safety (new:
   process ownership from `otp.md`/`concurrency.md` only serializes in-memory
   state on one node — a database row is shared, persisted state multiple
   nodes can write concurrently, so the database itself must enforce safety;
   covers unique-constraint-over-check-then-insert, `optimistic_lock/3`,
   `SELECT ... FOR UPDATE`, idempotency-key uniqueness as the concurrency
   guarantee for money movement), Migration Safety (new: additive/backward-
   compatible migrations for rolling deploys, no same-deploy rename/drop of a
   column still in use, avoid long-held locks on hot tables via
   `CONCURRENTLY`/batching, rehearse against a production-sized copy), Query
   Cost under Query Isolation (N+1 via unpreloaded associations, `Repo.stream`
   for large sets — references `performance.md#memory-behavior`), `Ecto.Multi`
   added to Transaction Boundaries as the idiomatic composition tool (named
   steps, atomic rollback, side effects kept out of the multi). Rewrote Error
   Translation to *apply* (not duplicate) `error_handling.md`'s adapter-
   boundary contract — added the Ecto-specific mapping detail (match by
   constraint name, `NoResultsError`, `Postgrex.Error` classification) instead
   of restating the generic rule. Added reciprocal Related-Documents links in
   `error_handling.md`, `concurrency.md`, `performance.md`, `otp.md`.

8. **`docs/review.md`** (commit `3c957c4`) — the file was already reference-
   style (correct for an application doc under the two-tier model), but
   referenced none of `error_handling.md`, `concurrency.md`, or any of the
   Engineering Framework sections added this session. Added Review Risk
   Tiering (new: the one genuinely distinct review-process concept not owned
   elsewhere — money-touching changes, migrations, authz changes, and
   cross-context changes need a second, more deliberate pass and, for a
   financial system, a second reviewer with domain context), migration/
   concurrent-write-safety checks in the PR Checklist, authz-at-the-edge /
   missing-audit-event entries in Anti-pattern Detection, linked the existing
   bare "error tuples use explicit reasons" line to
   `error_handling.md#error-representation`, and updated the stale "defined
   once, in the principles" line to the two-tier model. Related Documents
   expanded to list (and reciprocally link back from) all 7 Engineering
   Framework docs plus `phoenix.md`.

**DEPTH PASS COMPLETE.** All 8 Engineering Framework / review docs (otp →
concurrency → error_handling → observability → security → performance → ecto →
review) are now at BEAM-grade / production-grade depth for a financial system,
cross-referenced instead of duplicated, and bidirectionally linked. All eight
doc commits are on `docs/next-level-engineering-framework` and pushed to
`origin`, reflected in PR #2.

Governance note (resolved this session): the user added `.agent/contract.md`
with a strict "concepts defined ONLY in docs/principles/*" rule, which created
a tension with `error_handling.md`'s canonical-source status. Resolved via a
**two-tier canonical model** (see commits `ec0178e`/`8df4fba`): `principles/*`
= theory, Engineering Framework docs = canonical operational rules of their
layer. `error_handling.md`, `security.md`, `performance.md`, and now
`ecto.md`'s new sections are all canonical under this model — no remaining
tension. `.agent/contract.md` and `scripts/validate-playbook.md` were updated
in place but remain **untracked** — do not commit them without asking.

---

## Methodology (repeat this per file)

1. Read the target file in full.
2. Identify what's already correct vs where a senior BEAM/production
   reviewer would find a real gap — not stylistic nits, structural gaps
   (missing taxonomy, missing boundary clarity, missing failure-mode
   coverage).
3. Check for overlap with other docs before adding new sections — if the
   concept already lives elsewhere (e.g. retry mechanics in
   `concurrency.md`, error taxonomy in `error_handling.md`), cross-reference
   with an anchor link instead of restating.
4. Patch incrementally with `Edit` — never rewrite the file wholesale. Match
   the existing terse, declarative style (short sections, `---` separators,
   Anti-patterns list, Checklist, Related Documents).
5. Update `Purpose` and `References` at the top of the file if new
   cross-doc boundaries were introduced.
6. Extend `Anti-patterns` and `Checklist` sections to cover the new content.
7. Verify every new relative link and `#anchor` actually resolves
   (`grep -n "^#\|^##"` on the target file to confirm heading → anchor
   slug matches).
8. Add the new/changed doc to `README.md`'s index if a new file was
   created, and add reciprocal links in `Related Documents` of files that
   now reference each other (keep the doc graph bidirectional).
9. Commit with a message explaining what gap was closed and why, not just
   what changed. Co-author trailer:
   `Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>`

If a structural decision is ambiguous (e.g., "does this need a new file or
can it patch existing ones?"), ask the user — don't decide silently. That's
what happened before creating `error_handling.md` (user chose "new file"
over "patch the 3 existing files").

---

## Remaining files to audit

None. The depth pass is complete.

---

## How to resume

The depth pass is complete — nothing left to pick up from "Remaining files."
If a new gap surfaces (e.g. after further review of PR #2), re-run the
Methodology section against the specific file. This handoff file is a durable
log of the pass and stays untracked; ask the user before deleting or
committing it.
