# Long-Running Reports, HTTP 499 and NDJSON Streaming

## Problem

A multi-year financial-report request ended with HTTP 499, while shorter searches could complete. The failure appeared after substantial upstream collection. Increasing a timeout alone would still leave the connection idle during expensive work and would not reduce the amount of data fetched.

## Investigation and decision

The transport and collection costs were treated separately. The existing JSON route was retained, and a companion NDJSON route emitted progress frames while the same report use case ran. The user subsequently supplied an HTTP 200 response for the streaming route; that status was evidence of an open transport, not by itself proof of a completed report.

The recorded implementation uses a fifteen-second heartbeat, a ten-minute report deadline and cancellation propagation. Admission limits retain one active query per actor and two per instance. A bounded filter length prevents an arbitrarily large selection string. These are the limits of the inspected implementation, not a general Azure configuration recommendation.

## Result contract

Progress, successful result and error are distinct frame types. The client waits for a complete result and rejects an interrupted stream. Because HTTP headers may already have been sent, a later report error can arrive inside a successful HTTP transport. The frontend must inspect the frame contract instead of treating every HTTP 200 as report success.

## Validation and limits

Cancellation, incomplete streams and metadata delivery were checked during the implementation work. A heartbeat can help with idle connections, but it does not prevent deliberate browser cancellation, establish a proxy-buffering policy or optimize the upstream queries. Collection optimization and prepared snapshots are separate case studies. The error observed here is not evidence that every possible HTTP 499 has the same cause.

## Code example

```csharp
var heartbeat = Task.Delay(TimeSpan.FromSeconds(15), tick.Token);
if (await Task.WhenAny(query, heartbeat) == query) { tick.Cancel(); break; }
token.ThrowIfCancellationRequested();
await Write(new { type = "progress" });
```
