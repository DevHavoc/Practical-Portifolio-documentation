# Keeping Long Reports Connected with NDJSON

## Problem

A financial report could finish for one year and fail with HTTP 499 when the user selected several years. The browser waited while the server collected data from OpenProject.

## What changed

I added a streaming route alongside the existing JSON route. It sends a small progress message every fifteen seconds while the report runs. The browser then receives either the completed result or an error. The report has a ten-minute deadline, and cancelling the request also cancels its work.

The route limits active searches to one per user and two per server instance. It also limits the filter string length. These checks help keep a large search from using all the server's capacity.

## Why the browser checks the messages

The server may send HTTP 200 before it finishes calculating the report. A later error therefore has to travel in the stream. The frontend only accepts the report when it receives the complete result message; a progress message or an interrupted connection is not enough.

## What was checked

The implementation checks covered cancellation, errors and incomplete streams. The streaming route later returned HTTP 200 in the shared environment. That confirmed the route was responding, but the status alone did not prove that the report had finished.

Progress messages help keep the connection active. They do not make OpenProject return data faster or prevent every possible HTTP 499.

## Code example

```csharp
await Write(new { type = "progress" });
query = getDreConsolidado.HandleAsync(start, end, fiscalizadores, token);
while (!query.IsCompleted)
{
    using var tick = CancellationTokenSource.CreateLinkedTokenSource(token);
    var heartbeat = Task.Delay(TimeSpan.FromSeconds(15), tick.Token);
    if (await Task.WhenAny(query, heartbeat) == query) { tick.Cancel(); break; }
    token.ThrowIfCancellationRequested();
    await Write(new { type = "progress" });
}
var result = await query;
if (result.IsSuccess) await Write(new { type = "result", data = result.Data });
else await Write(new { type = "error", message = result.Error ?? "Não foi possível concluir o DRE." });
```
