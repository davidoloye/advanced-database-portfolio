# Advanced Database Portfolio

**Organisation:** eProcess Nigeria
**Domain:** Software Development (Fintech Services)

This repository contains a series of mini‑projects that demonstrate key database concepts and techniques tailored for a fintech system. The projects are organised into five portfolios, each focusing on a different aspect of database engineering:

```
advanced-database-portfolio/
├── portfolio-1-database-design/
├── portfolio-2-query-optimisation/
├── portfolio-3-transactions-concurrency/
├── portfolio-4-nosql/
├── portfolio-5-distributed-cloud/
└── README.md   <-- you are here
```

## Who uses the database?
- **Developers** – build services that interact with the data.
- **Business Analysts** – run reports and exploratory queries.
- **CTOs** – review architecture and performance.
- **QA Engineers** – validate data integrity and functional correctness.
- **Project Managers** – track progress of database‑related tasks.

## Core Data Model (fictional)
- **Customers** – basic personal information.
- **Accounts** – each customer may have one or more accounts.
- **Transactions** – transfers, deposits, withdrawals.
- **Transaction Logs** – audit trail for every change.

The subsequent portfolios build on this model, providing concrete DDL/DML, queries, performance tuning, concurrency handling, NoSQL alternatives, and a cloud‑distributed deployment.

---
### How to run the examples
Each portfolio contains its own `README.md` with step‑by‑step instructions, required tools, and how to verify the results (e.g., view screenshots, execution plans, logs).  Most examples use PostgreSQL; the NoSQL portfolio uses MongoDB, and the distributed‑cloud portfolio uses Google Cloud SQL together with Terraform.

---
### Technical Reflection (overall)
See `technical-reflection.md` inside each portfolio for a short write‑up of design choices, challenges, and lessons learned.
