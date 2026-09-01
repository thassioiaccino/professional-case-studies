# Case Study — User and Role Management Refinement

> **Confidentiality note:** this case is an anonymized and abstracted version of a specification developed in a real professional context. Names of systems, organizations, entities, identifiers, roles, integrations, API contracts, endpoints, routes, database structures and objects, payloads, documents, values, status codes, paths, personal data, and other technical or operational details were removed, changed, or generalized to preserve confidentiality. No code, data, contract, physical structure, or infrastructure detail from the original environment is reproduced. The requirements analysis logic, specification patterns, functional decisions, and concepts required to demonstrate the professional approach were preserved.

## 1. Context

A corporate application included a user administration feature used to query users, associate access roles, and display the permissions derived from those roles.

The listing had two main problems:

- the roles field did not correctly reflect the associations for each user;
- a permissions column displayed too much information, making the listing difficult to read.

On the user edit screen, roles could be selected, but there was no clear view of the permissions resulting from those selections.

## 2. Improvement objective

As a user responsible for access administration,

**I want** the listing to show only the roles actually associated with each user and, while editing, to view the permissions corresponding to the selected roles,

**so that** access information remains consistent, the listing becomes easier to read, and permissions can be reviewed before saving.

## 3. Functional scope

The specified solution includes:

- removing the detailed permissions column from the listing;
- correcting how associated roles are displayed;
- supporting multiple roles without duplication;
- creating a read-only area for permissions derived from selected roles;
- refreshing that view when role selection changes;
- preserving the existing rule that persistence only occurs after an explicit save action;
- preserving existing authentication and authorization rules.

## 4. Selected business rules

### BR01 — Visual removal only
The permissions column is removed only from the listing. Roles, permissions, and their existing relationships remain part of the access-control model.

### BR02 — Actually associated roles
The listing must display only the roles associated with the queried user, without applying a default role when that role is not actually assigned.

### BR03 — Multiple roles
When more than one role exists, names must be displayed in a readable, standardized format without duplicates.

Fictitious example:

```text
Query; Operations; Supervision
```

### BR04 — Consistency between listing and editing
The listing and edit screen must represent the same collection of roles associated with the user.

### BR05 — Permissions derived from roles
Permissions displayed on the edit screen must be calculated from the roles currently selected.

### BR06 — Union without duplication
When the same permission is granted by two or more selected roles, it must appear only once.

### BR07 — Read only
The permissions area is informational. Permissions continue to be granted or revoked through roles rather than by direct assignment to a user.

### BR08 — Explicit persistence
Changing role selection may immediately refresh the permission view, but new associations are persisted only after the save action completes successfully.

### BR09 — Access control preserved
The improvement must not modify existing authentication, authorization, or corporate identity-service integration rules.

## 5. Selected acceptance criteria

- The listing must no longer display the detailed permissions column.
- Existing information and actions must remain available.
- A user with one role must display only that role.
- Users with multiple roles must display all roles without duplicates.
- The listing must not assign a nonexistent default role.
- Roles shown in the listing must match those loaded on the edit screen.
- Opening the edit screen must display the permissions granted by the currently associated roles.
- Selecting or deselecting a role must recalculate the displayed permissions.
- Permissions shared by multiple roles must appear only once.
- The permissions area must not allow direct editing.
- Role changes must not be persisted before the explicit save action.
- Existing query, creation, editing, status-change, and save flows must continue to work after the improvement.

## 6. Out of scope

This improvement does not include:

- changing the role catalog;
- creating or deleting permissions;
- allowing manual permission assignment directly to users;
- changing authentication rules;
- changing unrelated reports or exports.

## 7. Analysis decisions

### Separate visualization from persistence
Immediate permission refresh while selecting roles improves reviewability, but it must not represent persisted change. This reduces accidental changes and preserves the existing transactional pattern.

### Derive permissions from roles
The specification avoids creating a second mechanism for managing permissions directly by user. Roles remain the source of entitlement, reducing inconsistencies across authorization mechanisms.

### Avoid duplication
A permission may belong to multiple roles. The interface should present the union of those permissions so duplicated entries do not reduce readability or misrepresent effective access.

## 8. Skills demonstrated

This case demonstrates:

- requirements elicitation and refinement;
- analysis of existing behavior;
- scope and out-of-scope definition;
- business-rule specification;
- testable acceptance criteria;
- regression analysis;
- role-based access-control modeling;
- consistency across application views;
- translation of business needs into implementable technical behavior.
