# Testing the Database Upgrade with PostgreSQL

## Problem

A merge without conflicts does not prove that the database upgrade will work. The API runs migrations before serving requests. A failure can stop the new instance from starting, even if earlier migrations already changed the database.

## How I tested it

I rebuilt the release baseline using its 54 migrations on a temporary PostgreSQL 18 instance. It used a separate data directory, localhost and a random test password. No production connection or records were used.

I added synthetic users, products, sprints, backlog items, logs, revenues and costs. Before upgrading, I recorded the original columns and calculated a hash of their data across 47 tables. Comparing those same columns afterward allowed new columns without treating them as damaged old data.

## Upgrade results

The direct upgrade applied 14 more migrations and ended with 68 recorded versions. The original-column data hashes matched. All four active test admins received the missing User Overview access.

A second test started from an older access schema. Inactive access became the inactive status, existing grants and audit fields stayed unchanged, and only missing grants were added. Running the migrations again did not add duplicates or change the version history.

## What this proves

The Release build passed without warnings or errors. Before the requested test cleanup, 911 of 926 application tests passed; the other 15 matched the existing baseline failures. The three new migration cases passed, and only the migration remained in the final application change.

This tested reconstructed schemas and synthetic records. It did not check production permissions, manual database changes or every real record. The full upgrade was tested on PostgreSQL, not SQL Server. The temporary database server was stopped afterward.

## Code example

```csharp
void Migrate(string database, Type[] types, long? target = null)
{
    var connection = new NpgsqlConnectionStringBuilder(builder.ConnectionString) { Database = database };
    var services = new ServiceCollection().AddFluentMigratorCore()
        .ConfigureRunner(r => r.AddPostgres().WithGlobalConnectionString(connection.ConnectionString));
    services.AddSingleton<IMigrationSourceItem>(new TypesSource(types));
    using var provider = services.BuildServiceProvider();
    using var scope = provider.CreateScope();
    var runner = scope.ServiceProvider.GetRequiredService<IMigrationRunner>();
    if (target.HasValue) runner.MigrateUp(target.Value); else runner.MigrateUp();
}
```
