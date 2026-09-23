# REST API Architecture: Controller to Repository

## Architecture

The backend follows a layered flow similar to:

```text
HTTP request
 ↓
Controller
 ↓
Use Case
 ↓
Repository
 ↓
Entity Framework Core
 ↓
PostgreSQL
```

Each layer has a different responsibility.

## Controller

The controller is the HTTP boundary.

Typical responsibilities:

- route handling;
- request binding;
- HTTP-specific validation;
- authorization attributes;
- invoking application logic;
- translating application results into HTTP responses.

The controller should not become the place where all business logic lives.

## Use Case

The use case represents an application operation.

For example:

```text
Import stakeholders
Update project
Load user accesses
Change account status
```

It coordinates business/application rules.

## Repository

The repository abstracts persistence operations.

Conceptually:

```text
Use Case
 ↓
Repository
 ↓
EF Core
```

This separates application behavior from database-specific implementation details.

## Entity Framework Core

EF Core handles:

- object-relational mapping;
- entity tracking;
- SQL generation;
- persistence;
- querying.

Understanding EF Core is therefore essential when debugging repository behavior.

## PostgreSQL

The database is the final persistence layer.

When a feature behaves unexpectedly, the database should sometimes be inspected directly instead of assuming that application state equals database state.

## Why Layering Helps Debugging

A layered architecture provides natural checkpoints:

```text
Did the controller receive the request?
        ↓
Did the use case execute?
        ↓
Did the repository execute?
        ↓
Did EF track the expected entities?
        ↓
Was SQL generated?
        ↓
Did PostgreSQL persist the change?
```

The architecture becomes a diagnostic tool.

## Lessons Learned

- Separation of concerns makes failures easier to localize.
- Controllers should remain focused on HTTP concerns.
- Use cases centralize application behavior.
- Repositories isolate persistence.
- EF Core behavior must be understood to debug database operations effectively.

## Engineering Takeaway

Architecture is not only about code organization.

A well-understood architecture also provides a **map for debugging production problems**.
