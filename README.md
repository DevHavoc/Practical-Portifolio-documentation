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

No proprietary source code, credentials, confidential records, internal URLs, or sensitive company information should be published here.

The examples are intentionally abstracted so that the engineering reasoning remains visible without exposing the original application's implementation.

---

## 🌐 Author & Connect

- **GitHub:** [DevHavoc](https://github.com)
- **LinkedIn:** [Victor Ferreira Guimarães](https://linkedin.com)
