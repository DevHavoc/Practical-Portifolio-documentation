# Stakeholder Excel Import

## Context

A project-management module required improvements to stakeholder registration.

The feature involved:

- adding stakeholder email;
- adding stakeholder phone;
- importing stakeholders from Excel;
- supporting bulk operations;
- preserving project-level authorization;
- keeping the database schema, DTOs, API and UI consistent.

The implementation was a useful example of a feature that initially looked like a simple upload but actually crossed multiple application layers.

## Architecture Flow

```text
SAPUI5
  ↓
File selection
  ↓
RequestModel
  ↓
HTTP multipart/form-data
  ↓
ASP.NET Core Controller
  ↓
Stakeholder Use Case
  ↓
Project validation / authorization
  ↓
Excel parsing
  ↓
Column normalization
  ↓
DTO / entity mapping
  ↓
Repository
  ↓
Entity Framework Core
  ↓
PostgreSQL
```

## Database Changes

The stakeholder entity was extended with:

- `Email`
- `Telefone`

A database migration was required so the application model and PostgreSQL schema stayed synchronized.

Phone numbers were treated as text rather than numeric data because phone numbers can contain:

- leading zeroes;
- country codes;
- separators;
- extensions;
- formatting characters.

## Excel Import

The import was implemented with an Excel-reading library capable of processing spreadsheet data.

A practical concern was that spreadsheets created by different people rarely have perfectly consistent headers.

For example, the same conceptual field can appear as:

```text
Nome
NOME
NomeExterno
E-mail
Email
Telefone
Organizacao
Organização
```

A robust importer should normalize headers before mapping them.

Conceptually:

```text
Raw header
   ↓
Trim
   ↓
Case normalization
   ↓
Accent normalization
   ↓
Alias resolution
   ↓
Canonical field
```

This makes the import less dependent on superficial formatting differences.

## Authorization

The import belonged to a project-specific resource.

Therefore, the endpoint was not treated as a generic global import.

The application first needed to establish that the current user could access the requested project before modifying its stakeholders.

This illustrates an important distinction:

```text
Authentication
    ≠
Authorization
    ≠
Data validation
```

A valid authenticated user does not automatically have permission to modify every project.

## Performance Investigation

A test spreadsheet contained approximately 2,300 rows.

At that scale, several questions become important:

- How much memory is consumed?
- Is the entire workbook loaded at once?
- Are all rows converted to objects before insertion?
- How many database operations are executed?
- Is `SaveChangesAsync()` called once or repeatedly?
- What happens when one row is invalid?
- Can the HTTP request remain open long enough to finish?

A request timing out does not necessarily mean that the import failed.

The correct investigation is:

```text
HTTP timeout
   ↓
Check server logs
   ↓
Check use case execution
   ↓
Check database activity
   ↓
Check transaction state
   ↓
Check resulting records
```

## Persistence Pitfall

One important EF Core failure mode is creating entities and then calling `SaveChangesAsync()` without adding those entities to the DbContext.

This:

```text
Create entity objects
      ↓
SaveChanges()
```

does not automatically mean:

```text
INSERT
```

The DbContext must know that the entities exist and should be inserted.

The correct conceptual sequence is:

```text
Create objects
   ↓
Add / AddRange
   ↓
Entity state = Added
   ↓
SaveChangesAsync
   ↓
INSERT
```

## Lessons Learned

### 1. File upload is a full-stack feature

The feature crossed the UI, HTTP, API, application, persistence and database layers.

### 2. Real-world spreadsheets are inconsistent

Importers should normalize and validate input instead of assuming ideal headers.

### 3. Bulk operations need explicit performance thinking

A few hundred rows and a few thousand rows can expose very different problems.

### 4. Successful HTTP responses are not proof of correct persistence

The database must be verified independently.

### 5. Authorization must follow the resource hierarchy

Project-specific operations should validate the relationship between the authenticated user and the target project.

## Possible Future Improvements

- maximum file size;
- maximum row count;
- streaming where appropriate;
- batch insertion;
- transaction strategy;
- duplicate detection;
- row-level validation report;
- partial-success reporting;
- import audit logs;
- asynchronous processing for very large files.

## Engineering Takeaway

The most important lesson was that an apparently simple requirement — **"import an Excel file"** — becomes a systems problem once real data, authorization, persistence and performance are involved.
