# Incremental Synchronization, Revisions and Deletions

## Problem

Re-reading a complete historical period after every search spent most of its time confirming data that had not changed. A naive updated-record query had another problem: deleted entries and records moved out of a filter do not appear as ordinary updates in the requested scope.

## Approach

The reader keeps a verified checkpoint and overlays changed entries by stable ID. Revision and count checks accompany the delta. A disagreement, or a revision change with no corresponding changes returned, can require a full reread. The design also revalidates names and classification dependencies; hours alone are insufficient when the grouping metadata changes.

An overlapping update window protects the boundary around timestamp granularity and time-zone interpretation. For the background checkpoint, extending the prepared period by a day retrieves the missing range rather than automatically reloading all earlier years. A first historical collection still has to do the original work.

## Concurrent changes

The live investigation observed the origin changing while the initial collection ran. The incomplete confidence state was rejected. The background path was then extended with bounded reconciliation using the already-loaded candidate as an in-memory checkpoint. Candidate data is not published before its revisions are confirmed. Persistent inconsistency still fails instead of converting partial data into a fresh result.

## Measured behavior

The later local incremental cycle took approximately 32.10 seconds and transferred 668,873 bytes, compared with 270.39 seconds and 78,400,935 bytes for historical preparation. These are different workloads, not a controlled claim that one identical query became 88 percent faster. The report-period data matched the reference and the finance comparison used identical synthetic financial inputs.

## Limits

Metadata checks and bounded retries are conservative consistency controls. They do not create a distributed transaction with the upstream application. Extensive changes, unavailable revision information or deletion inconsistencies may legitimately fall back to a full load or defer publication.

## Code example

```csharp
var merged = cached.Months.Values.SelectMany(x => x).ToDictionary(x => x.Id!.Value);
foreach (var entry in changes)
    merged[entry.Id!.Value] = entry;
entries = merged.Values.OrderBy(x => x.Id).ToList();
// Exclusões e saídas do filtro não aparecem no delta.
// Uma contagem divergente exige substituir a cópia completa.
if (entries.Count != Count(version) || version != cached.Revision && changes.Count == 0)
    entries = await FullAsync(filters, start, end, ids, ct);
```
