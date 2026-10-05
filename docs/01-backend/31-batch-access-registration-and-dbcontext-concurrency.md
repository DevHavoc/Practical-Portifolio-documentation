# Batch Access Registration and DbContext Concurrency

## Problem

Creating access assignments for multiple users and multiple systems raised an Entity Framework Core exception about a second operation starting before the previous operation had completed.

After the concurrency error was addressed, another investigation was needed because a batch could still produce no useful registration outcome.

## Context

The intended operation is a Cartesian expansion: each selected user receives an assignment for each selected system, using the requested access profile and status, except where that user/system assignment already exists.

For example, two users and two systems represent four candidate assignments, not one record containing an array of users.

The following payload is illustrative and uses generalized names rather than reproducing the application's proprietary contract:

```json
{
  "userIds": [101, 202],
  "systems": ["System A", "System B"],
  "profile": "Reader",
  "status": "Active"
}
```

## Investigation

Catalog validation started multiple repository calls with `Task.WhenAll`. Those calls shared the same scoped DbContext. Asynchronous execution did not make that context safe for overlapping operations.

The registration path was also examined for duplicate checks, persistence results, and user feedback. A successful HTTP transport should not be confused with evidence that assignments were created.

## Solution

Profile and system catalog validation now awaits each repository operation sequentially on the shared context. The profile is validated first, followed by the selected systems, with an early return for invalid values.

The batch flow also:

- removes invalid and repeated user IDs;
- trims and deduplicates system labels case-insensitively;
- validates that all selected users exist;
- validates catalog membership and length constraints;
- retrieves existing assignments for all selected users together;
- builds a case-insensitive in-memory user/system lookup;
- creates only missing combinations;
- persists the collection through a batch repository operation and transaction;
- reports the actual created count in the UI;
- returns a conflict when every selected combination already exists;
- treats a zero-created persistence result as an error rather than displaying success.

## How an Assignment Belongs to a User

The JSON request carries identifiers, but it is not the persistent relationship. Each stored assignment has a user identifier and a corresponding ORM relationship to the user entity.

```text
User
    -> Assignment: System A, Reader, Active
    -> Assignment: System B, Reader, Active
```

The database relationship is represented by the assignment's user reference. System and access-profile names are validated against a catalog in the inspected implementation. The legacy request field used for the access profile should not be confused with the user's organizational job position.

## Why This Solution

Sequential catalog checks respect the ownership of the scoped DbContext. In contrast, independent HTTP requests to an external API can be bounded and parallelized because they do not share that database context.

One existing-assignment query also replaces a database round trip inside every user/system iteration.

## Validation

The historical registration commit confirms the removal of overlapping catalog queries, the existing-assignment lookup, the zero-created guard, and the displayed creation count. Repository inspection confirms collection insertion and transaction completion.

Useful regression scenarios include four new combinations, a partially existing batch, an entirely existing batch, repeated input values, an unknown user, an invalid catalog value, and a persistence failure.

These scenarios are documented as a checklist; no additional live access assignments were created while preparing this portfolio.

## Result

The batch operation no longer starts concurrent queries on the shared context, and its outcome distinguishes actual creation from duplicates or unsuccessful persistence.

## Lessons Learned

Optimize round trips before adding concurrency. A bulk workflow benefits more from querying existing records once than from launching unsafe parallel queries.

## Possible Improvements

Application-level duplicate checks alone do not guarantee uniqueness under simultaneous batch submissions. A database uniqueness constraint, idempotency support, bounded batch sizes, and structured per-combination outcomes would strengthen the contract further.

Catalog identifiers instead of mutable labels would also make renaming and referential integrity more robust. These are recommendations, not implemented guarantees.
