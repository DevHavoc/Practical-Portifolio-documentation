# Updating Changed Records Without Reloading Every Year

## Problem

Fetching the full history for every search repeated a lot of work. Fetching only changed records was faster, but deleted records and records that no longer matched the filter could be missed.

## What changed

I kept the last checked data and replaced changed entries by their IDs. The reader then checks the source revision and record count. If they disagree with the combined data, it fetches the full period again. It also checks the names and classifications used to group the hours.

The update window overlaps the previous check to reduce the chance of missing changes near a date boundary. When background preparation adds another day, it can fetch that missing range without loading every earlier year again.

## Changes during a load

During testing, OpenProject changed while the first load was running. The reader rejected that result and checked again. The background path can retry a limited number of times using the data it already loaded. It only makes the result available after the revision checks pass. If the source keeps changing, the update fails instead of marking incomplete data as current.

## What was measured

Local history preparation took 270.39 seconds and transferred 78,400,935 bytes. A later update took 32.10 seconds and transferred 668,873 bytes. Those were different amounts of work, so they are not a comparison of the exact same request.

The returned entries matched the reference. The financial comparison used the same synthetic financial inputs. Large changes or failed checks can still require a full reload.

## Code example

```csharp
var changes = await ChangesAsync(start, end, ids, cached.CheckedAtUtc, ct);
var merged = cached.Months.Values.SelectMany(x => x).ToDictionary(x => x.Id!.Value);
foreach (var entry in changes)
    merged[entry.Id!.Value] = entry;
entries = merged.Values.OrderBy(x => x.Id).ToList();
// Exclusões e saídas do filtro não aparecem no delta.
// Uma contagem divergente exige substituir a cópia completa.
if (entries.Count != Count(version) || version != cached.Revision && changes.Count == 0)
    entries = await FullAsync(filters, start, end, ids, ct);
```
