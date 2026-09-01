# Case Study — Production Incident: Distributed Transaction Reconciliation After Migration

> **Confidentiality note:** this case study is an anonymized and abstracted version of a specification produced in a real professional context. System names, organizations, entities, identifiers, profiles, integrations, API contracts, endpoints, routes, database structures and objects, payloads, documents, values, status codes, paths, personal data, and other technical or operational details have been removed, changed, or generalized to preserve confidentiality. No code, data, contract, physical structure, or infrastructure detail from the original environment is reproduced. The requirements analysis logic, specification patterns, functional decisions, and concepts needed to demonstrate the professional approach have been preserved.

## 1. Context

This case study describes a real production incident identified during the operation of a critical application integrated with an external financial platform.

After an application migration, an operation was sent to the external system and processed successfully, but the corresponding state was not persisted correctly in the local application.

This created a divergence between two sources of truth:

```text
Local application
      │
      │ sends operation
      ▼
External financial system
      │
      ├── operation successfully created
      │
      ▼
response to application
      │
      └── incomplete local persistence
```

From the external system's perspective, the operation already existed. From the local application's perspective, it could still appear incomplete.

This type of situation requires caution because repeating an operation without reconciling the states first can create duplicates or require later correction/cancellation procedures.

## 2. Incident detection

In addition to product and functional-analysis responsibilities, I also participate in the operational execution of the process supported by the application.

I identified the inconsistent behavior directly while performing that operational work.

The first validation was not to assume that the integration had failed. The priority was to determine the actual state in the external system.

Using authorized institutional access to the external financial platform, I confirmed that the document had in fact been created.

That changed the direction of the investigation:

```text
Possible initial hypothesis
"The external operation failed"

        ↓ external verification

Evidence found
"The external operation succeeded"

        ↓

New investigation focus
"Why did the local application not reflect the success?"
```

## 3. My role

My participation in the incident included:

- detecting the issue during operational execution;
- independently confirming the external operation state;
- starting the investigation directly in the application's database;
- analyzing the records expected to represent the external response;
- comparing the confirmed external state with the locally persisted state;
- preparing and executing a controlled reconciliation directly in the production database;
- performing functional validation after reconciliation;
- subsequently engaging the development team to permanently correct the migrated application.

AI tools were used as support during part of the analysis and documentation process. The investigation direction, domain knowledge, relationship interpretation, operational decision-making, and execution of the reconciliation remained my responsibility.

## 4. Diagnosis: external success does not mean end-to-end success

One of the central points of this incident was separating two conditions that may appear equivalent but are not:

```text
External call success
        ≠
End-to-end transaction success
```

An operation is fully reconciled only when the local system correctly represents the result confirmed by the external platform.

Conceptually:

```text
[1] Local request
        ↓
[2] External call
        ↓
[3] External processing
        ↓
[4] Response received
        ↓
[5] Response mapping
        ↓
[6] Local persistence
        ↓
[7] Functional state shown to the user
```

In this incident, there was evidence of successful external processing, while the final local state remained inconsistent.

## 5. Evidence-driven investigation

Before changing any data, the investigation needed to answer objective questions:

1. Does the operation actually exist in the external system?
2. Does the local record being analyzed unequivocally correspond to that external operation?
3. Is another local record already using the same external identifier?
4. Which attributes should have been updated after the response?
5. Are request-control or relationship records associated with the transaction present?
6. Can reconciliation be performed without modifying functional values that are already correct?

A financial amount alone, for example, is not sufficient evidence for correlating records when different operations may contain identical amounts.

Reconciliation should use a combination of functional and technical attributes capable of uniquely identifying the intended record.

## 6. Why simply retrying was risky

Once the external operation had been confirmed, resending the request was no longer a safe recovery strategy.

The conceptual scenario was:

```text
First attempt
Application ───────► External system
                          │
                          └── Document A created

Application fails to persist the response

Retry without reconciliation
Application ───────► External system
                          │
                          └── risk of another operation
```

In integrations that mutate external state, a timeout, local error, or persistence failure **does not prove that the external operation failed**.

The adopted strategy was therefore:

```text
Do not resend
      ↓
Check external state
      ↓
Correlate with local state
      ↓
Reconcile only what is proven
```

## 7. Controlled database reconciliation

Because the external document had already been confirmed, the local records required reconciliation.

The correction was performed directly in the production database using a defensive approach, with validations before the change and the ability to roll back before final confirmation.

The following example is entirely fictional and demonstrates only the pattern used:

```sql
SAVEPOINT BEFORE_RECONCILIATION;

DECLARE
    v_expected_rows  NUMBER := :expected_rows;
    v_valid_rows     NUMBER;
    v_updated_rows   NUMBER;
BEGIN
    SELECT COUNT(*)
      INTO v_valid_rows
      FROM local_transaction t
     WHERE t.internal_id = :internal_id
       AND t.reference_year = :reference_year
       AND t.external_document IS NULL
       AND t.external_created_at IS NULL;

    IF v_valid_rows <> v_expected_rows THEN
        RAISE_APPLICATION_ERROR(
            -20001,
            'Reconciliation stopped: unexpected validation result.'
        );
    END IF;

    UPDATE local_transaction
       SET external_document   = :confirmed_external_document,
           external_sent_at    = :confirmed_date,
           external_created_at = :confirmed_date
     WHERE internal_id = :internal_id
       AND external_document IS NULL
       AND external_created_at IS NULL;

    v_updated_rows := SQL%ROWCOUNT;

    IF v_updated_rows <> v_expected_rows THEN
        RAISE_APPLICATION_ERROR(
            -20002,
            'Reconciliation stopped: unexpected number of updated rows.'
        );
    END IF;
END;
/
```

