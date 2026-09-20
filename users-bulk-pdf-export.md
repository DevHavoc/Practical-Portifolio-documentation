# Bulk User PDF Export

## Problem

After implementing individual user reports, the administration screen also needed to export multiple users. The export needed to support different scopes without duplicating the reporting implementation.

## Context

Three export scopes were exposed in the UI:

1. users currently displayed;
2. all users matching the current search/filter;
3. all users.

The backend uses the same report endpoint and decides which records to retrieve based on the request payload.

## Investigation

The important distinction was between **explicit IDs** and **query criteria**.

For displayed users, the frontend already knows the IDs currently loaded in the table. For search-based exports, sending every ID would be unnecessary and potentially incomplete because the table is paginated.

## Solution

The request supports both modes:

```text
userIds provided
    → export those users

userIds empty + filters provided
    → query matching users

userIds empty + no filters
    → query all users
```

## Technical Flow

```text
Export dialog
    ├── Displayed users
    │      ↓
    │   collect IDs
    │      ↓
    │   POST report
    │
    ├── Search results
    │      ↓
    │   send search + status
    │      ↓
    │   POST report
    │
    └── All users
           ↓
        empty criteria
           ↓
        POST report
                 ↓
          PDF generation
                 ↓
          browser download
```

## Why This Solution

Pagination should not limit an export intended to represent the entire search result. Sending search criteria to the backend allows the server to retrieve the complete matching dataset.

At the same time, explicit IDs are useful when the user intentionally wants only the currently displayed page.

## Result

The same reporting pipeline now supports page-level, filtered and complete exports without maintaining three independent PDF-generation implementations.

## Lessons Learned

Pagination is a UI concern; export scope is a business requirement. A paginated screen should not force a paginated export when the user selected the entire result set.

## Possible Improvements

For very large datasets, asynchronous report generation, background jobs or streaming could prevent long-running HTTP requests from becoming a bottleneck.
