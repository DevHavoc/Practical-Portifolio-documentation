# Project Update and Persistence Behavior

## Context

A project update operation modified a project together with related information such as manager, sponsor, products and stakeholders.

The challenge was understanding why changes appeared correct in memory but did not always persist as expected.

## Update Lifecycle

A simplified update flow is:

```text
HTTP request
 ↓
DTO
 ↓
Use Case
 ↓
Load existing project
 ↓
Load relevant relationships
 ↓
Apply changes
 ↓
EF Core tracks state
 ↓
SaveChangesAsync
 ↓
Database
```

## Existing Entity vs Replacement Object

There is an important difference between:

```text
Load existing tracked entity
```

and:

```text
Construct a new detached object
```

When the existing entity is tracked, changing its properties can be enough:

```text
Tracked entity
 ↓
Change property
 ↓
State = Modified
 ↓
SaveChanges
```

When a detached graph is passed into the DbContext, explicit state management may be required.

## Why Broad `Update()` Can Be Dangerous

Using:

```csharp
context.Projetos.Update(projeto);
```

can mark the graph as modified.

That may cause more database changes than intended and can interact badly with:

- navigation properties;
- duplicate tracked entities;
- detached relationships;
- existing records.

Therefore, the correct solution is not automatically "add Update()".

The correct question is:

> What entity state does EF Core currently know about?

## Debugging Entity State

Useful questions include:

```text
Is the project tracked?
Are related entities tracked?
Are there duplicate instances?
Which entities are Added?
Which are Modified?
Which are Deleted?
Which are Detached?
```

EF Core's change tracker can provide direct evidence.

## Related Collections

Updating collections such as:

```text
Products
Stakeholders
```

requires thinking about whether the operation means:

- append;
- replace;
- remove;
- synchronize;
- upsert.

These are different business operations and should not be treated as identical.

## Lessons Learned

- Persistence behavior depends on entity state.
- Updating a graph is more complex than updating a single row.
- Broad state changes can hide the real lifecycle of an entity.
- Related collections require explicit business semantics.

## Engineering Takeaway

A reliable update implementation starts by defining **what changed**, then choosing the EF Core state-management strategy that represents that change accurately.
