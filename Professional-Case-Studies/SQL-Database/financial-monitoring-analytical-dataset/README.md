# Case Study — Analytical Dataset Evolution for Financial Monitoring

> **Confidentiality note:** this case study is an anonymized and abstracted version of a specification produced in a real professional context. System names, organizations, entities, identifiers, profiles, integrations, API contracts, endpoints, routes, database structures and objects, payloads, documents, values, status codes, paths, personal data, and other technical or operational details have been removed, changed, or generalized to preserve confidentiality. No code, data, contract, physical structure, or infrastructure detail from the original environment is reproduced. The requirements analysis logic, specification patterns, functional decisions, and concepts needed to demonstrate the professional approach have been preserved.

## 1. Context

This case study represents the evolution of an Oracle SQL query used to build an **analytical dataset for financial monitoring**, consumed by a corporate Business Intelligence solution.

The analytical product supports departments responsible for monitoring process preparation before and after payment, providing a consolidated view of information originally distributed across different domains of the data model.

The query already existed and **was not originally created by me**. My work focused on relevant evolutions requested throughout the operational use of the dashboard. Functional needs were raised by management, while investigating the data model, defining the required relationships, and implementing the SQL changes were my responsibility.

## 2. Conceptual view of the dataset

The query consolidates different stages of the financial lifecycle into a single analytical view:

```text
Allocation / Business Reference
              ↓
        Business Rules
              ↓
          Commitment
              ↓
        Linked Account
              ↓
           Payment
              ↓
      Analytical Dataset / BI
```

This consolidation enables operational teams to monitor the process without manually reconstructing relationships across multiple transactional structures.

## 3. My contribution

My contributions can be grouped into three areas.

### 3.1 Operational account traceability

A requirement was raised to include branch and account information associated with business allocations.

The account is created as part of the operational flow associated with financial processing. The responsible team therefore needs to identify later **which business context originated a specific account opening**.

A typical use case occurs when a beneficiary asks why a particular account was opened. The analytical dataset allows the operational team to correlate the account with the allocation and payment context, reducing manual investigation across multiple information sources.

The functional request came from management. Identifying the appropriate data paths and implementing the SQL solution were performed by me based on accumulated domain knowledge and understanding of the relational model.

### 3.2 Evolution of responsible-member identification

Another change involved identifying the parliamentary member associated with certain types of budget allocations.

In some scenarios, the formal holder associated with an amendment does not necessarily represent the member who indicated or supported a specific beneficiary. The query evolved to account for this distinction and, when applicable, prioritize the **requesting/supporting parliamentary member**.

This evolution occurred in the context of increased public requirements for transparency and traceability of parliamentary amendments.

Conceptually, the rule can be represented as:

```sql
CASE
    WHEN transparency_rule_applies = 1
    THEN COALESCE(supporting_member, formal_author)
    ELSE formal_author
END AS responsible_member
```

This example is intentionally generic and does not reproduce fields, codes, or structures from the original environment.

### 3.3 Sensitive attribute removal

I also implemented a management-requested change to remove an attribute classified as sensitive from the analytical dataset.

The decision to classify and remove the information was made by management; my responsibility was to implement the technical change while preserving the remaining behavior of the query and dataset.

For confidentiality reasons, the attribute, its origin, retrieval rule, and other characteristics are not described in this case study.

## 4. Technical challenge — historical evolution of the account model

Adding branch and account information could not be solved with a single JOIN.

The relationship model between accounts and business allocations had evolved over the years. Records belonging to the current model have a more direct association, while legacy records require reconstructing the relationship through intermediate entities.

The conceptual strategy is similar to:

```text
                              ┌─ Current model ── direct relationship
Business Reference ──────────┤
                              └─ Legacy model ─── historical reconstruction
                                                       ↓
                                             account normalization
                                                       ↓
                                                consolidation
```

The solution uses two paths and combines them before joining them to the main analytical dataset.

An anonymized example:

```sql
WITH account_reference AS (
    SELECT DISTINCT
        current_data.business_reference,
        current_data.category,
        current_data.branch_number,
        current_data.account_number
    FROM current_account_relation current_data
    WHERE current_data.model_version = 'CURRENT'

    UNION ALL

    SELECT DISTINCT
        legacy_data.business_reference,
        legacy_data.category,
        legacy_data.branch_number,
        legacy_data.account_number
    FROM legacy_process legacy_data
    JOIN legacy_account_relation relation
      ON relation.process_id = legacy_data.process_id
    WHERE legacy_data.model_version = 'LEGACY'
)
SELECT ...
FROM analytical_base base
LEFT JOIN account_reference account
       ON account.business_reference = base.business_reference
      AND account.category = base.category;
```

The code above was recreated exclusively for this portfolio and does not represent the original physical data model.

## 5. Consolidating 1:N relationships

The dataset must represent information that naturally contains 1:N relationships.

A business allocation may be related to multiple financial records throughout its lifecycle. A direct JOIN could multiply rows in the main dataset and distort analytical indicators.

For that reason, some datasets are prepared before they are associated with the main query.

An equivalent pattern is:

```sql
WITH financial_events AS (
    SELECT
        business_reference,
        category,
        SUM(amount) AS total_amount,
        LISTAGG(document_number, ' | ')
            WITHIN GROUP (ORDER BY event_date DESC) AS documents,
        LISTAGG(TO_CHAR(event_date, 'DD/MM/YYYY'), ' | ')
            WITHIN GROUP (ORDER BY event_date DESC) AS event_dates
    FROM normalized_financial_event
    GROUP BY
        business_reference,
        category
)
SELECT ...
FROM analytical_base base
LEFT JOIN financial_events event
       ON event.business_reference = base.business_reference
      AND event.category = base.category;
```

