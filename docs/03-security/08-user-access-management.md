# User and Access Management

## Context

A centralized administration area was expanded to manage application users and their access-related information.

The existing user interface included information such as:

- ID;
- name;
- username;
- email;
- role/profile;
- creation date;
- active/inactive status.

Typical operations included editing, deactivation, password changes, profile changes and deletion.

## Active / Inactive Lifecycle

A boolean such as:

```text
IsActive
```

can represent whether an account is currently active.

This supports an account lifecycle without necessarily deleting the user record.

Conceptually:

```text
Active
  ↓
Deactivate
  ↓
Inactive
```

This is often preferable when historical references or audit information must remain available.

## Access Matrix

The administration model can be extended to represent access to different systems.

Examples include:

```text
Hub
GLPI
Microsoft 365
OpenProject
Bitbucket
Azure / Active Directory
Physical access systems
```

A useful conceptual model is:

```text
User
 ├── Identity
 ├── Role
 ├── Account status
 └── System permissions
```

## Identity vs Authorization

A user record answers:

> Who is this person?

Authorization answers:

> What is this person allowed to do?

These are different concerns.

A user can exist in the system without having access to every module or external system.

## UI Design

A compact user table is useful for overview information.

Detailed access information can be displayed through an expanded panel, modal or secondary view.

Conceptually:

```text
User list
   ↓
Select user
   ↓
Detailed access view
   ├── Hub
   ├── GLPI
   ├── M365
   ├── OpenProject
   └── Other systems
```

This avoids overcrowding the primary user-management table.

## Lessons Learned

- Account lifecycle should be modeled explicitly.
- Identity and permissions should not be treated as the same concept.
- Access data can be presented as a matrix.
- UI hierarchy matters when a large amount of administrative information must remain readable.

## Engineering Takeaway

A centralized access-management interface becomes more useful when it combines **identity, lifecycle and authorization information** without confusing those concepts.
