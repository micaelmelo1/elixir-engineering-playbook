# Architecture Guidelines

## Purpose

This document defines the architectural principles for Elixir applications.

Architecture is about controlling dependencies.

The goal is to keep the system easy to understand, easy to change, and easy to test.

Frameworks, databases, queues and external APIs are implementation details.

Business rules are the center of the application.

---

# Core Principles

The architecture should maximize:

- cohesion;
- modularity;
- testability;
- replaceability;
- readability.

Every dependency should point toward the business rules.

---

# Dependency Rule

Dependencies always point inward.

```
Web

↓

Application

↓

Domain

↑

Infrastructure
```

The domain knows nothing about Phoenix.

The domain knows nothing about Ecto.

The domain knows nothing about HTTP.

The domain knows nothing about Redis.

---

# Layers

## Domain

Contains business rules.

Examples:

Payment

Invoice

Customer

FeeCalculator

Validator

Policies

The domain should be pure whenever possible.

---

## Application

Coordinates use cases.

Examples:

CreatePayment

CancelPayment

ApprovePayment

GenerateInvoice

The application layer orchestrates the domain.

It should not contain business rules.

---

## Infrastructure

Implements external concerns.

Examples:

Repositories

HTTP Clients

Redis

Kafka

Email

S3

External APIs

Database

Everything here can be replaced.

---

## Delivery Layer

Phoenix

Controllers

Channels

LiveView

GraphQL

CLI

Background Jobs

This layer translates external requests into application use cases.

---

# Contexts

Phoenix Contexts are application boundaries.

Contexts should represent business capabilities.

Examples:

Payments

Accounts

Customers

Invoices

Notifications

Avoid creating contexts around technical concepts.

Bad

Database

Redis

API

Utils

---

# Behaviours

Every external dependency should have a behaviour.

Example

PaymentProvider

NotificationService

Storage

Clock

IdGenerator

Repositories

Behaviours belong close to the domain or application layer.

Implementations belong to infrastructure.

---

# Ports and Adapters

Use Hexagonal Architecture.

Ports define contracts.

Adapters implement contracts.

```
Domain

↓

Port (Behaviour)

↓

Adapter

↓

External System
```

The domain should never know which adapter is being used.

---

# Dependency Injection

Inject implementations.

Avoid global configuration inside business rules.

Prefer passing behaviours or options.

---

# Modules

Modules should be cohesive.

Avoid modules that perform unrelated responsibilities.

Large modules usually indicate multiple responsibilities.

---

# Processes

Do not create processes to organize code.

Create processes to model concurrency.

GenServer is a runtime abstraction.

Not an architectural abstraction.

---

# State

Prefer immutable state.

Persistent state belongs in storage.

Transient state belongs in processes only when necessary.

Avoid hidden mutable state.

---

# Domain First

Business rules come first.

Frameworks come second.

If Phoenix disappeared tomorrow, the domain should continue working.

---

# External Services

Treat every external system as unreliable.

Design for:

timeouts

retries

idempotency

observability

failure isolation

---

# Error Boundaries

Convert infrastructure errors into domain errors.

Avoid leaking HTTP or database errors into business rules.

Good

{:error, :provider_unavailable}

Avoid

{:error, %HTTPoison.Error{}}

---

# Composition

Compose systems from small modules.

Small modules are easier to understand, test and replace.

---

# Cross-cutting Concerns

Logging

Metrics

Tracing

Telemetry

Authentication

Authorization

These should not pollute business rules.

Keep them isolated whenever possible.

---

# Scalability

Prefer architectures that allow independent evolution.

A module should evolve without forcing unrelated modules to change.

---

# Anti-patterns

Avoid:

God modules

Shared mutable state

Circular dependencies

Framework-driven architecture

Large contexts

Utility modules

Global configuration

Database-centric design

---

# Checklist

Before introducing a new module ask:

- Does it have one responsibility?
- Does it belong to this layer?
- Does it depend only inward?
- Can it be tested in isolation?
- Can it be replaced?
- Does it expose a clear API?
- Does it hide implementation details?

If any answer is "no", reconsider the design.

---

# Related Documents

- philosophy.md
- coding_guidelines.md
- domain.md
- otp.md
- phoenix.md