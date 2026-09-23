# Dependency Injection Failure in ASP.NET Core

## Problem

The application returned an error similar to:

```text
Unable to resolve service for type '...'
while attempting to activate '...Controller'
```

The frontend operation failed while attempting to load user access information.

## What the Error Means

ASP.NET Core uses Dependency Injection to construct controllers.

Conceptually:

```text
HTTP request
 ↓
Controller activation
 ↓
Resolve constructor dependencies
 ↓
Create controller
 ↓
Execute action
```

If a constructor dependency cannot be resolved, the action is never executed.

Therefore, placing a breakpoint inside the action may show nothing.

## Investigation

Start with the controller constructor.

Example:

```csharp
public UserAccessController(
    IUserAccessService accessService)
{
    ...
}
```

Then verify:

1. Is the interface correct?
2. Does an implementation exist?
3. Is it registered in the service container?
4. Is the correct project/assembly referenced?
5. Is the dependency's own dependency graph valid?
6. Is its lifetime appropriate?

## Registration

A typical registration has the conceptual form:

```csharp
services.AddScoped<IUserAccessService, UserAccessService>();
```

The exact lifetime depends on the service.

Common lifetimes are:

```text
Transient
Scoped
Singleton
```

For services that depend on a request-scoped DbContext, `Scoped` is commonly appropriate.

## Dependency Chain

A registered service can still fail if one of its dependencies is missing.

For example:

```text
Controller
 ↓
AccessService
 ↓
Repository
 ↓
DbContext
```

The container must be able to resolve the entire chain.

## Debugging Strategy

When the error says "Unable to resolve service":

```text
Read exact missing type
 ↓
Inspect controller constructor
 ↓
Inspect DI registration
 ↓
Inspect implementation constructor
 ↓
Trace dependency chain
 ↓
Restart / rebuild
 ↓
Retry endpoint
```

## Lessons Learned

- Controller activation happens before action execution.
- DI errors can be diagnosed from the dependency graph.
- The exact missing type is usually the strongest clue.
- Debugging the frontend first can be misleading when the backend cannot instantiate the controller.

## Engineering Takeaway

Dependency Injection is not invisible magic.

It is a dependency graph, and errors become much easier to solve when that graph is followed explicitly.
