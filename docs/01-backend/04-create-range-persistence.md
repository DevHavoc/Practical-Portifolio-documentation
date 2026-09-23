# Bulk Persistence with Entity Framework Core

## Problem

During bulk creation, entity objects were constructed successfully, but the expected database records were not being created.

The code reached `SaveChangesAsync()`, creating the impression that persistence had already been requested.

## The Missing Step

Creating an object in C# does not register it with Entity Framework Core.

This is insufficient:

```text
new Entity(...)
   ↓
SaveChangesAsync()
```

The DbContext needs to track the entity as `Added`.

The correct conceptual flow is:

```text
Create entities
   ↓
AddRange
   ↓
EF state = Added
   ↓
SaveChangesAsync
   ↓
INSERT statements
```

## Example

A simplified pattern is:

```csharp
var entities = rows
    .Select(MapToEntity)
    .ToList();

await context.Entities.AddRangeAsync(entities);
await context.SaveChangesAsync();
```

The exact implementation depends on the repository abstraction and application architecture, but the important concept remains the same.

## Why This Bug Is Easy to Miss

The code can look successful because:

- Excel parsing works;
- mapping works;
- the list contains the expected objects;
- no exception is thrown;
- `SaveChangesAsync()` executes.

Yet nothing is inserted if the entities were never tracked.

## Debugging Checklist

When a create operation appears to succeed without persistence:

1. Confirm the input rows exist.
2. Confirm mapping creates entities.
3. Inspect the collection count.
4. Confirm `Add` or `AddRange` is called.
5. Inspect EF entity states.
6. Confirm `SaveChangesAsync()` is reached.
7. Inspect generated SQL when necessary.
8. Query the database directly to verify the result.

## Lessons Learned

A successful method call is not evidence that the intended state transition happened.

The persistence pipeline must be verified from:

```text
Input
 ↓
Mapped entity
 ↓
Tracked entity
 ↓
SaveChanges
 ↓
SQL
 ↓
Database row
```

## Engineering Takeaway

Understanding EF Core's change tracker prevents a large class of persistence bugs and avoids unnecessary changes to controllers or database code.
