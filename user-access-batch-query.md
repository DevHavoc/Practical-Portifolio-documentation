# User Access Queries and N+1 Investigation

## Problem

The user report needed access information for every exported user. A straightforward implementation loaded users first and then queried accesses inside a loop.

This was functional, but it introduced the classic N+1 query pattern.

## Context

The simplified flow was:

```text
Load N users
   ↓
For each user
   ↓
Load that user's accesses
```

For a small dataset this may appear harmless. As the number of users grows, the number of database round trips grows with it.

## Investigation

If 100 users are exported, the naïve approach can result in approximately:

```text
1 user query + 100 access queries
```

The problem is not necessarily the amount of returned data. It is the number of independent database round trips.

## Root Cause

Accesses were retrieved individually through a repository method such as:

```text
GetByUserIdAsync(userId)
```

inside the report-building loop.

## Solution Direction

A batch-oriented repository operation can retrieve accesses for all requested users at once.

Conceptually:

```text
SELECT accesses
FROM UserAccess
WHERE UserId IN (...)
```

The application can then group the returned records by `UserId` and map them to the corresponding report DTO.

## Improved Flow

```text
Load users
    ↓
Collect user IDs
    ↓
Load all accesses in one query
    ↓
Group accesses by UserId
    ↓
Build reports
    ↓
Generate PDF
```

## Why This Matters

The batch approach reduces database round trips and makes performance more predictable as the exported user count increases.

The biggest improvement often comes not from micro-optimizing C# code, but from changing the shape of the database interaction.

## Result

The investigation identified a scalability issue without changing the external behavior of the report feature.

## Lessons Learned

A loop containing an asynchronous repository call deserves attention whenever the loop can grow with user input or dataset size.

The important question is:

> Can the data required by the entire operation be retrieved in one set-oriented query?

## Possible Improvements

Additional optimization opportunities include projection directly to report DTOs, database-side filtering, pagination for interactive screens and profiling SQL execution plans.
