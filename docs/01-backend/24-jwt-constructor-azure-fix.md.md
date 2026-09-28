# Test Contract Alignment After Use Case Changes

## Problem

A user activation/deactivation use case was extended with a new `IUserAccessRepository` dependency and additional asynchronous operations.

The existing tests still followed the previous contract, causing `CS7036` during the Azure deployment build.

## Context

The implementation introduced a new dependency and additional repository operations:

```text
ToggleUserActive
    ├── User Repository
    ├── JWT dependency
    └── User Access Repository
            ↓
    DeactivateAllByUserIdAsync()
```

## Investigation

The tests were not updated to reflect the new implementation.

Three inconsistencies were identified:

```text
Missing constructor dependency
        ↓
Missing repository verification
        ↓
Missing CancellationToken propagation
```

## Solution

The test fixture was updated with the required mocks, including:

```csharp
private Mock<IJwtOptions> _mockJwtOptions = null!;
private Mock<IUserAccessRepository> _mockUserAccessRepository = null!;
```

The new repository interaction was explicitly verified:

```csharp
_mockUserAccessRepository.Verify(
    x => x.DeactivateAllByUserIdAsync(userId),
    Times.Once
);
```

`CancellationToken` was also propagated to downstream asynchronous operations.

## Success and Failure

```text
Valid operation
    ↓
Update user state
    ↓
Update related accesses
    ↓
Success
```

```text
Required operation fails
    ↓
Failure result
    ↓
No false success
```

## Technical Flow

```text
Use Case
    ↓
Repository operations
    ↓
User access operations
    ↓
CancellationToken
    ↓
Test verification
```

## Result

The tests were aligned with the updated use case contract, and the Azure build could compile the updated test setup successfully.

## Lessons Learned

When a use case changes, its tests must evolve with its dependencies, side effects and asynchronous contracts.

## Possible Improvements

Additional tests could verify that downstream operations are not executed when an earlier required operation fails.

## Engineering Takeaway

A `CS7036` detected during deployment exposed a broader test-contract mismatch rather than an Azure-specific problem.

Keeping tests aligned with application contracts helps CI/CD detect these inconsistencies before deployment.
