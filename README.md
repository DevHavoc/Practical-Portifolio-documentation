# Practical Portfolio Documentation

A technical portfolio focused on **engineering reasoning, problem solving, debugging, and implementation experience**.

This repository documents selected software engineering work performed in a professional environment without exposing proprietary source code, credentials, internal URLs, confidential business data, or company-specific implementation details.

## Purpose

The goal is to demonstrate not only *what* was implemented, but also:

- how a problem was investigated;
- how the application architecture was understood;
- how bugs were isolated;
- why a particular solution was chosen;
- what trade-offs were identified;
- what was learned from the implementation.

The documentation is intentionally implementation-oriented rather than theoretical.

## Technical Areas

- C#
- ASP.NET Core
- Entity Framework Core
- PostgreSQL
- REST APIs
- SAPUI5
- FluentMigrator
- Git
- Dependency Injection
- Clean Architecture concepts
- Data import and processing
- Access management
- Debugging and troubleshooting
- Deployment and environment configuration

## Case Studies

1. [Stakeholder Excel Import](01-stakeholder-excel-import.md)
2. [Bulk Import Performance](02-bulk-import-performance.md)
3. [Entity Framework Core Tracking](03-ef-core-tracking.md)
4. [Bulk Persistence with EF Core](04-create-range-persistence.md)
5. [SAPUI5 to ASP.NET Core Request Flow](05-sapui5-request-flow.md)
6. [Database Schema Evolution with FluentMigrator](06-fluentmigrator-schema.md)
7. [Result Pattern in .NET](07-result-pattern-dotnet.md)
8. [User and Access Management](08-user-access-management.md)
9. [Dependency Injection Troubleshooting](09-dependency-injection-troubleshooting.md)
10. [NuGet and Deployment Troubleshooting](10-nuget-deployment-troubleshooting.md)
11. [REST API Architecture](11-rest-api-architecture.md)
12. [Resource Authorization and IDOR Analysis](12-resource-authorization-and-idor.md)
13. [Production Debugging Methodology](13-production-debugging-methodology.md)
14. [Project Update and Persistence Behavior](14-project-update-persistence.md)

## Engineering Philosophy

A recurring principle throughout these case studies is:

> Do not fix the first symptom you see. Trace the system until you find the first layer where reality diverges from expectation.

For backend and full-stack problems, that usually means following the complete chain:

```text
UI
 ↓
Request construction
 ↓
HTTP API
 ↓
Controller
 ↓
Use Case
 ↓
Repository
 ↓
ORM
 ↓
Database
```

The same approach applies to deployment, dependency injection, persistence, and authorization problems.

## Confidentiality

The repository contains generalized technical documentation only.

No proprietary source code, credentials, confidential records, internal URLs, or sensitive company information should be published here.

The examples are intentionally abstracted so that the engineering reasoning remains visible without exposing the original application's implementation.
