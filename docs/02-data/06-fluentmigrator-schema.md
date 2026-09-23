# Database Schema Evolution with FluentMigrator

## Context

Adding fields to an existing entity requires more than modifying the C# model.

For stakeholder management, new fields such as email and phone required synchronization between:

```text
Domain / Entity
 ↓
DTOs
 ↓
Application logic
 ↓
Database migration
 ↓
PostgreSQL schema
 ↓
Frontend
```

## Why Migrations Matter

Changing the model locally does not change the production database.

A migration provides a repeatable description of the schema change.

Conceptually:

```text
Previous schema
      ↓
Migration
      ↓
New schema
```

This makes database changes versioned and reproducible.

## Data Type Decisions

Phone numbers should generally be represented as text.

A phone number is an identifier-like value, not a quantity to calculate.

Text preserves:

- leading zeroes;
- country prefixes;
- separators;
- extensions;
- formatting.

For example:

```text
+55 62 99999-9999
```

should not be treated as an integer.

## DTO Consistency

A schema change should be reflected in request and response contracts.

Otherwise, different layers can disagree about the available fields.

A useful consistency checklist is:

```text
Entity
 ↓
Migration
 ↓
DTO
 ↓
Use Case
 ↓
Repository
 ↓
API response
 ↓
Frontend model
```

## Lessons Learned

- Database changes are application changes.
- Migrations are part of the feature, not an afterthought.
- Data types should reflect business meaning.
- DTOs and frontend models must remain aligned with the backend.

## Engineering Takeaway

A reliable feature keeps the data contract synchronized from the database all the way to the UI.
