# Elixir Engineering Playbook - Agent Contract

## RULE 1: Single Source of Truth (Two Canonical Tiers)

Every concept has exactly one canonical home, in one of two tiers:

- **Principle theory** — the *what/why* of a concept — is defined ONLY in
  `docs/principles/*`.
- **Operational rules** — the *how* of a runtime/layer concern — are defined
  ONLY in their owning Engineering Framework doc: `otp.md`, `concurrency.md`,
  `error_handling.md`, `ecto.md`, `observability.md`, `performance.md`,
  `security.md`.

No file may redefine a concept already canonical in either tier. A doc that is
not the canonical owner references the owner instead. An Engineering Framework
doc applies principle theory; it never redefines it.

---

## RULE 2: No Concept Duplication

If a concept already exists in its canonical owner:

- do NOT rewrite it
- do NOT re-explain it
- only reference it

---

## RULE 3: Reference-Only Application Docs

Application docs — Core Docs (`domain.md`, `architecture.md`,
`coding_guidelines.md`, `testing.md`, `phoenix.md`) and `review.md` — MUST:

- explain role in the system
- reference the canonical principle or Engineering Framework doc
- define no theory and no operational rule of their own

---

## RULE 4: No Architectural Drift

Agents MUST NOT:

- invent new layers
- rename existing patterns
- introduce alternative terminology

---

## RULE 5: Deterministic Output

Given the same input:

- output must be structurally consistent
- no stylistic variation of concepts

---

## RULE 6: Validation Required

After changes:

- ensure no duplication exists across docs
- ensure principles remain canonical