# Handling POST Requests That Return Binary Data

## Problem

The frontend already had a request abstraction designed primarily around JSON responses. PDF export introduced a different response type: binary data returned from a POST request.

A normal JSON request flow was not enough because the browser needed the raw PDF bytes.

## Context

The frontend uses a request-model abstraction to centralize HTTP communication. Instead of making direct `fetch` calls throughout controllers, the controller creates a request model and invokes the appropriate operation.

## Investigation

The important distinction was between:

```text
JSON response
```

and:

```text
Binary response / Blob
```

Trying to parse a PDF as JSON would corrupt the response or cause parsing errors.

## Root Cause

The request abstraction did not originally expose a dedicated operation for a POST request whose response should be treated as a browser `Blob`.

## Solution

A blob-aware POST operation was used for report generation.

Conceptually:

```text
Request Model
    ↓
POST
    ↓
HTTP response
    ↓
response.blob()
    ↓
Blob
```

The controller then creates a temporary object URL:

```text
Blob
 ↓
URL.createObjectURL()
 ↓
Temporary <a>
 ↓
click()
 ↓
Download
 ↓
URL.revokeObjectURL()
```

## Why This Solution

Binary handling belongs in the request abstraction because multiple features may eventually need downloads. Keeping blob handling centralized avoids repeating low-level HTTP logic in every controller.

## Technical Flow

```text
SAPUI5 Controller
      ↓
createRequestModel(reportEndpoint)
      ↓
setData(reportCriteria)
      ↓
postBlob()
      ↓
Blob
      ↓
Object URL
      ↓
Browser download
```

## Result

The frontend can request generated PDFs using the same request-model architecture used by the rest of the application while correctly preserving the binary response.

## Lessons Learned

HTTP abstractions should represent the actual response semantics, not assume every endpoint returns JSON.

A mature request layer should make common response types explicit, for example:

```text
JSON
Blob
Text
File / stream
```

## Possible Improvements

A future request abstraction could expose consistent methods such as `get`, `post`, `getBlob`, `postBlob`, and potentially centralized handling for content disposition, filenames and download progress.

## Engineering Takeaway

The PDF feature was not only a reporting implementation. It exposed a reusable frontend infrastructure requirement: **HTTP clients need first-class support for binary responses.**
