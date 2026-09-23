# Bulk Import and Performance Investigation

## Problem

A stakeholder Excel file with approximately 2,300 rows was used to validate the import feature.

The request was significantly heavier than a normal CRUD operation and exposed questions about memory consumption, database writes and HTTP request duration.

## Main Risks

A naive implementation can follow this pattern:

```text
Read entire file
   ↓
Convert every row to objects
   ↓
Keep everything in memory
   ↓
Insert everything
   ↓
Save
```

This may work for small files but becomes increasingly expensive as the dataset grows.

## Memory Considerations

Operations such as:

```csharp
ToList()
```

materialize an entire sequence in memory.

That is not automatically wrong, but it should be intentional.

For bulk imports, ask:

- How large can the uploaded file become?
- How many rows can it contain?
- How much memory does one mapped entity require?
- Are intermediate objects also retained?
- Can rows be processed incrementally?

## Database Strategy

There are several common strategies.

### One large operation

```text
Read all
 ↓
Map all
 ↓
AddRange
 ↓
SaveChanges
```

Simple and potentially efficient for moderate datasets.

### Batched operations

```text
Read batch
 ↓
Validate
 ↓
AddRange
 ↓
SaveChanges
 ↓
Clear tracking
 ↓
Next batch
```

This can reduce memory pressure and make large imports more manageable.

### Asynchronous background processing

For very large imports:

```text
Upload
 ↓
Create import job
 ↓
Return immediately
 ↓
Background worker processes file
 ↓
Store progress/result
```

This prevents a long-running HTTP request from being responsible for the entire operation.

## Transaction Strategy

A key business decision is whether the import should be atomic.

### Atomic

```text
All rows valid
    ↓
Commit everything

Any critical failure
    ↓
Rollback everything
```

### Partial success

```text
Valid rows
    ↓
Inserted

Invalid rows
    ↓
Returned in error report
```

Neither approach is universally correct.

The decision depends on the business meaning of the import.

## Timeout Investigation

When the browser reports a timeout, investigate the complete chain rather than immediately changing the timeout value.

```text
Browser
 ↓
HTTP request
 ↓
API
 ↓
Use case
 ↓
Excel parser
 ↓
EF Core
 ↓
PostgreSQL
```

Possible causes include:

- slow parsing;
- excessive memory usage;
- many individual database operations;
- inefficient queries;
- database locks;
- transaction duration;
- infrastructure timeouts.

Increasing the HTTP timeout can hide the symptom without fixing the underlying bottleneck.

## Lessons Learned

- Large inputs require explicit limits.
- `ToList()` should be used deliberately.
- Database write strategy matters as much as file parsing.
- Long-running HTTP requests deserve architectural consideration.
- Performance debugging should be evidence-based rather than based on guesswork.

## Engineering Takeaway

Bulk processing is not just CRUD multiplied by a large number.

The scale changes the constraints around memory, database traffic, transaction duration and user experience.
