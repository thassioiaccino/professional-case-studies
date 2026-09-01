# Cloud Journey — Engineering Portfolio

> **Real-world systems experience, engineering case studies, and a hands-on journey toward Cloud, DevOps and Platform Engineering.**

This repository is my engineering portfolio.

It combines two parts of the same professional path:

1. **Professional case studies** derived from real work with critical information systems, covering requirements engineering, Oracle SQL, production incident investigation, integrations, data consistency and legacy-system modernization.
2. **Cloud & DevOps engineering projects** built hands-on as I expand my background toward infrastructure, automation, cloud computing and platform engineering.

The goal is not to collect tutorials or copied labs. Every item published here should demonstrate a problem, the reasoning behind the solution, the decisions made and what I learned from it.

> **Build → Understand → Document → Improve**

---

## Professional Background

My background is in **Product Ownership, Requirements Analysis and critical-system operations**, working at the intersection of business rules, software engineering teams, databases, integrations and production operations.

My experience includes translating operational needs into implementable requirements, validating complex business rules, investigating production behavior with SQL and system evidence, supporting integrations between systems, leading functional regression and acceptance activities, and preserving business behavior during legacy-system modernization.

I am now extending that foundation into **Cloud, DevOps and Platform Engineering**, with an initial focus on AWS, Linux, Git/GitHub, containers, Infrastructure as Code, CI/CD, Kubernetes and reliability practices.

This portfolio is intentionally designed to show that progression rather than present the Cloud journey as a career starting from zero.

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

## Cloud & DevOps Journey

The second pillar of this repository is a structured transition toward Cloud and Platform Engineering through hands-on projects.

The roadmap is deliberately sequential: foundations first, automation and orchestration later.

| Stage | Status | Focus |
|---|---|---|
| AWS Fundamentals | Completed | Core cloud concepts and AWS foundations |
| Linux Fundamentals | Completed | OS, filesystem, permissions, users, packages, services, networking and SSH |
| Linux Practical Project | In progress | Provisioning and administering a Linux server from scratch |
| Git & GitHub | Planned | Version control and engineering workflows |
| Docker | Planned | Containers and application packaging |
| Terraform | Planned | Infrastructure as Code |
| GitHub Actions / CI/CD | Planned | Build and delivery automation |
| Kubernetes | Planned | Container orchestration |
| Azure | Planned | Multi-cloud infrastructure knowledge |
| Platform Engineering / SRE | Long-term direction | Reliability, platforms, automation and developer experience |

See the detailed [`ROADMAP.md`](ROADMAP.md).

---

## Current Hands-on Project

### Project 001 — Linux Server: Provisioning & Administration

A clean virtual machine is being provisioned from scratch as a realistic infrastructure exercise rather than a command-by-command tutorial.

The project covers operating-system installation, identity and access design, filesystem permissions, package management, web-service deployment, service/process administration, networking, SSH, logs and a mandatory controlled incident with investigation and recovery.

The project will only be published as a completed portfolio artifact after technical review and validation of the decisions taken during implementation.

---

## Repository Structure

```text
cloud-journey/
├── Professional-Case-Studies/
│   ├── Requirements-Engineering/
│   ├── SQL-Database/
│   ├── Operations-Incident-Management/
│   └── System-Modernization/
├── docs/
├── README.md
└── ROADMAP.md
```

New Cloud/DevOps project directories will be added as practical work is completed. The repository structure follows the portfolio that actually exists rather than reserving empty folders for future technologies.

---

## Engineering Principles

**Evidence over claims.** Skills should be demonstrated through decisions, artifacts and working projects.

**Understand before publishing.** Course exercises and copied labs are training material, not portfolio projects unless they are substantially adapted and understood.

**Business and technology belong together.** Technical solutions are more valuable when the underlying operational problem and constraints are understood.

**Documentation is an engineering artifact.** Requirements, architecture decisions, troubleshooting evidence and operational knowledge are part of the solution.

**Failures are learning material.** Incidents and mistakes are documented through symptom → investigation → hypothesis → correction → validation.

**Confidentiality by design.** Professional case studies are anonymized and recreated to demonstrate reasoning without exposing internal systems, data, code or infrastructure.

---

## What Comes Next

The immediate focus is completing and technically reviewing **Project 001 — Linux Server**. From there, the journey advances through Git/GitHub, Docker, Terraform, CI/CD and Kubernetes, progressively combining them into larger infrastructure and platform-engineering projects.

This repository will evolve with that progression — but only when new knowledge becomes something I can explain, build, troubleshoot and defend technically.
