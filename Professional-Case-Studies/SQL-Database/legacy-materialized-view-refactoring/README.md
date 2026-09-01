# Case Study — Oracle Materialized View Evolution and Refactoring

> **Confidentiality notice:** this case study is an anonymized and abstracted version of a specification developed in a real professional context. Names of systems, organizations, entities, identifiers, profiles, integrations, API contracts, endpoints, routes, database structures and objects, payloads, documents, values, status codes, paths, personal data, and other technical or operational details have been removed, changed, or generalized to preserve confidentiality. No code, data, contract, physical structure, or infrastructure detail from the original environment is reproduced. The requirements analysis logic, specification patterns, functional decisions, and concepts required to demonstrate the professional approach have been preserved.

## 1. Context

This case study represents the evolutionary maintenance of a **legacy Oracle Materialized View** used to consolidate account, balance, entity, and business-reference information originating from different relational structures.

The original Materialized View already existed and **was not created by me**. My work focused on evolutionary changes to the existing object, including restoring information removed during a previous refactoring, adapting the query to two historical relationship models, and correcting duplicate rows caused by the cardinality of legacy data.

## 2. My contribution

The changes under my responsibility covered four main areas:

- restoring a grouping attribute that was no longer returned after an earlier change;
- recovering program and process references for records belonging to different historical periods;
- refactoring the query used to resolve those relationships;
- correcting duplicate legacy records where multiple business references could use the same account.

The main challenge was that the relationship model had changed over time. More recent records had a direct relationship between an account and its business reference, while legacy records required reconstructing that relationship through an intermediate processing chain.

## 3. Technical problem

A single JOIN strategy could not correctly represent the complete history.

For records in the current model, the reference could be obtained directly from the account. For legacy records, the association had to traverse intermediate entities before reaching the original reference.

In addition, in the legacy model, **the same account could be associated with multiple references**. A direct JOIN over this 1:N relationship multiplied rows in the final query and therefore duplicated balance information.

The actual problem was not to remove duplicated rows at the end of the query, but to correct the cardinality before joining the legacy result to the main dataset.

## 4. Adopted strategy

The query was conceptually divided into two paths using CTEs:

```text
                    ┌─ Current model ─────── direct relationship through account
Account + Balance ──┤
                    └─ Legacy model ──────── reconstruction through intermediate relations
                                              ↓
                                      consolidation by account
                                              ↓
                                    final result without multiplication
```

### Current path

For data belonging to the newer model, the direct relationship is preserved because a key exists that can associate the account with its corresponding business reference without historical reconstruction.

### Legacy path

For data predating the model change, references are located through intermediate relationships. Because multiple references may lead to the same account, they are first deduplicated and consolidated using normalized account information.

Only after this consolidation is the result joined to the main query.

## 5. Core decision — aggregate before joining

A simplified approach such as the following is problematic when the relationship is 1:N:

```sql
SELECT ...
FROM account a
JOIN legacy_reference r
  ON r.account_id = a.account_id;
```

If an account has three historical references, its balance may also appear three times.

The equivalent public solution introduces an intermediate aggregation step:

```sql
WITH legacy_reference AS (
    SELECT
        LISTAGG(x.reference_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS reference_numbers,
        LISTAGG(x.process_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS process_numbers,
        x.bank_code,
        x.branch_number,
        x.account_number
    FROM (
        SELECT DISTINCT
            r.reference_number,
            r.process_number,
            p.bank_code,
            LTRIM(p.branch_number, '0 ')  AS branch_number,
            LTRIM(p.account_number, '0 ') AS account_number
        FROM legacy_process p
        JOIN legacy_process_item i
          ON i.process_id = p.process_id
        JOIN business_reference r
          ON r.reference_id = i.reference_id
        WHERE p.reference_year <= :legacy_year
    ) x
    GROUP BY
        x.bank_code,
        x.branch_number,
        x.account_number
)
SELECT ...
FROM account_balance ab
LEFT JOIN legacy_reference lr
       ON lr.bank_code = ab.bank_code
      AND lr.branch_number = LTRIM(ab.branch_number, '0 ')
      AND lr.account_number = LTRIM(ab.account_number, '0 ');
```

This transforms multiple legitimate references into **one row per account before joining to the balance dataset**.

## 6. SQL techniques applied

### CTE — Common Table Expressions

