# Stale Search Responses and Synchronization Status

## Problem

A user can launch a second search while the first is still in flight. If the first response arrives late, it can replace newer data or attach an old synchronization timestamp to the current result. A valid timestamp is misleading when it belongs to a different request.

## Request lifecycle

The controller keeps a generation number and an AbortController for the active search. Starting another search aborts the previous request and advances the generation. Success, error, synchronization metadata and busy-state cleanup must all confirm that they still belong to that generation before changing the model.

The NDJSON service waits for a complete successful result before invoking its metadata callback. Failed or incomplete streams do not publish a fresh synchronization status. The financial array remains the existing response value, while optional metadata identifies the source publication actually read by that request.

## Presentation

The screen formats only a valid date and displays it through a text binding. A stale publication uses a warning state and keeps its original timestamp. Reading a global status after the report finishes could race with a newer background publication; the server instead supplies the status captured by the reader used for the result.

Clearing or cancelling a search does not let its later callback replace the current status. The guard is necessary even when transport cancellation exists because an obsolete operation may already have produced a response.

## Evidence and limits

The local frontend cases checked completed-stream metadata, failed/incomplete streams, invalid metadata and callbacks from a cancelled search. This is code-level verification, not a claim of a fresh visual inspection of every browser state in production. Synchronization metadata was part of the local background change, which had not been deployed in that session. Existing NDJSON transport and the optional background feature have separate delivery states.

## Code example

```javascript
if (generation !== that._dreGeneration || abort.signal.aborted) return;
that._setDreSyncStatus(sync);
```
