# Project Update and Persistence Behavior

## Context

A project update operation modified a project together with related information such as manager, sponsor, products and stakeholders.

The challenge was understanding why changes appeared correct in memory but did not always persist as expected.

## Update Lifecycle

A simplified update flow is:

```text
HTTP request
 ↓
DTO
 ↓
Use Case
 ↓
Load existing project
 ↓
Load relevant relationships
 ↓
Apply changes
 ↓
EF Core tracks state
 ↓
SaveChangesAsync
 ↓
Database
```

## Existing Entity vs Replacement Object

There is an important difference between:

```text
Load existing tracked entity
```

and:

```text
Construct a new detached object
```

When the existing entity is tracked, changing its properties can be enough:

```text
Tracked entity
 ↓
Change property
 ↓
State = Modified
 ↓
SaveChanges
```

When a detached graph is passed into the DbContext, explicit state management may be required.

## Why Broad `Update()` Can Be Dangerous

Using:

```csharp
context.Projetos.Update(projeto);
```

can mark the graph as modified.

That may cause more database changes than intended and can interact badly with:

- navigation properties;
- duplicate tracked entities;
- detached relationships;
- existing records.

Therefore, the correct solution is not automatically "add Update()".

The correct question is:

> What entity state does EF Core currently know about?

## Debugging Entity State

Useful questions include:

```text
Is the project tracked?
Are related entities tracked?
Are there duplicate instances?
Which entities are Added?
Which are Modified?
Which are Deleted?
Which are Detached?
```

EF Core's change tracker can provide direct evidence.

## Related Collections

Updating collections such as:

```text
Products
Stakeholders
```

requires thinking about whether the operation means:

- append;
- replace;
- remove;
- synchronize;
- upsert.

These are different business operations and should not be treated as identical.

## Lessons Learned

- Persistence behavior depends on entity state.
- Updating a graph is more complex than updating a single row.
- Broad state changes can hide the real lifecycle of an entity.
- Related collections require explicit business semantics.

## Engineering Takeaway

A reliable update implementation starts by defining **what changed**, then choosing the EF Core state-management strategy that represents that change accurately.

## Code example

```csharp
public async Task<Result<ProjetoResponseDTO>> HandleAsync(int id, ProjetoUpdateDTO dto, int requestingUserId)
{
    var projeto = await projetoRepository.GetByIdAsync(id);
    if (projeto is null)
        return Result<ProjetoResponseDTO>.Failure("Projeto não encontrado.", 404);

    if (!projeto.IsActive)
        return Result<ProjetoResponseDTO>.Failure("Projeto inativo não pode ser editado.", 400);

    // IsActive é controlado exclusivamente por DesativarProjetoCase.
    // Nunca atribuir IsActive aqui — o valor existente da entidade rastreada
    // deve ser preservado em qualquer atualização de projeto.

    if (dto.Name is not null) projeto.Name = dto.Name;
    if (dto.Description is not null) projeto.Description = dto.Description;
    if (dto.TipoProjeto is not null) projeto.TipoProjeto = dto.TipoProjeto;
    if (dto.GerenteProjetoId.HasValue) projeto.GerenteProjetoId = dto.GerenteProjetoId.Value;
    if (dto.SponsorId.HasValue) projeto.SponsorId = dto.SponsorId.Value;
    if (dto.DataInicio.HasValue) projeto.DataInicio = dto.DataInicio.Value;
    if (dto.DataFimPrevista.HasValue) projeto.DataFimPrevista = dto.DataFimPrevista.Value;
    if (dto.DataFimReal.HasValue) projeto.DataFimReal = dto.DataFimReal.Value;
    if (dto.OrcamentoPrevisto.HasValue) projeto.OrcamentoPrevisto = dto.OrcamentoPrevisto.Value;
    if (dto.OrcamentoRealizado.HasValue) projeto.OrcamentoRealizado = dto.OrcamentoRealizado.Value;

    var updated = await projetoRepository.UpdateAsync(projeto);
    return Result<ProjetoResponseDTO>.Success(new ProjetoResponseDTO(updated));
}
```
