# Data Case Studies

[Cloud Case Studies](../../README.md)

## Documentation

- [Database Schema Evolution with FluentMigrator](06-fluentmigrator-schema.md) · [code example](06-fluentmigrator-schema.md#code-example)
- [Result Pattern in .NET](07-result-pattern-dotnet.md) · [code example](07-result-pattern-dotnet.md#code-example)
- [Auditing Who Created a User](21-user-audit-created-by.md) · [code example](21-user-audit-created-by.md#code-example)
- [Generating a PDF Report for a User](22-user-report-pdf.md) · [code example](22-user-report-pdf.md#code-example)
- [Bulk User PDF Export](23-users-bulk-pdf-export.md) · [code example](23-users-bulk-pdf-export.md#code-example)
- [Selection-Aware Exports and Report Boundaries](30-selection-aware-exports-and-report-boundaries.md) · [code example](30-selection-aware-exports-and-report-boundaries.md#code-example)
- [Financial Report Query Optimization with OpenProject](34-dre-openproject-query-optimization.md) · [code example](34-dre-openproject-query-optimization.md#code-example)
- [Saving Report Data for Reuse After a Restart](37-persisted-reporting-snapshots-and-cache-validation.md) · [code example](37-persisted-reporting-snapshots-and-cache-validation.md#code-example)
- [Updating Changed Records Without Reloading Every Year](38-incremental-sync-revisions-and-deletions.md) · [code example](38-incremental-sync-revisions-and-deletions.md#code-example)
- [Testing the Database Upgrade with PostgreSQL](43-production-migration-rehearsal-with-postgresql.md) · [code example](43-production-migration-rehearsal-with-postgresql.md#code-example)

## Code example

```csharp
updateCmd.Parameters.AddWithValue(
    "@estoque",
    produtoEscolhido.Estoque
);

updateCmd.Parameters.AddWithValue(
    "@id",
    produtoEscolhido.Id
);

updateCmd.ExecuteNonQuery();
```
