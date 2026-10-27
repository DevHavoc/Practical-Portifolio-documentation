# Production Migration Rehearsal with PostgreSQL

## Problem

A clean branch merge cannot prove that a production database can reach the required schema. The API runs migrations before serving requests, so a database failure can prevent the new instance from starting. That is not a guarantee that earlier completed migrations caused no changes.

## Reconstructing the baseline

The rehearsal captured the 54 migration versions present in the release baseline, checked that those migration files had not changed in the target and ran that exact set against a disposable PostgreSQL 18 instance. The test server used a separate data directory, a random credential and localhost only. It did not read a production connection string.

Synthetic users, products, sprints, backlog items, integration logs, revenues and additional costs were added. The harness recorded the original columns of all 47 baseline tables and computed canonical row hashes before the upgrade. Adding a legitimate nullable column should not make that comparison mistake a schema addition for corruption of the old fields.

## Upgrade scenarios

The direct upgrade applied 14 pending migrations and finished with 68 recorded versions. Original-column fingerprints matched after the update, and all four active administrators in that synthetic scenario received the missing capability.

A second scenario began at an intermediate access schema with existing assignments and no status column. The old inactive flag migrated to an inactive status. The new administrator backfill then preserved existing grants and audit fields, adding only missing assignments. Repeating the runner left both assignments and version history stable.

## Evidence boundaries

The API Release build passed without warnings or errors. The application suite before the requested test cleanup reported 926 tests: 911 passed and the same 15 baseline failures by name. The three new migration regression cases passed; only the migration was kept in the final application change.

This was a schema/history reconstruction with synthetic data, not a copy of production records. Empty tables remaining empty contribute less evidence than seeded business scenarios. The rehearsal did not verify production credential permissions, unseen manual schema changes, every real-data constraint or a global rollback of all migrations. SQL Server compatibility code exists for the new backfill, but the full transition was tested only on PostgreSQL. The temporary server was stopped afterward.

## Code example

```csharp
var runner = scope.ServiceProvider.GetRequiredService<IMigrationRunner>();
if (target.HasValue) runner.MigrateUp(target.Value); else runner.MigrateUp();
```
