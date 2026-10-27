# Persisted Reporting Snapshots and Cache Validation

## Problem

An in-process cache lost its useful history when the API restarted. A new report could then repeat years of upstream pagination, metadata retrieval and classification, even when most records had not changed.

## Design

A dedicated auxiliary read model stores external time entries, classification information and revision metadata. It does not replace the financial use case or persist its salaries, revenues, taxes and calculated totals. Financial repositories are still read when the report is calculated.

The read model is divided into a repository and a storage service. The repository performs database operations and version checks. The storage service creates a scope per operation, applies payload limits and retention, and allows a remote read if auxiliary storage is unavailable. That separation keeps a long-lived service from sharing a DbContext between concurrent calls.

Keys are scoped to a hash of source and credential identity. The credential itself is not written into the payload. Persisted content is validated before reuse: required collections, periods, identifiers, revision metadata and classification coverage must agree. Invalid content is rebuilt; it is not accepted as a successful partial report. An observed upstream no-store directive disables the corresponding reuse.

## Publication and freshness

An expected version prevents an older writer from silently replacing a newer snapshot. Before optional background processing, remote failures did not become fresh success from an old cache. The later background mode deliberately exposes the last completed publication with its original timestamp and a pending-update indication. Those are different freshness contracts.

## Evidence and limitations

The local rehearsal reused prepared data after process restart without upstream calls, and compared the requested-period output with an independently captured reference. First-process reads varied; persistence is not a promise of uniform sub-second API latency. The payload-length cap counts characters and is separate from the compressed page-cache byte budget. Snapshot payloads themselves are not published in this portfolio.

## Code example

```csharp
if (payload.Length > 32 * 1024 * 1024)
{
    return;
}
```
