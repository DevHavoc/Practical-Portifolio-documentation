# Entity Framework Core Tracking Investigation

## Context

A project update operation involved a project and several related entities, including manager, sponsor, products and stakeholders.

The main difficulty was understanding why apparently correct object changes did not always produce the expected database updates.

## The Core Concept

Entity Framework Core maintains an internal change tracker.

For tracked entities, EF Core records states such as:

```text
Unchanged
Added
Modified
Deleted
Detached
```

`SaveChangesAsync()` generates SQL based on those states.

## Tracked vs Detached

A common source of confusion is assuming that changing a C# object automatically means EF Core will update the database.

It depends on whether EF is tracking that object.

Conceptually:

```text
Tracked entity
   ↓
Property changed
   ↓
State becomes Modified
   ↓
SaveChanges
   ↓
UPDATE
```

But:

```text
Detached object
   ↓
Property changed
   ↓
EF knows nothing about it
   ↓
No update
```

## The `Update()` Trap

Calling:

```csharp
context.Projetos.Update(projeto);
```

can mark an entire object graph as modified depending on the graph and state information available.

This may appear to solve persistence problems, but it can also create unintended updates or tracking conflicts.

The better approach is to understand why the entity is detached or incorrectly tracked before applying `Update()` broadly.

## Duplicate Tracking

EF Core generally expects one tracked instance per entity key within a DbContext.

A problematic scenario is:

```text
DbContext
 ├── User Id = 1 (instance A)
 └── User Id = 1 (instance B)
```

EF Core cannot safely determine which instance represents the database row.

This can produce errors involving multiple instances with the same key.

## Better Investigation

When an update behaves unexpectedly:

1. Identify the DbContext instance.
2. Determine whether the entity is tracked.
3. Inspect the entity state.
4. Check whether another instance with the same key is already tracked.
5. Inspect navigation properties.
6. Check whether the graph contains detached objects.
7. Only then decide whether `Attach`, `Update`, explicit state assignment or a different loading strategy is appropriate.

## Why `Include()` Matters

When an operation intentionally works with related entities, loading the relevant graph can make state management easier to reason about.

However, `Include()` is not a universal fix.

The real question is:

> Which entities must be loaded and tracked for this operation?

Loading an unnecessarily large graph can create performance and state-management problems of its own.

## Lessons Learned

- EF Core persistence depends on entity state.
- Object graphs require deliberate tracking strategy.
- `Update()` is powerful but should not be used as a blind persistence button.
- Duplicate tracked instances usually indicate a lifecycle or graph-management problem.
- Debugging EF Core requires looking at state, not only at C# values.

## Engineering Takeaway

Many "EF didn't update my database" bugs are not SQL problems.

They are **state-management problems** that happen before SQL is generated.
