---
name: golang-clean-architecture
description: Use this skill when structuring or refactoring Go backend projects using Clean Architecture with domain, application, infrastructure, and delivery layers.
---

# Golang Clean Architecture

Use this skill to keep Go backend code structured, layered, and maintainable.

## Dependency Direction

```text
Delivery → Application → Domain ← Infrastructure
```

Rules:

* Dependencies must point inward.
* Domain must not depend on any outer layer.
* Application depends on Domain.
* Delivery depends on Application.
* Infrastructure implements interfaces from Domain or Application.

## Layer Responsibilities

### Domain

Contains core business rules.

Allowed:

* Entities
* Value Objects
* Domain Services
* Repository Interfaces
* Domain Errors
* Enums / Constants

Not allowed:

* SQL
* HTTP logic
* Framework code
* Database clients
* External service clients

### Application

Contains usecases and orchestration.

Allowed:

* Usecases
* Application DTOs
* Application Services
* Calls to Domain interfaces

Not allowed:

* SQL
* HTTP request/response logic
* Direct database access
* Framework handler logic
* Concrete infrastructure implementation

### Infrastructure

Contains technical implementations.

Allowed:

* Repository implementations
* SQL queries
* Database clients
* External service clients
* Cache clients
* Message broker clients

Not allowed:

* HTTP handlers
* Business rules
* Request/response formatting
* Usecase orchestration

### Delivery

Contains transport-specific code.

Allowed:

* HTTP handlers
* Routes
* Middleware
* Request parsing
* Response formatting
* HTTP error mapping

Not allowed:

* SQL queries
* Database access
* Business logic
* Repository implementation

## Interface Rules

* Define interfaces where they are consumed.
* Repository interfaces should live in Domain or Application.
* Repository implementations should live in Infrastructure.
* Usecase interfaces may live in Application when Delivery needs abstraction.
* Prefer small, behavior-based interfaces.

Good names:

* `OrderFinder`
* `OrderCreator`
* `OTPGenerator`
* `UserReader`

Avoid:

* `IOrderRepository`
* `OrderRepositoryInterface`
* `UsecaseGenerateOtp`
* `GenerateOtpUsecaseInterface`

## Dependency Injection

* Use constructor injection.
* Do not create concrete infrastructure dependencies inside usecases.
* Wire dependencies in the composition root, usually `cmd/api/main.go` or bootstrap package.

## Core Rules

* Handler calls Usecase, not Repository.
* Usecase calls interfaces, not concrete implementations.
* Repository handles persistence only.
* Business logic belongs in Domain or Application.
* SQL belongs in Infrastructure.
* HTTP status mapping belongs in Delivery.
* Do not put business logic in `pkg/`.
* Avoid generic folders like `utils`, `helpers`, or `common` unless the purpose is clear.
* Follow existing project conventions if they are already clean.

## Agent Behavior

* Inspect the existing project structure before adding files.
* Preserve Clean Architecture boundaries.
* Prefer small structural changes.
* Do not force large rewrites unless requested.
* Explain boundary violations when found.
* Keep generated code aligned with the chosen project structure.
