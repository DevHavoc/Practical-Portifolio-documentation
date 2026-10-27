# Practical Portfolio Documentation 📑

A technical portfolio focused on **engineering reasoning, problem solving, debugging, and implementation experience**.

This repository documents selected software engineering work performed in a professional environment using short excerpts from work I implemented or changed, without publishing credentials, internal URLs or confidential business data.

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

The case studies include explanations and short examples from the implementation.

No complete proprietary application files, credentials, confidential records, internal URLs, or sensitive company information are published here. The code examples use real implementation excerpts. Descriptions explain their purpose and limits.

The examples explain engineering decisions without exposing complete private application files or confidential records.

---

## 🌐 Author & Connect

- **GitHub:** [DevHavoc](https://github.com)
- **LinkedIn:** [Victor Ferreira Guimarães](https://linkedin.com)


## New implementation series

- [Keeping Long Reports Connected with NDJSON](./docs/05-cloud/36-long-running-report-streaming-and-client-cancellation.md)
- [Saving Report Data for Reuse After a Restart](./docs/02-data/37-persisted-reporting-snapshots-and-cache-validation.md)
- [Updating Changed Records Without Reloading Every Year](./docs/02-data/38-incremental-sync-revisions-and-deletions.md)
- [Preparing Report Data in the Background](./docs/01-backend/39-background-report-sync-with-hosted-services.md)
- [Preventing Two Servers from Saving the Same Update](./docs/05-cloud/40-distributed-sync-leases-and-atomic-publication.md)
- [Keeping Old Search Responses Out of the Current Screen](./docs/04-frontend/41-stale-search-responses-and-sync-status.md)
- [Giving Existing Admins the Missing User Overview Access](./docs/03-security/42-backfilling-native-access-for-existing-admins.md)
- [Testing the Database Upgrade with PostgreSQL](./docs/02-data/43-production-migration-rehearsal-with-postgresql.md)
- [Checking What a Merge and Deployment Will Include](./docs/05-cloud/44-merge-readiness-and-deployment-boundaries.md)
- [Fixing Broken Icons in a GitHub Profile README](./docs/04-frontend/45-github-profile-svg-links-and-image-proxies.md)

## Code example

```csharp
public void RegistrarVenda(decimal valor)
{
    if (valor <= 0)
        throw new ExceptionVenda("Valor inválido!");

    Saldo += valor;
}
```
