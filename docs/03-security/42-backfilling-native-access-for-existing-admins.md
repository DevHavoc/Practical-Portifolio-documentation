# Backfilling Native Access for Existing Administrators

## Problem

New administrator registration automatically created an active user-overview capability. Administrators who existed before that behavior had no equivalent backfill. The module could be visible to their role while the guarded detail endpoint correctly rejected them for lacking the separate active capability.

## Forward-only data correction

A new migration selects users whose stored profile is administrator, whose account is active and who do not already have the capability. It inserts only those missing grants. Existing active, inactive or suspended assignments keep their status, assignment profile, timestamps and creator. Ordinary users and inactive administrators receive no new assignment in this pass.

The migration uses fixed SQL and the enum's stored administrator value, not a request-supplied role or user ID. Database defaults supply the existing UTC audit timestamps, and the creator remains null because a system migration is not a human administrator. A conflict guard prevents duplicate insertion.

## Rollback decision

The new migration does not rewrite previous migrations or edit their recorded versions. Its Down method preserves grants: the table does not retain a reliable marker that would distinguish a migration-created grant from a later human assignment. Automatically deleting matching rows during rollback could revoke legitimate access. This describes the new data migration only, not the behavior of rolling the whole database below the table-creation version.

## Evidence

Three regression cases passed before the user requested that the final application change retain only the migration. A PostgreSQL rehearsal also verified active-administrator selection, preservation of blocked assignments and repeated startup. The final application change contains one migration file. Production data was not modified by this validation.

An inactive administrator who is later reactivated still needs an explicit access review; this migration does not silently create future grants or bypass the endpoint's capability check.

## Code example

```sql
INSERT INTO public."UserAccesses" ("UserId", "System", "IsActive", "Cargo", "Status")
SELECT u."Id", 'Visão do Usuário', TRUE, 'Admin', 'Ativo'
FROM public."UsersDB" u
WHERE u."Profile" = 0 AND u."IsActive" = TRUE
  AND NOT EXISTS (
      SELECT 1 FROM public."UserAccesses" a
      WHERE a."UserId" = u."Id" AND a."System" = 'Visão do Usuário'
  )
ON CONFLICT ("UserId", "System") DO NOTHING;
```
