---
name: laravel-service-repository
description: Use this skill when structuring, refactoring, or creating CRUD/API features in Laravel backend projects using Controller, Service, Repository, Form Request, Resource, Model, and Eloquent layers.
---

# Laravel Service Repository Pattern

Use this skill to keep Laravel backend code structured, testable, and easy to maintain.

## Dependency Direction

```text
Controller → Service → Repository → Model/Eloquent
```

Rules:

* Controller must not contain business logic.
* Controller calls Service, not Repository directly.
* Service contains business logic and orchestration.
* Repository handles database queries.
* Model represents Eloquent entity and relationships.
* Form Request handles validation.
* Resource handles API response transformation.

## Layer Responsibilities

### Controller

Allowed:

* Receive HTTP request
* Call Form Request validation
* Call Service
* Return Resource or JSON response
* Map response status

Not allowed:

* Business logic
* Complex query logic
* Direct database access
* Long validation rules inline

### Service

Allowed:

* Business rules
* Usecase orchestration
* Transaction handling
* Calling repositories
* Calling external services
* Deciding application flow

Not allowed:

* HTTP request parsing
* Eloquent query chains for complex queries
* Response formatting
* Route-specific logic

### Repository

Allowed:

* Eloquent queries
* Query Builder logic
* Database persistence
* Data retrieval
* Filtering, sorting, pagination queries

Not allowed:

* HTTP response logic
* Business decisions
* Validation rules
* Controller-specific behavior

### Model

Allowed:

* Table mapping
* Fillable/guarded fields
* Casts
* Relationships
* Scopes for reusable simple queries

Not allowed:

* Large business workflows
* Controller logic
* API response formatting

### Form Request

Allowed:

* Validation rules
* Authorization rules
* Custom validation messages
* Preparing request data when needed

Not allowed:

* Business workflow
* Database persistence
* Response transformation

### Resource

Allowed:

* API response transformation
* Formatting output fields
* Conditional fields
* Relationship formatting

Not allowed:

* Business logic
* Database queries
* Validation