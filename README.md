# Professional Case Studies — Engineering Portfolio

> **Anonymized real-world case studies across requirements engineering, Oracle SQL, production operations and legacy-system modernization.**

This repository is a curated collection of **professional case studies based on real situations from my work with critical systems**.

The goal is to demonstrate how I approach business rules, requirements, data, integrations, production incidents and system evolution through evidence, decisions and technical reasoning — without exposing confidential information from the original environments.

> **Experience → Analyze → Abstract → Document → Share**

---

## Professional Background

My background is in **Product Ownership, Requirements Analysis and critical-system operations**, working at the intersection of business rules, software engineering teams, Oracle databases, integrations and production operations.

My experience includes translating operational needs into implementable requirements, validating complex business rules, investigating production behavior with SQL and system evidence, supporting integrations between systems, conducting functional regression and acceptance activities, and preserving business behavior during legacy-system modernization.

These cases focus specifically on that **real professional experience**. My Cloud/DevOps hands-on projects are maintained separately from this repository.

---

## Featured Case Studies

All cases below are **anonymized and abstracted from real professional situations**. System names, organizations, internal endpoints, physical database objects, production data, infrastructure details and other sensitive information are not reproduced.

### 1. Legacy System Modernization — Functional Governance

Preserving business behavior during a major technology modernization of a critical legacy application.

Highlights:
- functional baseline reconstruction in an environment without consolidated documentation;
- legacy vs. modernized environment comparison;
- business-rule clarification for developers;
- regression testing and functional acceptance;
- defect classification and prioritization;
- functional go/no-go participation;
- post-production monitoring and regression identification.

[Read the case study →](System-Modernization/legacy-system-modernization-functional-governance/README.md)  
[Versão em Português →](System-Modernization/legacy-system-modernization-functional-governance/README.pt.md)

### 2. Production Incident — Cross-System Transaction Reconciliation

A production incident in which an operation succeeded on an external financial platform while the corresponding local state was not correctly persisted after a system migration.

Highlights:
- incident detection during real operation;
- verification of external state before retrying;
- database-driven investigation;
- cross-system consistency reasoning;
- controlled production data reconciliation;
- validation after recovery;
- separation between operational recovery and permanent application correction.

[Read the case study →](Operations-Incident-Management/distributed-transaction-reconciliation/README.md)  
[Versão em Português →](Operations-Incident-Management/distributed-transaction-reconciliation/README.pt.md)

### 3. Requirements Engineering — Stateful Synchronization Workflow

A comprehensive requirements-engineering case involving synchronization between an external source and a stateful local application.

Highlights:
- scope and boundary definition;
- conditional uniqueness rules;
- new / unchanged / changed record classification;
- state-transition rules;
- backend revalidation;
- auditability and batch processing;
- exception handling;
- acceptance criteria and decision flows.

[Read the case study →](Requirements-Engineering/stateful-synchronization-workflow/README.md)  
[Versão em Português →](Requirements-Engineering/stateful-synchronization-workflow/README.pt.md)

### 4. Oracle SQL — Legacy Materialized View Refactoring

Evolution of a legacy analytical structure while supporting current and historical relationship models.

Highlights:
- Oracle SQL and materialized views;
- CTEs and complex joins;
- historical relationship reconstruction;
- one-to-many consolidation;
- `LISTAGG`, aggregation and conditional logic;
- duplicate-control reasoning.

[Read the case study →](SQL-Database/legacy-materialized-view-refactoring/README.md)  
[Versão em Português →](SQL-Database/legacy-materialized-view-refactoring/README.pt.md)

### 5. Oracle SQL — Financial Monitoring Analytical Dataset

Evolution of an analytical dataset used to support financial monitoring and business analysis.

Highlights:
- discovery of new data relationships from business needs;
- Oracle SQL across multiple relational sources;
- historical/current data-model handling;
- `ROW_NUMBER`, `LISTAGG`, `UNION ALL`, aggregation and deduplication;
- translating operational questions into analytical data structures.

[Read the case study →](SQL-Database/financial-monitoring-analytical-dataset/README.md)  
[Versão em Português →](SQL-Database/financial-monitoring-analytical-dataset/README.pt.md)

Additional requirements-engineering cases are available under [`Requirements-Engineering`](Requirements-Engineering/).

---

## Repository Structure

```text
professional-case-studies/
├── Requirements-Engineering/
├── SQL-Database/
├── Operations-Incident-Management/
├── System-Modernization/
└── README.md
```

Each category groups cases by the primary engineering capability being demonstrated.

---

## Publication Model

This repository contains **reviewed public artifacts derived from professional experience**, not raw internal documentation.

The publication flow is intentionally conservative:

```text
Real professional situation
        ↓
Identify transferable knowledge
        ↓
Remove confidential details
        ↓
Anonymize and abstract the scenario
        ↓
Reconstruct the case study
        ↓
Review for confidentiality and accuracy
        ↓
Publish
```

The purpose is to prove professional capability **without revealing how the original internal environment operates**.

---

## Engineering Principles

**Evidence over claims.** Skills should be demonstrated through decisions, reasoning and artifacts.

**Business and technology belong together.** Technical solutions are more valuable when the underlying operational problem and constraints are understood.

**Documentation is an engineering artifact.** Requirements, decision flows, troubleshooting evidence and operational knowledge are part of the solution.

**Failures are valuable engineering evidence.** Production incidents can demonstrate investigation, risk assessment, recovery and learning when documented responsibly.

**Confidentiality by design.** Every professional case is anonymized and recreated to preserve the reasoning while protecting internal systems, data, code and infrastructure.

---

## Portfolio Scope

This repository is intentionally focused on **professional case studies**.

Hands-on Cloud/DevOps projects, labs and infrastructure engineering work belong to a separate portfolio track, keeping a clear distinction between **real professional experience** and **authorial technical projects built for learning and demonstration**.
