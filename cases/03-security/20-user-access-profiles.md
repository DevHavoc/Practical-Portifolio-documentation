# User Access Management and Profile Modeling

## Problem

A user-management module needed to represent more than application identity. A user could be active or inactive, have an application profile and job role, and also have access to several external systems. The initial model was centered on the user record itself, which made access management difficult to evolve and mixed identity data with authorization data.

The goal was to build a clearer model in which:

- application identity remains on the user entity;
- system-specific access is represented independently;
- each access can be active or inactive;
- access records can carry their own profile/context;
- the UI can display and manage those records without duplicating user data;
- the database enforces one access record per user/system combination.

## Context

The implementation follows a layered architecture:

```text
SAPUI5 UI
   ↓
Controller / Request Model
   ↓
REST API
   ↓
Application Layer
   ↓
Repositories
   ↓
Entity Framework Core
   ↓
PostgreSQL
```

The existing user entity contained identity-oriented fields such as name, username, e-mail, application profile, role and active state. Instead of adding an ever-growing list of boolean columns such as `HasSystemA`, `HasSystemB`, and `HasSystemC`, access information was extracted into a dedicated relationship.

## Investigation

The first important distinction was between **who the user is** and **what systems the user can access**.

A user record answers questions such as:

- What is the user's name?
- Which username identifies the user?
- Which application profile does the user have?
- Is the account active?

An access record answers different questions:

- Which system is involved?
- Is that access active?
- Which system-specific profile was assigned?
- When was the access created or updated?
- Who created the access?

Keeping these concepts separate prevents the user table from becoming a collection of unrelated authorization flags.

## Root Cause

The core modeling problem was treating access as a property of the user instead of as a first-class domain concept.

A boolean-per-system design would have created several problems:

1. Every new system would require a schema change.
2. System-specific metadata would have nowhere natural to live.
3. Audit information would become difficult to represent.
4. Queries would become increasingly coupled to a fixed system catalog.
5. The UI would need special handling for every new access type.

## Solution

A dedicated `UserAccess` entity was introduced with the following conceptual fields:

```text
UserAccess
├── Id
├── UserId
├── System
├── IsActive
├── Cargo / System Profile
├── Source
├── CreatedAt
├── UpdatedAt
└── CreatedByUserId
```

The relationship becomes:

```text
User 1 ──────────── * UserAccess
```

The user keeps identity and application-level properties, while `UserAccess` represents access to a specific system.

### Database constraint

A unique constraint was added for:

```text
(UserId, System)
```

This is important because the same user should not accidentally receive two independent records representing the same system access.

### Foreign key behavior

The access record references the user through `UserId`. The relationship is configured so that deleting a user also removes dependent access records, preventing orphaned access entries.

## Why This Solution

This model scales with the number of systems instead of the number of systems being hard-coded into the user table.

For example, adding another system becomes a data-level operation:

```text
User 42
 ├── System A → Active
 ├── System B → Inactive
 ├── System C → Active
 └── System D → Active
```

The user entity itself does not need another property for System D.

This also makes the UI generic: the access list can render records rather than having a dedicated section for every system.

## Technical Decisions

### 1. Access as a separate entity

Access information was modeled independently because it has its own lifecycle.

A user can remain active while one particular system access is inactive, and an access can be changed without modifying the user's identity data.

### 2. Per-access status

`IsActive` belongs to `UserAccess`, not only to `User`.

This allows states such as:

```text
User: Active
System A: Active
System B: Inactive
System C: Active
```

### 3. System identifier as data

The system name is stored with the access record rather than represented as a large enum containing every possible external system.

This reduces coupling between the database model and the list of integrations.

### 4. Audit metadata

Access records can retain information about who created them and when they were created or updated. This is particularly useful for administrative operations.

### 5. Repository boundary

The application layer does not directly manipulate the database context. Access operations are exposed through an `IUserAccessRepository` abstraction.

This keeps persistence details inside the infrastructure layer.

## Technical Flow

### Loading accesses

```text
User Overview
     ↓
GET /user-access/by-user/{id}
     ↓
UserAccessRepository
     ↓
Database query
     ↓
Access DTOs
     ↓
SAPUI5 access list
```

### Creating access

```text
Select system + status + profile
              ↓
        Add access
              ↓
      POST /user-access
              ↓
     Application validation
              ↓
       Repository insert
              ↓
       Database constraint
              ↓
        Refresh access list
```

### Updating status

```text
Activate / Deactivate
          ↓
PUT /user-access/{id}
          ↓
Repository update
          ↓
Database
          ↓
Refresh UI state
```

### Deleting access

```text
Delete
  ↓
Confirmation dialog
  ↓
DELETE /user-access/{id}
  ↓
Repository
  ↓
Database
  ↓
Remove item from UI
```

## UI Design

The administration dialog separates general user information from access management.

```text
User Dialog
├── Identity
│   ├── Name
│   ├── Username
│   ├── E-mail
│   ├── Application Profile
│   └── Job Role
│
└── Access Management
    ├── Existing accesses
    ├── System selector
    ├── Status selector
    ├── System profile
    └── Add access
```

The access list exposes the state of each system independently. This is easier for administrators to reason about than a large collection of unrelated checkboxes.

## Authorization Consideration

The UI also checks whether the current operator has permission to open the user overview. This is important because hiding a button is not equivalent to enforcing authorization.

The frontend check improves the user experience, while the API remains responsible for enforcing the actual permission boundary.

Conceptually:

```text
UI permission check
        ↓
Can open overview?
        ↓
API authorization
        ↓
Data access
```

## Result

The resulting model provides a reusable access-management foundation:

- users and accesses have separate responsibilities;
- each system access has its own status;
- duplicate user/system records are prevented by the database;
- access data can be queried independently;
- the UI can add, update and remove accesses dynamically;
- audit metadata can be associated with access operations;
- new systems do not require new columns on the user table.

## Lessons Learned

### Model the lifecycle, not only the data

If a piece of information can be independently created, updated, disabled, audited or deleted, it may deserve its own entity.

### Database constraints are part of business integrity

Application validation helps, but uniqueness should also be enforced by the database. Otherwise concurrent requests can still create duplicates.

### Avoid boolean explosion

A design such as:

```text
HasSystemA
HasSystemB
HasSystemC
HasSystemD
...
```

works initially but becomes increasingly expensive as the system grows.

A relationship table is usually a better fit when the set of related resources can evolve.

### UI state should mirror domain state

The access list became simpler once each access record carried its own status instead of trying to derive everything from the parent user.

## Possible Improvements

Future iterations could introduce:

- a controlled catalog of systems instead of free-form system identifiers;
- stronger system-specific profile validation;
- bulk access provisioning;
- access history instead of only the latest state;
- filtering and searching by system;
- explicit permission policies per operation;
- automated tests around duplicate access creation and authorization.

## Engineering Takeaway

The important architectural improvement was not simply adding another table. It was recognizing that **user identity and system access are different concepts with different lifecycles**.

Once that boundary was established, database design, API contracts and UI behavior became easier to reason about and extend.
