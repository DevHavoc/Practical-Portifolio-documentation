# Resource Authorization and IDOR Analysis

## Context

A project-based API contained endpoints where a project identifier appeared in the route and resource-specific operations were performed beneath that project.

This led to an important security analysis.

## Route-Level Authorization

An authorization mechanism such as:

```text
ProjectAccess(projectId)
```

can establish that the current user has access to the project represented by the route.

However, that does not automatically prove that every child resource identifier supplied to the endpoint belongs to that project.

## Example

Consider:

```text
/api/projects/10/stakeholders/55
```

There are two identifiers:

```text
Project = 10
Stakeholder = 55
```

It is necessary to establish:

```text
Stakeholder 55 belongs to Project 10
```

before modifying or deleting it.

## Object-Level Validation

A robust operation can conceptually perform:

```text
Authenticate user
      ↓
Authorize project 10
      ↓
Load stakeholder 55
      ↓
Verify stakeholder.ProjectId == 10
      ↓
Perform operation
```

Without the ownership relationship check, an attacker who can legitimately access one project might attempt to manipulate a resource belonging to another project by changing the child ID.

This is the general class of issue commonly associated with insecure direct object references / broken object-level authorization.

## Important Distinction

Finding a potential authorization weakness during feature development does not necessarily mean the new feature introduced the weakness.

A responsible engineering report should distinguish:

```text
New vulnerability introduced by change
```

from:

```text
Pre-existing risk discovered while modifying the area
```

This distinction matters for both remediation and incident analysis.

## Defensive Pattern

For resource-specific operations, the query should usually enforce the relationship directly.

Conceptually:

```text
WHERE Resource.Id = requestedId
AND Resource.ProjectId = routeProjectId
```

This is stronger than loading by ID alone and checking the relationship later.

## Lessons Learned

- Route authorization and object authorization are different.
- Nested resources should validate their parent-child relationship.
- Security reviews can emerge naturally from ordinary feature work.
- Engineers should clearly distinguish newly introduced problems from pre-existing risks.

## Engineering Takeaway

Authorization should be evaluated at the **same level of specificity as the resource being modified**.
