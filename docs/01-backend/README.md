# Backend Case Studies

[Cloud Case Studies](../../README.md)

## Documentation

- [Stakeholder Excel Import](01-stakeholder-excel-import.md) · [code example](01-stakeholder-excel-import.md#code-example)
- [Bulk Import and Performance Investigation](02-bulk-import-performance.md) · [code example](02-bulk-import-performance.md#code-example)
- [Entity Framework Core Tracking Investigation](03-ef-core-tracking.md) · [code example](03-ef-core-tracking.md#code-example)
- [Bulk Persistence with Entity Framework Core](04-create-range-persistence.md) · [code example](04-create-range-persistence.md#code-example)
- [SAPUI5 to ASP.NET Core Request Flow](05-sapui5-request-flow.md) · [code example](05-sapui5-request-flow.md#code-example)
- [Test Contract Alignment After Use Case Changes](24-jwt-constructor-azure-fix.md.md) · [code example](24-jwt-constructor-azure-fix.md.md#code-example)
- [Batch Access Registration and DbContext Concurrency](31-batch-access-registration-and-dbcontext-concurrency.md) · [code example](31-batch-access-registration-and-dbcontext-concurrency.md#code-example)
- [Preparing Report Data in the Background](39-background-report-sync-with-hosted-services.md) · [code example](39-background-report-sync-with-hosted-services.md#code-example)

## Code example

```csharp
public void ReduzirEstoque(int quantidade)
{
    if (quantidade <= 0)
        throw new ExceptionVenda("Quantidade inválida!");

    if (quantidade > Estoque)
        throw new ExceptionVenda("Estoque insuficiente!");

    Estoque -= quantidade;
}
```
