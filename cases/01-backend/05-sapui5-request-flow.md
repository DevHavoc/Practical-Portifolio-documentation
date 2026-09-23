# SAPUI5 to ASP.NET Core Request Flow

## Context

The application uses SAPUI5 on the frontend and ASP.NET Core on the backend.

When a feature fails, the most efficient approach is to trace the request from the UI event to the database rather than debugging each layer in isolation.

## Complete Request Lifecycle

```text
SAPUI5 event
    ↓
Controller handler
    ↓
RequestModel / BaseController
    ↓
HTTP request
    ↓
ASP.NET Core routing
    ↓
Controller action
    ↓
Use Case
    ↓
Repository
    ↓
Entity Framework Core
    ↓
PostgreSQL
```

## Frontend Layer

A user action can trigger a controller method such as:

```text
Button click
 ↓
handleAction()
 ↓
request model
```

The frontend should be inspected first to verify:

- the event is actually fired;
- the correct handler runs;
- the expected parameters exist;
- the correct URL is generated;
- the correct HTTP method is used;
- the payload or `FormData` is correct.

## Browser Network Tab

The browser Network tab is one of the strongest debugging tools for this architecture.

Inspect:

- request URL;
- HTTP method;
- status code;
- request payload;
- headers;
- response body;
- request duration.

This immediately tells you whether the problem reached the backend.

## Backend Routing

Once the request reaches ASP.NET Core, verify:

```text
Route
 ↓
Controller
 ↓
Action
```

If the controller action is never reached, debugging the use case is premature.

## Application Layer

The controller should generally translate HTTP concerns into application operations.

The use case is responsible for business/application behavior.

For example:

```text
Controller
    ↓
Validate request boundary
    ↓
Call use case
    ↓
Translate result to HTTP response
```

## Persistence Layer

The repository and EF Core layer handle database interaction.

When data is missing, trace:

```text
Use Case input
 ↓
Repository method
 ↓
Entity state
 ↓
Generated SQL
 ↓
PostgreSQL
```

## First-Failing-Layer Method

The most useful debugging question is:

> What is the first layer where the actual behavior differs from the expected behavior?

For example:

```text
UI works
 ↓
HTTP request correct
 ↓
Controller reached
 ↓
Use case reached
 ↓
Repository not called
```

The bug is probably between the use case and repository.

This is much more efficient than searching the entire codebase randomly.

## Lessons Learned

- The frontend Network tab provides strong evidence.
- Each layer has a distinct responsibility.
- Debugging should follow the real execution path.
- The first failing layer is usually the most valuable place to investigate.

## Engineering Takeaway

The architecture itself becomes a debugging map.

Once the request lifecycle is understood, many apparently complex full-stack bugs become localized problems.
