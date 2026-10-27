# Background Report Synchronization with Hosted Services

## Problem

Request-time optimization reduced repeated work, but a first historical search could still spend minutes reading the external source. Preparing the read model independently of an interactive session moved that cost away from the user's search after preparation had completed.

## Implementation

An optional hosted worker starts its first pass after API startup and awaits each execution. A PeriodicTimer supplies a configurable interval, five minutes by default. Each execution creates its own asynchronous service scope, resolves the synchronizer and passes the server shutdown token. It is not driven by a logged-in browser or a user-facing synchronization endpoint.

The feature is disabled by default. The prepared range begins at a configurable date and extends through the current UTC day. Queries inside a valid publication can filter the prepared entries locally. A request outside coverage or before the first publication retains the previous remote path.

The historical checkpoint and published read model are separate. A completed pass publishes the full result; a failed pass retains the previous completed data and its original timestamp, with an update-pending indication. A detected no-store directive or 401/403 response invalidates the publication. Source authorization is learned on the next upstream check rather than in real time for each prepared read.

## Scheduling boundaries

Execution is sequential in one worker. If a pass takes longer than the interval, pending ticks coalesce; the system does not promise an updated publication exactly every five minutes. In-process processing also depends on the host staying loaded. An App Service flag alone does not keep an unloaded application running; the hosting configuration must support that requirement.

## Status and evidence

This feature was implemented and validated locally during the session and remained outside the integrated application branches at that point. It is not described as deployed. Historical preparation took about 4 minutes 30 seconds locally; a later incremental pass took 32 seconds. Prepared-source reads were fast in an already-running process, but new-process measurements ranged from 0.57 to 10.37 seconds. Full-page Azure latency and real financial-repository time were not measured by that harness.

## Code example

```csharp
await using var scope = scopes.CreateAsyncScope();
await scope.ServiceProvider.GetRequiredService<DreBackgroundSynchronizer>().SynchronizeAsync(stoppingToken);
```
