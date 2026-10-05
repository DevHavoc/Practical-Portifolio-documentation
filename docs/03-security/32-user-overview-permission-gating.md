# User Overview Permission Gating

## Problem

The user-overview popup could still appear after the actor's corresponding access was deactivated or suspended. The UI interaction did not reliably reflect the effective permission state.

## Context

User overview contains identity information and access assignments. Its authorization must be evaluated for the authenticated actor requesting the view, not inferred from the user whose details are being displayed.

The application already had role-based and module-level authorization, plus an active-access requirement for this particular capability.

## Investigation

The investigation distinguished three questions:

1. Is the actor authenticated?
2. Does the actor satisfy role and module permission requirements?
3. Does the actor currently have active access to the overview capability?

A role alone could not answer the third question. A frontend flag could also be stale after a permission change.

## Solution

Dedicated overview routes were introduced for the target user's information and assignments. They retain the existing administrative and module permission checks and additionally consult the authenticated actor's active-access state.

An actor without active overview access receives a forbidden response before the target user's data is returned. Frontend overview requests use those guarded routes rather than the broader data paths.

Suspension and inactivity both fail the active-access check, while remaining distinct business states in access administration.

## Technical Flow

```text
Actor requests target user's overview
    -> authentication and role/module checks
    -> actor's current active-access check
    -> deny before returning target data, or retrieve authorized data
    -> frontend handles success or forbidden response
```

## Why This Solution

UI restrictions improve interaction, but server-side checks enforce the boundary. Directly requesting the endpoint must not bypass a disabled button or a hidden popup trigger.

The capability check belongs to the actor. The target user identifier only identifies the resource being inspected.

## Validation

The historical permission-fix commit confirms additional backend guards on both overview data routes and the frontend route changes.

Recommended regression checks cover an active actor, an inactive actor, a suspended actor, an unauthorized direct request, and a permission change followed by a new request in the same browser session.

Historical implementation is distinguished from current deployment state: the documentation checkout is on another branch and does not contain this entire user-access module.

## Result

Overview data retrieval is tied to the actor's effective permission state rather than only to the presence of a UI control or an administrative role.

## Lessons Learned

Authorization must protect the data path, not only the visual entry point. Multiple requests supporting the same popup need equivalent protection.

## Possible Improvements

Centralize the capability rule to reduce duplicated controller checks, add dedicated negative authorization tests, and define how an already-open overview reacts to a later revocation.

The inspected fix protects subsequent requests. It is not a claim of real-time removal of data already loaded in the browser or automatic invalidation of every other report route.
