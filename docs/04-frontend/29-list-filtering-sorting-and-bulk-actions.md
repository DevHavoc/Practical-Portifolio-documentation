# List Filtering, Sorting and Bulk Action Consistency

## Problem

Two related administration screens had inconsistent interaction rules. Sorting was not intuitive, export actions did not consistently reflect row selection, and the toolbar arrangement changed during successive UI revisions.

An action could look available without a selected record, while another remained visually disabled after records had been selected. These were state-management problems as well as presentation problems.

## Context

The workflow covered Users and Access Assignments. Administrators needed search, status filters, additional classification filters, ascending or descending ordering, and actions on several selected rows.

The final toolbar separated import from export. Import retained a short label, while Add User occupied the rightmost action position. Export retained four explicit choices: Excel, PDF, selected Excel, and selected PDF.

## Investigation

The investigation followed the request from the SAPUI5 controls to API parameters and repository ordering. A selector is not functional merely because its displayed value changes: the selected field and direction must affect the database query before pagination.

Selection behavior was inspected separately. Enabled state, selected-ID extraction, and export handlers needed to agree on the same selected records. A visual color adjustment alone would not correct an action that still sent the wrong payload.

## Solution

The screens received explicit filter-and-order configuration. Users can be narrowed by status, profile, and position; assignments can be narrowed by status, system, and access profile. Ordering includes a field and a readable ascending/descending direction.

The query pipeline applies filtering and ordering before selecting a page. Changing filters or page size returns to the first page, avoiding a stale page index that points beyond the new result set.

Selection changes update both selected export actions and the bulk-action label. Selected exports are disabled when there are no selected rows. Handlers also reject an empty selection instead of treating it as an implicit request for all records.

Bulk actions use a dialog with a selection count, a required action selector, Apply, and Cancel. Available operations depend on the screen rather than assuming that user accounts and access assignments have identical lifecycle rules.

## Technical Flow

```text
Filter/order configuration
    -> controller state
    -> request parameters
    -> repository filtering and ordering
    -> pagination
    -> table and selection-dependent actions
```

## Why This Solution

Server-side ordering is necessary when a table is paginated. Sorting only the currently loaded page would create the appearance of ordering without producing a consistently ordered result set.

A single selection-derived state also prevents toolbar actions from disagreeing with their handlers.

## Validation

The historical implementation was inspected for filter parameter propagation, repository ordering, selected-ID collection, and menu enabled-state updates. The following scenarios remain a useful regression checklist:

- Select no records and confirm that both selected export actions are unavailable.
- Select several records and confirm that both actions become available.
- Compare ascending and descending results on more than one page.
- Apply a restrictive filter while on a later page and confirm the page index resets.
- Cancel a bulk-action dialog and confirm no write request is sent.
- Apply a destructive bulk action only after the explicit confirmation step.

This checklist describes expected checks; it is not a claim that every scenario was rerun during documentation preparation.

## Result

The administration screens use a more predictable relationship between filters, ordering, row selection, and available actions. Toolbar labels and placement were refined without changing their business purpose.

## Lessons Learned

Visual state and request state must have the same source of truth. A button that looks correct but sends the wrong scope is still a functional defect.

## Possible Improvements

Preserve configuration between visits, add explicit secondary ordering by a unique identifier for equal sort values, and define whether selection should persist across page changes.

Bulk actions on user accounts may still issue separate requests. Their UI should not be described as an atomic database transaction unless the backend actually provides that guarantee.
