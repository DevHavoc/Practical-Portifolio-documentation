# Financial Report Query Optimization with OpenProject

## Problem

A consolidated financial report filtered by oversight groups and a multi-year period could take more than 30 minutes according to user observation. Other attempts ended in HttpClient timeouts or an HTTP 504 gateway timeout.

Increasing waiting time did not remove the expensive remote work. Successful HTTP 200 responses on individual requests also did not prove that the complete report would finish in time.

## Context

The calculation combines external time entries, work-package classification, local revenue, additional costs, and personnel cost information. Work packages without their own classification require descendant lookup.

The final change retained the existing synchronous endpoint and screen interaction. It optimized data collection rather than replacing financial formulas or introducing a new report UI.

## Investigation

The remote path was measured separately from the presentation layer. The large interval contained 14,042 time entries, spanning 29 pages at 500 records per page and referencing 6,979 distinct work packages.

The important costs were sequential page retrieval, metadata retrieval, and classification fallback. Loading the entire remote work-package catalog was unnecessary: only work packages referenced by the requested time entries and relevant descendants were needed.

The collection payload did not expose a usable children list. Therefore, absence of such a property could not safely prove that a work package was a leaf. A global parent filter was verified against actual IDs instead.

## Solution

### Time Entry Pagination

The first page determines the declared total and effective page size. When it is a full page, remaining pages are fetched with a limit of three concurrent requests. Results are assembled in page order, with stable ID sorting in the remote query.

If the first page is partial or the page size is not established, the original sequential termination path is retained. Parallel results are checked for an incomplete count or repeated IDs instead of silently accepting inconsistent data.

### Targeted Work Package Retrieval

Distinct referenced IDs are retrieved in batches of up to 100 with bounded concurrency. All statuses are included, preserving closed work packages in historical calculations.

The work package's own classification is used first. A partial list of time-entry-related work packages is not mistaken for a complete descendant tree.

### Descendant Resolution

Parents requiring fallback are grouped into batches across projects. Descendants are fetched a level at a time and resolved from their parent links in memory. Visited identifiers and the existing depth guard prevent pathological traversal.

Failures in the new batch path propagate rather than silently converting unavailable remote data into an unclassified financial result. Existing classification cache integration is retained, but the performance measurement did not rely on cache hits.

### Transport and Diagnostics

GZip/Deflate decompression is enabled for the named OpenProject client. Responses are disposed after use. A transient timeout repeats the affected read-only page once after a short delay; it does not restart the report. Cancellation requested by the caller is not retried.

The 30-second per-request timeout remains. Stage logs record counts and elapsed time without publishing authentication headers, internal addresses, or financial records.

## Technical Flow

```text
Period + selected oversight groups
    -> retrieve period time entries with bounded pagination
    -> extract distinct work-package IDs
    -> retrieve required metadata in batches
    -> resolve missing classification through batched descendants
    -> apply selected classification scope
    -> aggregate hours and existing cost/revenue formulas
    -> return the existing report contract
```

The classification filter is not claimed to run inside the upstream time-entry query. Classification is resolved before the local report scope is applied.

## Approaches Not Retained

| Approach | Observation | Final decision |
|---|---|---|
| Full global work-package catalog | Large unrelated retrieval and request timeout risk | Retrieve only referenced IDs and necessary descendants |
| Per-item descendant requests | Too many round trips | Group parents and resolve loaded relationships locally |
| Longer timeouts alone | Did not address total query work or gateway limits | Keep request timeout and reduce remote work |
| Experimental jobs/polling flow | The chat records pending/processing states alongside UI errors | Not part of the final implementation |
| Four concurrent collection requests | A live measurement exceeded the 30-second request timeout | Return to the measured three-request limit |

The polling symptoms are not proof of one specific backend root cause. They are documented as an unsuccessful earlier path, not as a delivered asynchronous architecture.

## Validation

The final read-only measurement used the interval 2023-01-01 through 2026-10-01, without classification cache hits:

| Stage | Records | Cumulative elapsed time |
|---|---:|---:|
| Time entries | 14,042 | 102.5 seconds |
| Referenced work packages | 6,979 | 172.2 seconds |
| Classification fallback | 1,991 roots | 179.1 seconds |

The complete remote collection finished in approximately 2 minutes 59 seconds. An earlier successful intermediate configuration took approximately 3 minutes 55 seconds. The four-request experiment failed and is not included as a success result.

The 47 focused regression tests passed. They cover calculations, batch resolution, classification filtering in the use case, page ordering, concurrency bounds, cancellation, failed pages, and bounded timeout retry.

These are recorded measurements from the implementation session, not new benchmarks run while writing this document. User confirmation later reported that the change worked. Neither that confirmation nor the local timing is a universal production latency guarantee.

## Result

The final collection path completed the large interval in minutes rather than the previously reported tens of minutes, while retaining the existing report interface and financial calculation rules.

The initial more-than-30-minute duration was user-reported, not captured as a controlled baseline. An exact percentage speedup or identical-environment comparison would therefore overstate the evidence.

## Lessons Learned

Concurrency must match the upstream service's capacity. Four requests were not automatically better than three. Database-context concurrency and independent HTTP concurrency also require different safety decisions.

Avoiding unnecessary remote work, batching identifiers, and measuring the full dependency chain matters more than increasing one timeout value.

## Possible Improvements

Consider a deliberately designed local reporting snapshot, freshness policy, single-flight request deduplication, and production monitoring. Durable asynchronous jobs remain an option if future report sizes exceed gateway limits, but would require separately validated lifecycle and frontend contracts.
