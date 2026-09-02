# Cloud Journey — Engineering Portfolio

> **Real-world systems experience, engineering case studies, and hands-on projects across software, data, operations and infrastructure.**

This repository is my **public engineering portfolio**.

It brings together selected artifacts that demonstrate how I work across requirements engineering, Oracle SQL, production incident investigation, integrations, data consistency, legacy-system modernization and, as they reach publication quality, Cloud/DevOps engineering projects.

The goal is not to collect tutorials, course exercises or unfinished labs. Every item published here should demonstrate a real problem or meaningful technical challenge, the reasoning behind the solution, the decisions made, validation and what I learned from it.

> **Build → Understand → Review → Document → Publish**

---

## Professional Background

My background is in **Product Ownership, Requirements Analysis and critical-system operations**, working at the intersection of business rules, software engineering teams, databases, integrations and production operations.

My experience includes translating operational needs into implementable requirements, validating complex business rules, investigating production behavior with SQL and system evidence, supporting integrations between systems, leading functional regression and acceptance activities, and preserving business behavior during legacy-system modernization.

I am extending that foundation into **Cloud, DevOps and Platform Engineering**, adding infrastructure, automation and reliability practices to an existing background in critical systems and production-oriented problem solving.

This portfolio is intentionally designed to show that progression rather than present the engineering journey as a career starting from zero.

---

## Featured Professional Case Studies

The case studies below are **anonymized and abstracted from real professional situations**. Confidential system names, internal endpoints, database objects, production data, infrastructure details and other sensitive information are not reproduced.

### 1. Legacy System Modernization — Functional Governance

A case study about preserving business behavior during a major technology modernization of a critical legacy application.

Highlights:
- functional baseline reconstruction in an environment without consolidated documentation;
- legacy vs. modernized environment comparison;
- business-rule clarification for developers;
- regression testing and functional acceptance;
- defect classification and prioritization;
- functional go/no-go participation;
- post-production monitoring and regression identification.

[Read the case study →](Professional-Case-Studies/System-Modernization/legacy-system-modernization-functional-governance/README.md)  
[Versão em Português →](Professional-Case-Studies/System-Modernization/legacy-system-modernization-functional-governance/README.pt.md)

### 2. Production Incident — Cross-System Transaction Reconciliation

A production incident in which an operation succeeded on an external financial platform while the corresponding local state was not correctly persisted after a system migration.

Highlights:
- incident detection during real operation;
- verification of external system state before retrying;
- database-driven investigation;
- cross-system consistency reasoning;
- controlled production data reconciliation;
- validation after recovery;
- separation between operational recovery and permanent application correction.

[Read the case study →](Professional-Case-Studies/Operations-Incident-Management/distributed-transaction-reconciliation/README.md)  
[Versão em Português →](Professional-Case-Studies/Operations-Incident-Management/distributed-transaction-reconciliation/README.pt.md)

### 3. Requirements Engineering — Stateful Synchronization Workflow

A complete requirements-engineering case involving synchronization between an external source and a stateful local application.

Highlights:
- scope and boundary definition;
- conditional uniqueness rules;
- new / unchanged / changed record classification;
- state-transition rules;
- backend revalidation;
- auditability;
- batch processing;
- exception handling;
- acceptance criteria and decision flows.

[Read the case study →](Professional-Case-Studies/Requirements-Engineering/stateful-synchronization-workflow/README.md)  
[Versão em Português →](Professional-Case-Studies/Requirements-Engineering/stateful-synchronization-workflow/README.pt.md)

### 4. Oracle SQL — Legacy Materialized View Refactoring

A database case focused on evolving a legacy analytical structure while supporting current and historical relationship models.

Highlights:
- Oracle SQL;
- materialized views;
- CTEs and complex joins;
- historical relationship reconstruction;
- one-to-many consolidation;
- `LISTAGG`, aggregation and conditional logic;
- duplicate-control reasoning.

[Read the case study →](Professional-Case-Studies/SQL-Database/legacy-materialized-view-refactoring/README.md)  
[Versão em Português →](Professional-Case-Studies/SQL-Database/legacy-materialized-view-refactoring/README.pt.md)

### 5. Oracle SQL — Financial Monitoring Analytical Dataset

Evolution of an analytical dataset used to support financial monitoring and business analysis.

Highlights:
- discovery of new data relationships from business needs;
- Oracle SQL across multiple relational sources;
- historical/current data-model handling;
- `ROW_NUMBER`, `LISTAGG`, `UNION ALL`, aggregation and deduplication;
- translating operational questions into analytical data structures.

[Read the case study →](Professional-Case-Studies/SQL-Database/financial-monitoring-analytical-dataset/README.md)  
[Versão em Português →](Professional-Case-Studies/SQL-Database/financial-monitoring-analytical-dataset/README.pt.md)

Additional requirements-engineering cases are available under [`Professional-Case-Studies/Requirements-Engineering`](Professional-Case-Studies/Requirements-Engineering/).

---

## Portfolio Publication Model

This repository contains **reviewed public artifacts**, not the operational backlog behind them.

Study notes, experiments, unfinished projects, raw troubleshooting trails and project planning remain outside the public portfolio. When a technical project reaches a point where I can reproduce it, explain its decisions, troubleshoot it and defend the implementation, it is curated and published here as a finished engineering artifact.

This keeps the portfolio focused on **evidence of capability rather than promises of future capability**.

---

## Repository Structure

```text
cloud-journey/
├── Professional-Case-Studies/
│   ├── Requirements-Engineering/
│   ├── SQL-Database/
│   ├── Operations-Incident-Management/
│   └── System-Modernization/
└── README.md
```

New categories and project directories are added only when there is a reviewed artifact ready to publish.

---

## Engineering Principles

**Evidence over claims.** Skills should be demonstrated through decisions, artifacts and working projects.

**Understand before publishing.** Course exercises and copied labs are training material, not portfolio projects unless they are substantially adapted and understood.

**Business and technology belong together.** Technical solutions are more valuable when the underlying operational problem and constraints are understood.

**Documentation is an engineering artifact.** Requirements, architecture decisions, troubleshooting evidence and operational knowledge are part of the solution.

**Failures are learning material.** Incidents and mistakes are documented through symptom → investigation → hypothesis → correction → validation.

**Confidentiality by design.** Professional case studies are anonymized and recreated to demonstrate reasoning without exposing internal systems, data, code or infrastructure.

---

## Portfolio Direction

The portfolio will continue to grow with reviewed professional cases and completed hands-on engineering projects. Publication happens only when new knowledge becomes something I can **explain, build, troubleshoot and defend technically**.
