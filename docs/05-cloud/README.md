# Cloud Case Studies

[Cloud Case Studies](../../README.md)

## Documentation

- [Dependency Injection Failure in ASP.NET Core](09-dependency-injection-troubleshooting.md) · [code example](09-dependency-injection-troubleshooting.md#code-example)
- [NuGet Source and Azure Deployment Troubleshooting](10-Azure-nuget-deployment-troubleshooting.md) · [code example](10-Azure-nuget-deployment-troubleshooting.md#code-example)
- [REST API Architecture: Controller to Repository](11-rest-api-architecture.md) · [code example](11-rest-api-architecture.md#code-example)
- [Production Debugging Methodology](13-production-debugging-methodology.md) · [code example](13-production-debugging-methodology.md#code-example)
- [Project Update and Persistence Behavior](14-project-update-persistence.md) · [code example](14-project-update-persistence.md#code-example)
- [Frontend Payload Debugging](16-frontend-payload-debugging.md) · [code example](16-frontend-payload-debugging.md#code-example)
- [SAPUI5 Row Interaction](17-sapui5-row-interaction.md) · [code example](17-sapui5-row-interaction.md#code-example)
- [Regression Testing and Evidence-Driven Delivery](35-regression-testing-and-evidence-driven-delivery.md) · [code example](35-regression-testing-and-evidence-driven-delivery.md#code-example)
- [Keeping Long Reports Connected with NDJSON](36-long-running-report-streaming-and-client-cancellation.md) · [code example](36-long-running-report-streaming-and-client-cancellation.md#code-example)
- [Preventing Two Servers from Saving the Same Update](40-distributed-sync-leases-and-atomic-publication.md) · [code example](40-distributed-sync-leases-and-atomic-publication.md#code-example)
- [Checking What a Merge and Deployment Will Include](44-merge-readiness-and-deployment-boundaries.md) · [code example](44-merge-readiness-and-deployment-boundaries.md#code-example)

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
