# Elixir Engineering Playbook

A comprehensive engineering playbook for building high-quality Elixir and Phoenix applications.

This repository provides a set of engineering guidelines, architectural principles, coding standards, examples, templates, and AI instructions that can be reused across multiple projects.

The goal is not only to produce working software, but to produce software that is:

- Maintainable
- Testable
- Idiomatic
- Scalable
- Observable
- Secure
- Easy to evolve

---

# Philosophy

The playbook follows the following principles:

- Functional Programming
- SOLID
- Clean Architecture
- Hexagonal Architecture
- Domain Driven Design (lightweight)
- Explicitness over magic
- Simplicity over cleverness

---

# Repository Structure

```
docs/
```

Engineering documentation.

```
examples/
```

Reference implementations.

```
templates/
```

Reusable templates.

```
checklists/
```

Development and review checklists.

---

# AI Support

This repository includes instructions for AI coding assistants.

Supported:

- Claude Code
- OpenAI Codex
- Cursor
- Gemini CLI
- Windsurf

---

# Main Principles

- Small modules
- Small functions
- Explicit error handling
- Pure functions whenever possible
- Side effects only at the boundaries
- Behaviour driven architecture
- Comprehensive testing
- Self-review before completion

---

# Development Workflow

1. Understand the problem.
2. Understand the current architecture.
3. Reuse existing patterns.
4. Design before coding.
5. Implement.
6. Write tests.
7. Review the implementation.
8. Refactor.
9. Run quality checks.
10. Deliver.

---

# Quality Gate

Every implementation should satisfy:

- Compiles without warnings
- All tests passing
- Proper documentation
- Idiomatic Elixir
- No unnecessary complexity
- No duplicated code
- Reviewed before completion

---

This playbook is intended to be technology-agnostic whenever possible while remaining focused on the Elixir ecosystem.