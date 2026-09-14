# User Access Overview UI

## Context

The Users screen already supported user editing and access management.

A separate read-only overview was introduced so administrators could inspect a user's identity, status and registered accesses without entering the editing workflow.

The main goal was to separate **inspection** from **modification**.

## Overview Structure

The overview was implemented as a wider SAPUI5 dialog containing:

- User identity: name, username and e-mail
- Current status
- Cargo, profile and creation date
- Registered system accesses
- Access status, source and last update

The structure can be summarized as:

    User Overview
        ↓
    Identity
        ↓
    User Information
        ↓
    Accesses

The dialog is intentionally read-only. Changes remain inside the existing edit flow.

## Loading Access Data

User information is already available when the dialog opens.

Access records are loaded separately through:

    GET /api/user-access/user/{userId}

The flow is:

    Open overview
        ↓
    Display user information
        ↓
    Load access records
        ↓
    Populate access list

A `BusyIndicator` is used while the access request is being processed.

## Layout Decisions

The dialog was made wider to accommodate the amount of information without compressing the access list.

Some layout issues appeared during implementation. For example, the access row initially used widths that exceeded 100%, causing content to be clipped.

The final layout uses a `62% / 38%` split between access information and update information.

The overview also avoids unnecessary draggable or resizable behavior, keeping it focused on quick inspection.

## Lessons Learned

- Inspection and editing can benefit from separate UI components.
- Visual hierarchy becomes important when displaying several types of administrative information.
- Dynamic access records are better represented as a list instead of additional fields on the user.
- Small layout calculations can have a significant impact on readability.

## Engineering Takeaway

A dedicated overview provides a simpler way to understand the current state of a user without mixing consultation with modification.

Separating these responsibilities keeps both workflows more predictable as the access-management feature grows.