Current and legacy strategies are isolated into independent logical blocks, reducing complexity in the main SELECT and improving maintainability and troubleshooting.

### `DISTINCT` before aggregation

Deduplication occurs before `LISTAGG`, preventing the same reference from being repeated inside the consolidated list.

### `LISTAGG`

Represents multiple legitimate references associated with the same account without multiplying the primary row.

### `COALESCE`

In the consolidated result, it prioritizes the reference obtained from the current model and falls back to the legacy strategy when a direct relationship is unavailable.

### Normalization with `LTRIM`

Legacy banking attributes may contain zero or space padding. Normalization is consistently applied to both sides of the relationship to prevent mismatches caused only by textual representation.

### Null handling

Functions such as `NVL` are conceptually used to ensure that null components of a consolidated balance do not invalidate the total calculation.

### Conditional rules

`CASE` and Oracle conditional functions can derive presentation attributes based on entity characteristics or account classifications.

## 7. Anonymized example of the final pattern

```sql
WITH current_reference AS (
    SELECT
        r.reference_id,
        TO_CHAR(r.reference_number) AS reference_number,
        TO_CHAR(r.process_number)   AS process_number,
        a.account_id
    FROM account a
    JOIN business_reference r
      ON r.reference_id = a.reference_id
    WHERE r.reference_year >= :current_model_year
),
legacy_reference AS (
    SELECT
        LISTAGG(x.reference_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS reference_number,
        LISTAGG(x.process_number, ' | ')
            WITHIN GROUP (ORDER BY x.reference_number) AS process_number,
        x.bank_code,
        x.branch_number,
        x.account_number
    FROM (
        SELECT DISTINCT
            TO_CHAR(r.reference_number) AS reference_number,
            TO_CHAR(r.process_number)   AS process_number,
            p.bank_code,
            LTRIM(p.branch_number, '0 ')  AS branch_number,
            LTRIM(p.account_number, '0 ') AS account_number
        FROM legacy_process p
        JOIN legacy_process_item i
          ON i.process_id = p.process_id
        JOIN business_reference r
          ON r.reference_id = i.reference_id
        WHERE r.reference_year <= :legacy_year
    ) x
    GROUP BY
        x.bank_code,
        x.branch_number,
        x.account_number
)
SELECT
    ab.account_id,
    COALESCE(cr.reference_number, lr.reference_number) AS reference_number,
    COALESCE(cr.process_number, lr.process_number)     AS process_number,
    NVL(ab.checking_balance, 0)
      + NVL(ab.savings_balance, 0)
      + NVL(ab.investment_balance, 0)                 AS total_balance
FROM account_balance ab
LEFT JOIN current_reference cr
       ON cr.account_id = ab.account_id
LEFT JOIN legacy_reference lr
       ON lr.bank_code = ab.bank_code
      AND lr.branch_number = LTRIM(ab.branch_number, '0 ')
      AND lr.account_number = LTRIM(ab.account_number, '0 ')
WHERE COALESCE(cr.reference_number, lr.reference_number) IS NOT NULL;
```

The SQL above is an **anonymized educational reconstruction**. Names, structures, and relationships have been changed and simplified; it does not represent the physical model of the original professional environment.

## 8. Outcome

The evolution enabled the query to:

- return business references for data from both historical models;
- preserve multiple legitimate references associated with the same account;
- prevent unintended multiplication of balance rows;
- maintain a single consolidated account representation in the result;
- restore functional information that had stopped being returned;
- make the separation between current and legacy rules more explicit for future maintenance.

## 9. Skills demonstrated

- Oracle SQL;
- legacy code maintenance;
- 1:N cardinality analysis;
- diagnosis of JOIN-induced duplication;
- CTEs;
- `INNER JOIN` and `LEFT JOIN`;
- `LISTAGG` and `GROUP BY`;
- `DISTINCT`;
- `COALESCE` and `NVL`;
- data normalization for relational matching;
- handling different historical data models;
- business-rule-driven refactoring;
- impact analysis in consolidated queries.

## 10. Authorship and transparency

The Materialized View that originated this study was already part of an existing solution and was originally developed by other professionals. This case study **does not claim authorship of the original object**.

My contribution corresponds to the evolutions, fixes, and refactorings described in this document. The published SQL example was recreated specifically for the portfolio and serves only to demonstrate the techniques and reasoning applied during those changes.
