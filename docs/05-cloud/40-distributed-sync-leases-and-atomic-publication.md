# Preventing Two Servers from Saving the Same Update

## Problem

One background worker runs its jobs in order. But two Azure instances could still start the same update. An older worker could also resume after a pause and try to overwrite newer data.

## What changed

I added a shared database lease: a temporary right to run the update. It has an owner ID and an expiry time. Another instance can take over only after it expires. The source and credential identity keep unrelated sources separate.

The update has a ten-minute deadline and the lease lasts twelve minutes. This gives the server some time to stop work and release ownership. If the process disappears, another instance can recover after the lease expires.

## Saving the result

Before saving, the repository checks that the worker still owns a valid lease. It holds the lease row inside the same database transaction while replacing the prepared data. If ownership has changed, the old worker cannot save. Releasing a lease also requires the same owner ID.

A failed update keeps the last complete data and its original date. The screen shows that an update is pending.

## What was checked

Local tests covered an occupied lease, expiry and takeover, separate sources, and an old owner trying to save or release another owner's lease. The full workflow was not tested against the production database, and this feature had not been integrated or deployed.

The lease prevents overlapping work. It does not make the timers on different servers run at exactly the same time.

## Code example

```csharp
public async Task<bool> TryPublishAsync(string source, string leaseKey, Guid owner,
    DateTime expiredBeforeUtc, string key, string payload, CancellationToken ct)
{
    await using var transaction = await context.Database.BeginTransactionAsync(ct);
    // Mantém a linha da concessão bloqueada até a publicação: um executor antigo não publica após perder a posse.
    var owned = await context.DreReadModelSnapshots
        .Where(x => x.SourceKey == source && x.SnapshotKey == leaseKey
            && x.Version == owner && x.StoredAtUtc > expiredBeforeUtc)
        .ExecuteUpdateAsync(set => set.SetProperty(x => x.StoredAtUtc, x => x.StoredAtUtc), ct);
    if (owned != 1) return false;

    var saved = await GetAsync(source, key, ct);
    if (!await TrySaveAsync(source, key, payload, saved?.Version, ct)) return false;
    await transaction.CommitAsync(ct);
    var tracked = context.DreReadModelSnapshots.Local.FirstOrDefault(x => x.SourceKey == source && x.SnapshotKey == key);
    if (tracked is not null) context.Entry(tracked).State = EntityState.Detached;
    return true;
}
```