This example does not reproduce the script used in the original environment.

## 8. Why the correction was defensive

The goal was not simply to "fill null fields."

A manual production intervention needed to prevent an already reconciled record from being overwritten or a different operation from being modified accidentally.

The approach therefore used:

- validation of the expected row count before modification;
- filters based on the current record state;
- correlation with previously verified identifiers;
- `SQL%ROWCOUNT` verification;
- a `SAVEPOINT` before modification;
- post-change validation;
- `COMMIT` only after verification;
- the ability to `ROLLBACK` if any inconsistency appeared.

Conceptually:

```text
External evidence
      ↓
Local validation
      ↓
Preconditions satisfied?
   ┌───────┴───────┐
   NO              YES
   │                │
 Abort          Reconcile
                    ↓
             Validate result
               ┌────┴────┐
              ERROR      OK
               │          │
            Rollback    Commit
```

## 9. Post-reconciliation validation

Validation after the intervention was not limited to the SQL command succeeding.

Both the record state and the application's functional behavior were checked.

Validation included:

- number of reconciled records matching the expected quantity;
- correct association of the external identifier with the local record;
- absence of duplicate external documents;
- preservation of functional values outside the scope of the correction;
- consistency of dates associated with the external response;
- presentation of the document in the expected application state.

After manual reconciliation, the application correctly recognized the affected documents.

## 10. Data recovery is not root-cause remediation

Reconciliation restored operational consistency, but it did not mean that the application itself had been fixed.

The incident appeared after an application migration. Equivalent behavior worked in the previous implementation, while the migrated version failed to persist part of the state returned by the integration.

The functional and data investigation established:

```text
external operation completed
        +
incomplete local state
        +
behavior introduced after migration
```

This evidence was sufficient to direct the development team, which was engaged **after the diagnosis**, to compare the migrated implementation against the previously functioning behavior and correct the application flow.

This case study does not claim a more specific implementation-level root cause — such as a transaction, repository, constraint, or mapping defect — because the investigation established the functional failure point but did not publicly isolate a more specific code-level mechanism.

## 11. Technical areas for code-level investigation

After establishing the functional problem, an application-level investigation may examine areas such as:

```text
External response
      ↓
Deserialization / mapping
      ↓
Update rule
      ↓
Repository / persistence
      ↓
Transaction
      ↓
Local commit
```

Other areas to consider include:

- rollback after external success;
- constraint errors;
- migration/configuration differences;
- failure to create auxiliary relationships;
- logs between external acceptance and local commit;
- exceptions being handled without adequate propagation;
- behavioral differences between legacy and migrated implementations.

These are investigation targets and **are not presented as confirmed causes of this incident**.

## 12. Idempotency and reconciliation

The incident reinforced an important principle of distributed integrations:

> Missing local confirmation must not automatically be interpreted as missing external execution.

A robust solution should consider mechanisms such as:

- correlation identifiers;
- idempotency keys when supported;
- external-state verification before retrying;
- duplicate prevention;
- auditable request and response records;
- explicit reconciliation strategies;
- handling of external-success/local-failure scenarios.

Conceptually:

```text
           ┌── known state ─────► continue flow
Retry? ────┤
           └── uncertain state ─► query before resend
```

## 13. Separation of responsibilities during the incident

The initial diagnosis was not performed by the development or infrastructure teams.

My work covered detection, external confirmation, data investigation, inconsistency identification, reconciliation, and functional validation.

The development team was engaged afterward to address the application and prevent recurrence of the behavior introduced after migration.

This separation was important because the incident contained two different needs:

```text
Immediate operational recovery
            +
Permanent application correction
```

The first restored consistency for the affected records. The second addressed recurrence of the defect.

## 14. Skills demonstrated

- incident investigation;
- production support;
- Oracle SQL;
- system integration analysis;
- data reconciliation;
- evidence-driven troubleshooting;
- distributed-state analysis;
- migrated-application troubleshooting;
- production data validation;
- defensive SQL for controlled corrections;
- `SAVEPOINT`, `ROLLBACK`, and `COMMIT`;
- `SQL%ROWCOUNT` validation;
- duplicate prevention;
- idempotency concepts;
- operational risk analysis;
- domain knowledge applied to technical investigation;
- communication across operations, product, and development;
- distinction between operational recovery and permanent remediation.

## 15. Lessons learned

This incident reinforced principles that apply far beyond the original application:

1. **A local error does not mean an external failure.**
2. **Before retrying a state-changing operation, determine the actual state of the external system.**
3. **Manual production corrections require preconditions, validation, and rollback capability.**
4. **Domain knowledge can dramatically reduce the technical investigation space.**
5. **Restoring data resolves the incident; correcting the application prevents recurrence.**
6. **In distributed systems, intermediate states and partial failures must be treated as expected architectural scenarios.**

## 16. Authorship and transparency

This case study describes a real professional incident, but all published technical content has been reconstructed and anonymized.

The operational investigation, data analysis, manual reconciliation, and validation were conducted by me. The development team was engaged afterward to correct the behavior of the migrated application.

AI tools contributed as support for analysis and documentation structuring, using technical context and documentation previously organized by me. No proprietary code, physical database structure, endpoint, payload, identifier, real financial data, or infrastructure detail from the original environment is reproduced in this portfolio.
