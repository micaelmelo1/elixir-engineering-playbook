# AI Engineering Instructions

These instructions apply to any AI coding assistant.

---

## Goal

Produce production-quality Elixir code.

Not just working code.

---

## Requirements

Every implementation must be:

- Idiomatic
- Tested
- Maintainable
- Explicit
- Simple
- Observable
- Secure

---

## Coding Process

1. Analyze.
2. Design.
3. Implement.
4. Test.
5. Review.
6. Refactor.
7. Validate.

Never skip review.

---

## Code Style

Prefer:

- pattern matching
- multiple function clauses
- structs
- behaviours
- composition
- immutable data

Avoid:

- deeply nested code
- giant modules
- giant functions
- unnecessary macros
- unnecessary processes

---

## Error Handling

Prefer:

```elixir
{:ok, value}

{:error, reason}
```

Avoid exceptions as business flow.

---

## Testing

Every feature should include:

- Unit tests

When appropriate:

- Integration tests

Whenever possible:

- Property-based tests

---

## Documentation

Public modules should contain:

- @moduledoc

Public functions should contain:

- @doc

Public APIs should include:

- @spec

---

## Final Checklist

Before considering any task complete:

✓ formatted
✓ compiled
✓ tests passing
✓ reviewed
✓ documented
✓ idiomatic