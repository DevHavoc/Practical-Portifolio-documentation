# Giving Existing Admins the Missing User Overview Access

## Problem

New admins received the User Overview access automatically. Older admins did not. They could see the module but receive a forbidden response when opening the protected details.

## What changed

I added a migration that gives this access only to active admins who do not already have it. Existing access records stay unchanged, including inactive or suspended ones. Regular users and inactive admins receive no new access.

The SQL uses fixed conditions and the stored admin enum value. A conflict check prevents duplicates. Existing database defaults fill the audit dates, and the creator stays null because a migration is not a human administrator.

## Why rollback does not delete the access

The table cannot reliably tell whether this migration or an administrator created a matching access record. Deleting those records during rollback could remove a valid manual assignment. The migration therefore leaves them in place.

## What was checked

Three focused regression cases passed. A separate PostgreSQL test checked active admins, existing blocked access, audit fields and repeated startup. The final application change kept only this migration file, as requested. These checks used local test data and did not change production.

An inactive admin who is reactivated later still needs an access review. This migration fills the current gap; it does not automatically grant future access.

## Code example

```csharp
    public override void Up()
    {
        // Completa somente o vínculo ausente. Um acesso existente, inclusive bloqueado, mantém sua decisão e auditoria.
        IfDatabase("PostgreSQL").Execute.Sql($@"
INSERT INTO public.""UserAccesses"" (""UserId"", ""System"", ""IsActive"", ""Cargo"", ""Status"")
SELECT u.""Id"", 'Visão do Usuário', TRUE, 'Admin', 'Ativo'
FROM public.""UsersDB"" u
WHERE u.""Profile"" = {(int)UserProfile.Admin} AND u.""IsActive"" = TRUE
  AND NOT EXISTS (
      SELECT 1 FROM public.""UserAccesses"" a
      WHERE a.""UserId"" = u.""Id"" AND a.""System"" = 'Visão do Usuário'
  )
ON CONFLICT (""UserId"", ""System"") DO NOTHING;
");
        IfDatabase("SqlServer").Execute.Sql($@"
INSERT INTO [public].[UserAccesses] ([UserId], [System], [IsActive], [Cargo], [Status])
SELECT u.[Id], N'Visão do Usuário', 1, N'Admin', N'Ativo'
FROM [public].[UsersDB] u
WHERE u.[Profile] = {(int)UserProfile.Admin} AND u.[IsActive] = 1
  AND NOT EXISTS (
      SELECT 1 FROM [public].[UserAccesses] a WITH (UPDLOCK, HOLDLOCK)
      WHERE a.[UserId] = u.[Id] AND a.[System] = N'Visão do Usuário'
  );
");
    }
```
