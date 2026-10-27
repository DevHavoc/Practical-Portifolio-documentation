# Saving Report Data for Reuse After a Restart

## Problem

The API kept useful report data in memory. After a restart, it could fetch years of OpenProject data again, even when most of it had not changed.

## What changed

I added database storage for a snapshot: a saved copy of time entries, their classifications and the information needed to check for changes. It does not save the final financial totals. The report still reads revenues, salaries, taxes and other financial inputs when it runs.

The repository handles database reads and writes. A separate storage service opens a fresh dependency scope for each operation, limits the saved data size and removes old snapshots. If this extra storage is unavailable, the reader can fetch from OpenProject.

## Keeping the saved data safe to reuse

Each snapshot belongs to a specific source and credential identity. The key uses a hash; it does not contain the credential itself. Before reuse, the reader checks dates, IDs, required collections, classifications and source revision information. Invalid data is rebuilt. A no-store response from the source prevents reuse.

Writes also check the expected snapshot version, so an older writer cannot silently replace a newer copy. In background mode, a failed update keeps the previous complete copy and its original date, with an update-pending status.

## What was measured

A local test reused the saved data after restarting the process without calling OpenProject. The returned data matched a separately captured reference. Startup reads varied, so this does not promise a report in under a second on Azure.

## Code example

```csharp
public async Task SaveAsync(string source, string key, string payload, Guid? expectedVersion, CancellationToken ct)
{
    if (payload.Length > 32 * 1024 * 1024)
    {
        return;
    }
    await ExecuteAsync(async r =>
    {
        var saved = await r.TrySaveAsync(source, Hash(key), payload, expectedVersion, ct);
        if (saved)
        {
            await r.PruneAsync(source, DateTime.UtcNow.AddDays(-Math.Clamp(options.Value.RetentionDays, 1, 365)), ct);
        }

        return saved;
    }, false, ct);
}
```
