# Practical Portfolio Documentation 📑

A technical portfolio focused on **engineering reasoning, problem solving, debugging, and implementation experience**.

This repository documents selected software engineering work performed in a professional environment without exposing proprietary source code, credentials, internal URLs, confidential business data, or company-specific implementation details.

---

## 📑 Engineering Architecture Domains

Para facilitar a navegação, os estudos de caso foram organizados e centralizados por domínios de arquitetura. Clique em qualquer uma das categorias abaixo para explorar os arquivos de documentação diretamente:

*   🧠 **[backend](./docs/01-backend)** — Core architecture, dependency injection, business rules, and C#/.NET engine.
*   🗄️ **[data](./docs/02-data)** — Data processing, ORM performance, migrations, and high-volume reports.
*   🛡️ **[security](./docs/03-security)** — Identity, multi-tenancy isolation, resource authorization, and security edge cases.
*   🌐 **[frontend](./docs/04-frontend)** — Enterprise UI synchronization, SAPUI5 state machine, and client-side integrations.
*   ☁️ **[cloud](./docs/05-cloud)** — Deployment troubleshooting, production debugging methodologies, and infrastructure logs.

---

## Recent Case Studies

The latest implementation series covers administrative workflows, access registration, permission enforcement, dashboard summaries, and financial-report query optimization. Each case separates delivered behavior from validation limits and future improvements.

- [List Filtering, Sorting and Bulk Action Consistency](./docs/04-frontend/29-list-filtering-sorting-and-bulk-actions.md)
- [Selection-Aware Exports and Report Boundaries](./docs/02-data/30-selection-aware-exports-and-report-boundaries.md)
- [Batch Access Registration and DbContext Concurrency](./docs/01-backend/31-batch-access-registration-and-dbcontext-concurrency.md)
- [User Overview Permission Gating](./docs/03-security/32-user-overview-permission-gating.md)
- [Summary Cards, Result Counts and Pagination](./docs/04-frontend/33-summary-cards-result-counts-and-pagination.md)
- [Financial Report Query Optimization with OpenProject](./docs/02-data/34-dre-openproject-query-optimization.md)
- [Regression Testing and Evidence-Driven Delivery](./docs/05-cloud/35-regression-testing-and-evidence-driven-delivery.md)

---

## Purpose

The goal is to demonstrate not only *what* was implemented, but also:

- how a problem was investigated;
- how the application architecture was understood;
- how bugs were isolated;
- why a particular solution was chosen;
- what trade-offs were identified;
- what was learned from the implementation.

The documentation is intentionally implementation-oriented rather than theoretical.

---

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

---

## Confidentiality

The repository contains generalized technical documentation only.

No complete proprietary application files, credentials, confidential records, internal URLs, or sensitive company information are published here. Examples are short code excerpts or adapted illustrations with sensitive details removed.

The examples explain engineering decisions without exposing complete private application files or confidential records.

---

## 🌐 Author & Connect

- **GitHub:** [DevHavoc](https://github.com)
- **LinkedIn:** [Victor Ferreira Guimarães](https://linkedin.com)


## New implementation series

- [Long-Running Reports, HTTP 499 and NDJSON Streaming](./docs/05-cloud/36-long-running-report-streaming-and-client-cancellation.md)
- [Persisted Reporting Snapshots and Cache Validation](./docs/02-data/37-persisted-reporting-snapshots-and-cache-validation.md)
- [Incremental Synchronization, Revisions and Deletions](./docs/02-data/38-incremental-sync-revisions-and-deletions.md)
- [Background Report Synchronization with Hosted Services](./docs/01-backend/39-background-report-sync-with-hosted-services.md)
- [Distributed Synchronization Leases and Atomic Publication](./docs/05-cloud/40-distributed-sync-leases-and-atomic-publication.md)
- [Stale Search Responses and Synchronization Status](./docs/04-frontend/41-stale-search-responses-and-sync-status.md)
- [Backfilling Native Access for Existing Administrators](./docs/03-security/42-backfilling-native-access-for-existing-admins.md)
- [Production Migration Rehearsal with PostgreSQL](./docs/02-data/43-production-migration-rehearsal-with-postgresql.md)
- [Merge Readiness and Deployment Boundaries](./docs/05-cloud/44-merge-readiness-and-deployment-boundaries.md)
- [GitHub Profile SVG Links and Image Proxies](./docs/04-frontend/45-github-profile-svg-links-and-image-proxies.md)

## Code example

```csharp
public void RegistrarVenda(decimal valor)
{
    if (valor <= 0)
        throw new ExceptionVenda("Valor inválido!");

    Saldo += valor;
}
```
