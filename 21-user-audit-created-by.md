# Auditing Who Created a User

## Problem

The user-management interface needed to answer a simple administrative question: **who created this user?**

The original user record contained creation information, but not the identity of the operator responsible for creating the record. A timestamp alone was insufficient for administrative traceability.

## Context

The application already authenticated users and persisted user records. The solution was to associate the newly created user with another user representing the creator.

Conceptually:

```text
Creator User
     │
     └──── creates ────> Target User
```

## Investigation

The key point was that `CreatedBy` is not merely display text. It represents a relationship between two records of the same entity type.

Storing only a name would duplicate data and become inconsistent if the creator later changed their name.

## Root Cause

The user entity stored creation metadata but lacked a self-referencing foreign key identifying the operator responsible for the creation.

## Solution

A nullable self-reference was introduced conceptually as:

```text
CreatedByUserId
CreatedByUser
```

The API then exposes the creator's relevant display information in the response DTO.

The frontend can render:

```text
Created by
John Doe (jdoe)
```

without needing to perform a second request for every row.

## Technical Flow

```text
Authenticated operator
        ↓
Create user request
        ↓
Application layer identifies current user
        ↓
CreatedByUserId assigned
        ↓
Database persists relationship
        ↓
DTO exposes creator information
        ↓
UI displays creator
```

## Why This Solution

A foreign-key relationship preserves normalization and provides a stable reference to the creator.

The relationship is also useful beyond the current screen because the same metadata can later support reporting, auditing and administrative history.

## Result

User records can now display both creation time and creation responsibility without duplicating creator identity data.

## Lessons Learned

Audit fields should model relationships when the application needs to answer **who performed an action**, not merely **when it happened**.

## Possible Improvements

A future audit model could record a complete event history, including changes to user status, profile, password resets and access permissions.