This allows multiple legitimate events to be represented without turning every occurrence into another row of the main analytical record.

## 6. Retrieving the latest state

Status history is another naturally 1:N relationship: a single business reference may have several state transitions.

To retrieve only the most recent state, the query conceptually uses an analytical-function pattern:

```sql
SELECT business_reference,
       status_description,
       status_date
FROM (
    SELECT
        business_reference,
        status_description,
        status_date,
        ROW_NUMBER() OVER (
            PARTITION BY business_reference
            ORDER BY status_date DESC
        ) AS rn
    FROM status_history
)
WHERE rn = 1;
```

This prevents the full status history from multiplying rows in the analytical layer when the objective is to display the current state.

## 7. Transactional data vs. analytical data

One of the central lessons from this work is that a query intended for Analytics/BI should not simply reproduce JOINs between transactional tables.

The dataset needs to transform different structures into a representation suitable for analytical consumption:

- histories must be reduced to the relevant state;
- 1:N relationships must be consolidated when the analytical grain requires one main row;
- legacy and current models must coexist;
- null values must be handled predictably;
- attributes must respect business rules and information-exposure decisions;
- regulatory or operational changes may alter the interpretation of an attribute without necessarily changing its physical source.

## 8. Anonymized SQL architecture example

```sql
WITH latest_status AS (
    SELECT business_reference, status_description
    FROM (
        SELECT
            business_reference,
            status_description,
            ROW_NUMBER() OVER (
                PARTITION BY business_reference
                ORDER BY status_date DESC
            ) AS rn
        FROM status_history
    )
    WHERE rn = 1
),
commitment_summary AS (
    SELECT
        business_reference,
        category,
        SUM(amount) AS committed_amount,
        LISTAGG(document_number, ' | ')
            WITHIN GROUP (ORDER BY document_date DESC) AS documents
    FROM commitment
    GROUP BY business_reference, category
),
account_summary AS (
    SELECT
        business_reference,
        category,
        LISTAGG(branch_number, ' | ')
            WITHIN GROUP (ORDER BY branch_number) AS branches,
        LISTAGG(account_number, ' | ')
            WITHIN GROUP (ORDER BY account_number) AS accounts
    FROM normalized_account_relation
    GROUP BY business_reference, category
),
payment_summary AS (
    SELECT
        business_reference,
        category,
        SUM(NVL(amount, 0)) AS paid_amount
    FROM payment
    GROUP BY business_reference, category
)
SELECT
    base.reference_year,
    base.region,
    base.beneficiary,
    base.resource_type,
    base.programmed_amount,
    commitment.committed_amount,
    account.branches,
    account.accounts,
    payment.paid_amount,
    status.status_description
FROM analytical_base base
LEFT JOIN latest_status status
       ON status.business_reference = base.business_reference
LEFT JOIN commitment_summary commitment
       ON commitment.business_reference = base.business_reference
      AND commitment.category = base.category
LEFT JOIN account_summary account
       ON account.business_reference = base.business_reference
      AND account.category = base.category
LEFT JOIN payment_summary payment
       ON payment.business_reference = base.business_reference
      AND payment.category = base.category;
```

This example demonstrates only the technical pattern. Entities, names, rules, dates, codes, and relationships have been replaced or simplified.

## 9. Operational impact

The changes increased the usefulness of the dataset for teams responsible for monitoring the financial process.

Adding account information improved operational traceability between the business allocation and the account opened as part of the flow. The responsible-member rule improved how information is represented under evolving transparency requirements. Removing the sensitive attribute aligned the dataset with management's information-exposure decision.

The result was an evolution of the analytical layer that reduced the need for users to manually reconstruct these relationships across different information sources.

## 10. Transparency and public context

The concept of a **supporting/requesting parliamentary member** is used in public transparency mechanisms to identify members who indicated or supported amendments or beneficiaries.

As a public contextual reference, Brazil's Transparency Portal provides a dedicated consultation related to amendment supporters:

**Transparency Portal — Amendment Supporters**  
https://portaldatransparencia.gov.br/emendas/apoiadores

This reference is included only to provide public context for the concept. It **does not document or imply a technical integration** between the environment described in this case study and the Transparency Portal.

The corporate dashboard consuming the dataset in this study is not referenced because its access was not identified as public and unrestricted.

## 11. Skills demonstrated

- Oracle SQL;
- Analytics / BI data preparation;
- analytical dataset modeling;
- data-oriented requirements analysis;
- translation of operational needs into SQL rules;
- legacy query maintenance and evolution;
- `ROW_NUMBER()` and analytical functions;
- `LISTAGG`;
- `SUM` and `GROUP BY`;
- `DISTINCT`;
- `UNION ALL`;
- `INNER JOIN` and `LEFT JOIN`;
- `CASE`, `COALESCE`, and `NVL`;
- 1:N cardinality handling;
- coexistence of current and legacy data models;
- data traceability;
- temporal rules and business evolution;
- data governance and sensitive-information handling;
- domain knowledge applied to data investigation.

## 12. Authorship and transparency

The query that originated this study was already part of an existing solution and was not originally developed by me.

My contribution corresponds to the evolutions described in this document: investigating the required relationships, implementing account traceability, adapting the responsible-member identification rule, and implementing the removal of a sensitive attribute at management's request.

All SQL examples in this case study were recreated specifically for the portfolio. No original code, physical database structure, real data, or information classified as sensitive is reproduced.
