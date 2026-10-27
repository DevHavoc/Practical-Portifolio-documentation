# Distributed Synchronization Leases and Atomic Publication

## Problem

Sequential execution inside one hosted worker does not prevent another cloud instance from starting the same synchronization. A worker that pauses, loses ownership and later resumes must also be prevented from publishing over a newer executor.

## Lease and ownership

The local implementation uses the shared auxiliary database for a source-scoped lease. Acquisition creates a row or atomically takes an expired row with a new owner token. The source namespace includes the configured credential identity. A ten-minute synchronization budget is paired with a twelve-minute lease, providing a recovery margin if a process disappears.

The owner and lease freshness are checked inside the publication transaction. The lease row remains locked while the complete prepared payload is saved. A stale executor cannot publish after another owner has taken over, and release removes only the lease belonging to the releasing owner. Checking an ownership flag only at the start of a long job would not provide that protection.

## Failure behavior

Publication is a complete replacement, not a stream of partially updated financial inputs. Transient failure preserves the last successful publication and marks it pending without renewing the success timestamp. Shutdown and release have bounded cancellation handling. A crashed process is recovered through expiry rather than an indefinitely held in-memory lock.

## Validation and scope

Tests exercised source isolation, a busy lease, expiry takeover, rejected publication by an old owner and an old owner attempting to release a new owner's lease. The feature remained a local implementation during this session; the earlier snapshot repository provided version-based writes but not this complete lease workflow.

The lease prevents overlap. Independent timers on different instances can still perform sequential checks, so it is not a globally exact five-minute scheduler. The lease test used an isolated relational database; the complete workflow was not exercised on the production database. Database availability and command execution still bound the operation.

## Code example

```csharp
await using var transaction = await context.Database.BeginTransactionAsync(ct);
// Mantém a linha da concessão bloqueada até a publicação: um executor antigo não publica após perder a posse.
var owned = await context.DreReadModelSnapshots
    .Where(x => x.SourceKey == source && x.SnapshotKey == leaseKey
        && x.Version == owner && x.StoredAtUtc > expiredBeforeUtc)
    .ExecuteUpdateAsync(set => set.SetProperty(x => x.StoredAtUtc, x => x.StoredAtUtc), ct);
if (owned != 1) return false;

var saved = await GetAsync(source, key, ct);
```
