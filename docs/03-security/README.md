# Security Case Studies

[Cloud Case Studies](../../README.md)

## Documentation

- [User and Access Management](08-user-access-management.md) · [code example](08-user-access-management.md#code-example)
- [Resource Authorization and IDOR Analysis](12-resource-authorization-and-idor.md) · [code example](12-resource-authorization-and-idor.md#code-example)
- [User Access Management and Profile Modeling](20-user-access-profiles.md) · [code example](20-user-access-profiles.md#code-example)
- [User Overview Permission Gating](32-user-overview-permission-gating.md) · [code example](32-user-overview-permission-gating.md#code-example)
- [Giving Existing Admins the Missing User Overview Access](42-backfilling-native-access-for-existing-admins.md) · [code example](42-backfilling-native-access-for-existing-admins.md#code-example)

## Code example

```csharp
[HttpGet("user-overview/{userId:int}")]
[Authorize(Policy = "Admin")]
[HasPermission("Acessos.Usuarios.Visualizar")]
public async Task<IActionResult> GetUserOverview(int userId)
{
    var canViewOverview = await getUserAccessCase.HasActiveAccessAsync(
        jwtClaimsExtractor.GetRequiredUserId(),
        "Visão do Usuário");
    if (!canViewOverview.Data)
        return Forbid();

    return (await getUserByIdCase.HandleAsync(userId)).ToActionResult();
}
```
