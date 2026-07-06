# Phoenix Guidelines

## Purpose

Phoenix is the delivery layer of the application.

Its responsibility is to expose the application's capabilities through HTTP,
WebSockets, LiveView or APIs.

Phoenix is not the business layer. It should remain as thin as possible.

---

# References

Delivery layer is defined in:
[docs/principles/hexagonal_architecture.md](principles/hexagonal_architecture.md)

Domain rules:
[docs/principles/ddd.md](principles/ddd.md)

Error classification and propagation contract:
[docs/error_handling.md](error_handling.md)

---

# Core Responsibilities

Phoenix should:

- Receive requests
- Validate request format
- Call the appropriate application use case
- Serialize responses

Nothing more.

---

# Request Flow

```
HTTP Request

↓

Router

↓

Controller

↓

Application

↓

Domain

↓

Infrastructure

↓

Response

↓

Controller

↓

HTTP Response
```

Every layer has a single responsibility (see
[principles/solid.md](principles/solid.md)).

---

# Controllers

Controllers coordinate requests.

Controllers should never contain business rules.

Responsibilities:

- receive parameters
- validate request format
- authorize when applicable
- call the application layer
- serialize responses

Avoid:

- calculations
- domain validations
- repository access
- HTTP calls
- business decisions

---

# Parameter Validation

Controllers validate request shape.

Business validation belongs to the domain (see references above).

Controller

✓ required field exists

✓ JSON format

✓ parameter types

---

# Controllers Should Be Thin

A controller action should usually fit on one screen.

Good

```elixir
def create(conn, params) do
  with {:ok, payment} <- Payments.create(params) do
    render(conn, :show, payment: payment)
  end
end
```

Bad

200 lines of business logic.

---

# Contexts

Phoenix Contexts are the delivery-facing entry point to application capabilities.

They should:

- expose use cases
- delegate business rules to the domain

Domain and boundary rules are defined in
[docs/principles/ddd.md](principles/ddd.md).

---

# Views

Views (or JSON serializers) should transform data.

They should not execute business logic.

Good

format dates

rename fields

serialize structs

Avoid

fee calculations

permission decisions

business validations

---

# Routers

Keep routing simple.

Routes should express business capabilities.

Good

POST /payments

POST /payments/:id/cancel

GET /accounts/:id

Avoid exposing implementation details.

---

# LiveView

Use LiveView for interactive interfaces.

Keep business logic outside LiveView.

LiveView should coordinate UI state only.

---

# Channels

Channels should focus on communication.

Avoid placing business rules inside channels.

Treat channels similarly to controllers.

---

# Authentication

Authentication belongs to the delivery layer.

The domain should receive the authenticated identity.

Avoid accessing conn inside business logic.

---

# Authorization

Authorization may happen in:

Controllers

Policies

Application layer

Avoid scattering authorization logic.

---

# Error Handling

Convert domain errors into HTTP responses.

Example

```elixir
{:error, :not_found}
```

↓

404

```elixir
{:error, :invalid_amount}
```

↓

422

The domain should never know HTTP status codes.

The error envelope shape and the rule against leaking internal reasons to
clients are canonical in
[error_handling.md](error_handling.md#external-error-representation).

---

# Rendering

Controllers should render serialized data.

Avoid exposing internal structs directly.

Prefer dedicated JSON modules.

---

# Background Jobs

Controllers should return quickly.

Long-running work should be delegated.

Examples

Sending emails

Generating reports

Calling slow providers

Image processing

---

# Transactions

Database transactions belong outside controllers.

Controllers should never start transactions.

---

# Dependency Direction

Phoenix depends on:

Application

Domain

Infrastructure

The inverse must never happen.

---

# Testing

Controllers should test:

- request validation
- routing
- response codes
- serialization

Do not duplicate domain tests.

See [testing.md](testing.md).

---

# Anti-patterns

Avoid:

Business logic in controllers

Business logic in LiveView

Business logic in Channels

Calling Repo directly from controllers

Calling external APIs from controllers

Fat controllers

HTTP-aware domain code

---

# Checklist

Before implementing a Phoenix feature ask:

- Is the controller thin?
- Is the business logic in the domain?
- Is request validation separated from business validation?
- Is serialization isolated?
- Is HTTP hidden from the domain?
- Can the domain be reused without Phoenix?

---

# Related Documents

- [architecture.md](architecture.md)
- [domain.md](domain.md)
- [coding_guidelines.md](coding_guidelines.md)
- [testing.md](testing.md)
- [error_handling.md](error_handling.md)
