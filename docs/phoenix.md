# Phoenix Guidelines

## Purpose

Phoenix is the delivery layer of the application.

Its responsibility is to expose the application's capabilities through HTTP,
WebSockets, LiveView or APIs.

Phoenix is not the business layer.

Business rules belong to the Domain.

Use Cases belong to the Application layer.

Phoenix should remain as thin as possible.

---

# Core Principles

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

Every layer has a single responsibility.

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
- validations that belong to the domain
- repository access
- HTTP calls
- business decisions

---

# Parameter Validation

Controllers validate request shape.

The domain validates business rules.

Example:

Controller

✓ required field exists

✓ JSON format

✓ parameter types

Domain

✓ amount must be positive

✓ payment cannot be approved twice

✓ account must have balance

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

Phoenix Contexts expose application capabilities.

Contexts are not repositories.

Contexts are not controllers.

Contexts are the public API of the application.

Examples

Accounts

Payments

Customers

Notifications

---

# Context Responsibilities

Contexts should:

- expose use cases
- orchestrate workflows
- delegate business rules to the domain
- coordinate repositories

Avoid placing domain logic directly inside contexts.

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

LiveView should coordinate UI state.

Application and Domain remain unchanged.

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

---

```elixir
{:error, :invalid_amount}
```

↓

422

The domain should never know HTTP status codes.

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

Notifications

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

# Configuration

Phoenix configuration belongs to infrastructure.

The domain should not access endpoint configuration.

---

# Testing

Controllers should test:

- request validation
- routing
- response codes
- serialization

Do not duplicate domain tests.

See testing.md.

---

# Anti-patterns

Avoid:

Business logic in controllers

Business logic in LiveView

Business logic in Channels

Calling Repo directly from controllers

Calling external APIs from controllers

Fat controllers

Database-driven controllers

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

- architecture.md
- domain.md
- coding_guidelines.md
- testing.md
- ecto.md