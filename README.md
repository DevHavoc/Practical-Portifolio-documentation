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
