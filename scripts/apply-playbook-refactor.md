# Elixir Engineering Playbook - Structural Refactor (Agent CLI)

This document is intended for CLI agents or automated coding assistants.

Its goal is to normalize and refactor the existing repository after initial
manual creation of documentation files.

---

# Context

The repository currently contains:

- manually pasted documentation files
- duplicated concepts across multiple documents
- inconsistent cross-references
- missing structural normalization between principles and core docs

This refactor ensures the repository becomes a **single-source-of-truth
engineering handbook**.

---

# GLOBAL RULES

Apply all changes with the following constraints:

- Do NOT delete entire files unless explicitly instructed
- Prefer replacing duplicated content with references
- Ensure no concept is defined in more than one place
- Ensure all principle documents are canonical
- Ensure all non-principle docs reference principles instead of redefining them

---

# 1. PRINCIPLES ARE CANONICAL

All core concepts must live ONLY in:

```
docs/principles/
```

These files are the single source of truth:

- functional_programming.md
- hexagonal_architecture.md
- ddd.md
- solid.md
- let_it_crash.md
- composition.md
```

---

# 2. APPLY CROSS-REFERENCE NORMALIZATION

## 2.1 domain.md

### Replace entire conceptual sections with:

```
Domain modeling rules are defined in:
docs/principles/ddd.md

Domain behavior follows:
docs/principles/composition.md
```

Remove:

- all DDD explanations
- entities/value objects explanations
- aggregates explanations
- invariant descriptions (keep only references)

---

## 2.2 architecture.md

Replace content with:

```
Architecture is defined in:
docs/principles/hexagonal_architecture.md

Functional foundation:
docs/principles/functional_programming.md
```

Remove:

- layered explanations
- duplicated architecture definitions

---

## 2.3 coding_guidelines.md

Replace SOLID and behaviours section with:

```
Behaviours define contracts for external dependencies.

Design principles:
docs/principles/solid.md

Architecture principles:
docs/principles/hexagonal_architecture.md

Composition rules:
docs/principles/composition.md
```

Remove:

- full SOLID explanation
- duplicated behaviour definitions
- example lists unrelated to references

---

## 2.4 phoenix.md

Ensure:

- Phoenix is described ONLY as delivery layer
- No domain or DDD explanations remain

Replace conceptual sections with references:

```
Delivery layer is defined in:
docs/principles/hexagonal_architecture.md

Domain rules:
docs/principles/ddd.md
```

---

## 2.5 testing.md

Ensure:

- remove duplicated FP / architecture explanations
- keep testing philosophy ONLY

Add missing references:

```
Functional foundation:
docs/principles/functional_programming.md

Architecture model:
docs/principles/hexagonal_architecture.md

Domain model:
docs/principles/ddd.md
```

---

## 2.6 README.md

Normalize structure:

```
# Elixir Engineering Playbook

## Principles

- functional_programming.md
- hexagonal_architecture.md
- ddd.md
- solid.md
- let_it_crash.md
- composition.md

## Core Docs

- domain.md
- architecture.md
- coding_guidelines.md
- testing.md
- phoenix.md
```

---

# 3. ENSURE NO DUPLICATION RULE

After refactor:

## Must NOT exist:

- SOLID definitions outside solid.md
- DDD definitions outside ddd.md
- architecture explanations outside hexagonal_architecture.md
- FP explanations outside functional_programming.md
- crash handling explanations outside let_it_crash.md
- composition explanations outside composition.md

---

# 4. VALIDATION CHECKLIST

After applying changes:

- [ ] Each principle exists in exactly one file
- [ ] All other docs contain only references
- [ ] No duplicated explanations remain
- [ ] domain.md is purely referential
- [ ] architecture.md is purely referential
- [ ] coding_guidelines.md is lightweight and referential
- [ ] phoenix.md contains only delivery-layer concerns
- [ ] testing.md focuses only on testing philosophy

---

# 5. OUTCOME

The repository becomes:

- a modular engineering handbook
- fully normalized knowledge base
- safe for LLM consumption
- consistent single source of truth system