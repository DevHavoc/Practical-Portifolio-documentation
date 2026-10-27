# Preparing Report Data in the Background

## Problem

Even after reducing repeated requests, the first search over several years could take minutes. The user still had to wait for OpenProject to return the history.

## What changed

I added an optional .NET background worker. It prepares data after the API starts and checks for updates on a timer, every five minutes by default. It runs on the server without a browser session or a user clicking Sync. Each run gets its own dependency scope and finishes before the worker starts another run.

The feature is off by default. It prepares a configured date range through the current UTC day. Searches inside that range can use the saved data. Before preparation finishes, or for dates outside the range, the report uses the existing source-reading path.

## When an update fails

A completed run replaces the saved report data as a whole. An update failure keeps the last complete copy, its original date and an update-pending status. A detected 401, 403 or no-store response makes that prepared copy unavailable. Changes to source access are detected on the next source check.

## Timing and delivery

Five minutes is a check interval, not a promise that new data will be ready every five minutes. A long run must finish first. The application also needs to stay loaded; on Azure App Service, the hosting setup must support that.

This work was tested locally and had not been integrated or deployed during this session. Local preparation took about 4 minutes 30 seconds. A later update took 32 seconds. Reads after a process restart ranged from 0.57 to 10.37 seconds; these were source-reader measurements, not full-page Azure timings.

## Code example

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    // A primeira carga não bloqueia a inicialização da API.
    await Task.Yield();
    if (!options.Value.Enabled || !options.Value.BackgroundSyncEnabled) return;
    var interval = TimeSpan.FromMinutes(Math.Clamp(options.Value.SyncIntervalMinutes, 1, 1440));
    using var timer = new PeriodicTimer(interval);
    try
    {
        do
        {
            if (stoppingToken.IsCancellationRequested) break;
            try
            {
                await using var scope = scopes.CreateAsyncScope();
                await scope.ServiceProvider.GetRequiredService<DreBackgroundSynchronizer>().SynchronizeAsync(stoppingToken);
            }
            catch (OperationCanceledException) when (stoppingToken.IsCancellationRequested) { break; }
            catch (Exception ex)
            {
                logger.LogWarning(ex, "DRE: a sincronização automática não foi concluída. O horário da última publicação não foi alterado.");
            }
        }
        while (await timer.WaitForNextTickAsync(stoppingToken));
    }
    catch (OperationCanceledException) when (stoppingToken.IsCancellationRequested) { }
}
```
