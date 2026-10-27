# Security Case Studies

[Back to the portfolio](../../README.md)

## Documentation

- [User and Access Management](08-user-access-management.md) · [code example](08-user-access-management.md#code-example)
- [Resource Authorization and IDOR Analysis](12-resource-authorization-and-idor.md) · [code example](12-resource-authorization-and-idor.md#code-example)
- [User Access Management and Profile Modeling](20-user-access-profiles.md) · [code example](20-user-access-profiles.md#code-example)
- [User Overview Permission Gating](32-user-overview-permission-gating.md) · [code example](32-user-overview-permission-gating.md#code-example)
- [Backfilling Native Access for Existing Administrators](42-backfilling-native-access-for-existing-admins.md) · [code example](42-backfilling-native-access-for-existing-admins.md#code-example)

## Code example

```csharp
var canViewOverview = await getUserAccessCase.HasActiveAccessAsync(
    jwtClaimsExtractor.GetRequiredUserId(),
    "Visão do Usuário");
if (!canViewOverview.Data)
    return Forbid();
```
