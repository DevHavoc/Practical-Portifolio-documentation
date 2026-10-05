# Selection-Aware Exports and Report Boundaries

## Problem

Selected-user PDF export sometimes contained only one user despite multiple selected rows. The menu also exposed inconsistent availability and PDF iconography.

Another requirement concerned report scope: a PDF exported from Users should describe users, not silently include their access assignments.

## Context

The final export menu exposes four actions:

| Action | Record scope | Availability |
|---|---|---|
| Export Excel | All users | Independent of selection |
| Export PDF | All users | Independent of selection |
| Export selected Excel | Explicit selected user IDs | Requires selection |
| Export selected PDF | Explicit selected user IDs | Requires selection |

The final all-user PDF handler explicitly requests the complete scope. It should not be described as exporting the current filtered result merely because the table has active filters.

Earlier portfolio documentation describes additional page-level and search-based report scopes. Those capabilities and the final four-action menu are related, but they are not identical UI contracts.

## Investigation

The selected-PDF path was treated as an end-to-end scope problem: table selection, ID extraction, request serialization, backend retrieval, and report composition.

The historical controller collects distinct IDs from the selected binding contexts and sends the entire array. It downloads the response as a binary blob through the authenticated request abstraction rather than expecting a JSON report body.

The distinction between a user report and an access report was reviewed as a data-minimization boundary, not just a choice of columns.

## Solution

Selected Excel and PDF actions use the same selection-derived ID collection. Empty selections are rejected locally. The PDF action uses a PDF attachment icon, and both selected actions share selection-dependent enabled behavior.

The toolbar keeps Import separate, with the short label Import. Existing spreadsheet import remains an account-creation workflow; this iteration reorganized its entry point rather than introducing a new importer from scratch.

Users-screen exports remain user-oriented. A detailed individual overview is a separate context where that user's assignments can intentionally be included. Passwords and unrelated users' assignment data are not appropriate output for a general users report.

## Technical Flow

```text
Selected table contexts
    -> distinct selected IDs
    -> authenticated report request with the full ID array
    -> backend retrieves requested users
    -> user-oriented document generation
    -> PDF blob download
```

## Why This Solution

An explicit ID array separates intentional selection from the currently visible page and from a broad export request. Binary download handling is kept consistent with the application's request infrastructure.

Separating report contexts prevents a convenient reporting implementation from expanding the amount of information exported without the administrator's intent.

## Validation

Historical source inspection confirms that selected export sends an ID array and that the final all-user PDF clears search and status criteria. The chat also records the original single-user symptom and the subsequent requested correction.

A complete report regression check should compare the selected ID set with the generated document, including two or more users, an empty selection, and user-only content. Documentation preparation did not regenerate reports against live confidential records.

## Result

Selected-user export has an explicit multiple-record payload and a consistent binary-download path. The UI distinguishes all-user export from selected-user export, and report content follows the screen's intended scope.

## Lessons Learned

The count of selected checkboxes is not enough evidence that an export is correct. The same scope must survive serialization, query construction, and document generation.

## Possible Improvements

Add report-content assertions using synthetic users, explicit filtered-result export if required, and asynchronous generation for genuinely large outputs. These are future options, not features introduced in this iteration.
