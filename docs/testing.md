# Testing Guidelines

## Purpose

Testing is an integral part of software development.

Tests are not a separate phase.

Every feature should be designed with testability in mind.

Good architecture naturally leads to simple tests.

---

# Core Principles

Tests should be:

- Fast
- Deterministic
- Independent
- Readable
- Maintainable

A good test suite increases confidence without slowing development.

---

# Testing Philosophy

Test behavior.

Do not test implementation details.

The purpose of a test is to verify that the software behaves correctly from the
consumer's perspective.

Refactoring should not require rewriting tests unless behavior changes.

---

# Testing Pyramid

Prefer the following distribution:

70%

Unit Tests

20%

Integration Tests

10%

End-to-End Tests

Avoid relying primarily on integration or end-to-end tests.

---

# Unit Tests

Most tests should be unit tests.

Unit tests should:

- run in milliseconds;
- not require external services;
- execute in isolation;
- validate business rules.

Unit tests should not require:

- Phoenix
- Database
- Redis
- HTTP
- Kafka
- RabbitMQ
- Email providers

---

# Domain Testing

The domain should contain the highest concentration of tests.

Every business rule should be covered.

Examples:

- validations
- calculations
- state transitions
- policies
- invariants

---

# Integration Tests

Use integration tests to verify interaction between components.

Examples:

Repository

Ecto

Phoenix

External adapters

Telemetry

Avoid testing business rules here.

---

# End-to-End Tests

Keep end-to-end tests focused.

Only validate critical user journeys.

Avoid excessive E2E coverage.

---

# Test Structure

Follow the Arrange → Act → Assert pattern.

Example

Arrange

Create input.

Act

Execute behavior.

Assert

Verify result.

Keep each section visually separated.

---

# Test Naming

Describe behavior.

Good

```elixir
test "approves a pending payment"
```

Good

```elixir
test "returns an error when the amount is negative"
```

Avoid

```elixir
test "payment test"
```

Avoid

```elixir
test "test approve"
```

---

# One Assertion per Behavior

Each test should verify one behavior.

Large tests usually indicate multiple responsibilities.

---

# Fixtures

Use fixtures only when they improve readability.

Avoid giant shared fixtures.

Prefer small builders.

---

# Factories

Factories should generate valid objects.

Allow overriding only necessary fields.

Avoid complex factory hierarchies.

---

# Builders

Builders are preferred when they improve readability.

Example

PaymentBuilder.approved()

PaymentBuilder.pending()

PaymentBuilder.failed()

---

# Test Data

Keep test data minimal.

Only create the data required by the test.

Avoid unnecessary setup.

---

# Edge Cases

Always test:

- invalid input
- nil values
- empty collections
- limits
- duplicates
- unexpected states

---

# State Transitions

Every state transition should be tested.

Example

Pending

↓

Approved

↓

Settled

↓

Cancelled

Both valid and invalid transitions should be covered.

---

# Error Testing

Every error path deserves a test.

Example

{:error, :invalid_amount}

{:error, :timeout}

{:error, :already_paid}

Avoid testing only the happy path.

---

# Property-Based Testing

Use StreamData whenever properties are more valuable than examples.

Examples

Money calculations

Parsers

Validators

Formatters

Algorithms

Prefer property tests for deterministic transformations.

---

# Mocking

Mock only external dependencies.

Examples

HTTP

Storage

Email

Clock

Payment Provider

Notification Provider

Avoid mocking the module under test.

---

# Mox

Prefer Mox for behaviours.

Avoid mocking concrete implementations.

Always mock contracts.

---

# Database Tests

Only test persistence behavior.

Avoid testing domain logic through the database.

---

# Phoenix Tests

Controllers should test:

- request validation
- response serialization
- status codes

Do not duplicate domain tests.

---

# LiveView Tests

Test behavior.

Avoid testing implementation details.

---

# Concurrency

When concurrent code exists, test:

- race conditions
- ordering
- retries
- timeouts
- failures
- supervision

---

# Idempotency

Whenever an operation is expected to be idempotent,
write explicit tests proving it.

---

# Deterministic Tests

Tests should always produce the same result.

Avoid:

DateTime.utc_now()

Random values

Global state

Sleeping

Network dependencies

Inject dependencies instead.

---

# Performance

Tests should execute quickly.

Prefer thousands of fast tests over a few slow ones.

---

# Flaky Tests

Flaky tests are bugs.

Do not ignore intermittent failures.

Fix them immediately.

---

# Coverage

Coverage is a metric.

Not a goal.

100% coverage does not imply correct software.

Prioritize meaningful coverage.

---

# Regression Tests

Every bug fix should include a regression test.

A bug should never return unnoticed.

---

# Code Review

Every Pull Request should verify:

- new behavior has tests;
- edge cases are covered;
- failures are tested;
- tests remain readable;
- duplicated tests are avoided.

---

# Test Quality Checklist

Before considering tests complete:

✓ Happy path covered

✓ Failure paths covered

✓ Edge cases covered

✓ Deterministic

✓ Independent

✓ Fast

✓ Readable

✓ Maintainable

✓ No duplicated setup

✓ No unnecessary mocks

---

# Recommended Tools

- ExUnit
- Mox
- StreamData
- ExCoveralls (optional)
- Bypass
- Faker (only when useful)

---

# Anti-patterns

Avoid:

Testing private functions

Testing implementation details

Sleeping in tests

Randomized assertions

Huge fixtures

Shared mutable state

Mocking everything

Database-heavy unit tests

Assertions without intent

---

# Definition of Done

A feature is only complete when:

- Business rules are tested.
- Failure scenarios are tested.
- Edge cases are tested.
- Tests pass consistently.
- Tests are easy to understand.
- Tests provide confidence for future refactoring.

---

# Related Documents

- philosophy.md
- architecture.md
- domain.md
- coding_guidelines.md
- review.md