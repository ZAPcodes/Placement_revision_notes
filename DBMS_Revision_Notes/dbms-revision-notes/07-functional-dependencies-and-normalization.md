# 07. Functional Dependencies and Normalization

> Normalization uses dependencies to reduce update anomalies while preserving correct reconstruction and important constraints.

## Functional dependency (FD)

`X -> Y` means: whenever two valid tuples agree on attributes X, they must agree on Y.

An FD is a semantic rule about all valid states, not merely a coincidence in current sample rows.

## FD types

- **Trivial:** `Y` is a subset of `X`.
- **Non-trivial:** `Y` contains something outside `X`.
- **Full dependency:** Y depends on all of composite X; removing any part breaks dependency.
- **Partial dependency:** Y depends on a proper subset of a candidate key.
- **Transitive dependency:** key determines X and X determines a non-key Y.

## Armstrong’s axioms

1. Reflexivity: if `Y ⊆ X`, then `X -> Y`.
2. Augmentation: if `X -> Y`, then `XZ -> YZ`.
3. Transitivity: if `X -> Y` and `Y -> Z`, then `X -> Z`.

Useful derived rules: union, decomposition and pseudotransitivity.

## Attribute closure

`X+` is every attribute functionally determined by X under an FD set.

Uses:

- Test whether X is a superkey.
- Test whether an FD is implied.
- Find candidate keys.
- Validate decomposition reasoning.

Candidate key means **minimal by set inclusion**, not necessarily the key with globally fewest columns.

## Minimal/canonical cover

A compact equivalent FD set generally has:

- One attribute on each right side.
- No extraneous left-side attributes.
- No redundant dependencies.

Order of reduction can produce different but equivalent minimal covers.

## Anomalies from redundancy

- **Update anomaly:** fact repeated in multiple rows must change consistently.
- **Insertion anomaly:** cannot record one fact without unrelated fact.
- **Deletion anomaly:** removing one fact accidentally removes another.

Normalization separates independently changing facts.

## Normal forms

### 1NF

Attributes contain atomic values for the chosen relational design; no repeating groups/multivalued fields in one cell.

### 2NF

In 1NF and every non-prime attribute is fully dependent on every candidate key—no partial dependency on part of a composite candidate key. If every candidate key is single-column, 2NF violations via partial key dependency cannot occur.

### 3NF

For every non-trivial FD `X -> A`, either:

- X is a superkey, or
- A is prime (belongs to some candidate key).

The informal “no transitive dependency” rule is useful but less precise.

### BCNF

For every non-trivial FD `X -> Y`, X must be a superkey.

BCNF is stricter than 3NF. A relation can satisfy 3NF but violate BCNF when the dependent attribute is prime.

### 4NF

For every non-trivial multivalued dependency `X ->> Y`, X should be a superkey. Handles independent multivalued facts.

### 5NF

Addresses non-trivial join dependencies not implied by candidate keys. Rare in ordinary interviews; know purpose, not lengthy procedures.

## Decomposition properties

### Lossless join

Joining decomposed relations recreates exactly the original valid relation—no spurious tuples and no lost tuples.

For binary decomposition of R into R1 and R2, a common FD-based test is whether intersection functionally determines R1 or R2.

### Dependency preservation

All original constraints can be enforced by checking individual decomposed relations without joining them.

- 3NF synthesis can provide losslessness and dependency preservation.
- BCNF decomposition is lossless but may lose dependency preservation.

## Denormalization

Intentional redundancy for measured performance/access needs. Requires a consistency strategy through transactions, generated data, refresh pipelines or controlled ownership.

Do not denormalize merely to avoid learning joins.

## Common traps

- FDs come from business semantics, not current rows alone.
- Prime attribute belongs to any candidate key, not only the primary key.
- 2NF matters only when a non-prime attribute depends on part of a composite candidate key.
- 3NF formal definition has the prime-attribute exception; BCNF does not.
- Lossless join and dependency preservation are independent goals.
- A surrogate key can make a table appear key-normalized while business dependencies/redundancy remain.
- More normalization can increase joins and reduce locality; it is not an unconditional performance win.

## Interview checks

1. What does `X -> Y` mean semantically?
2. Superkey vs candidate key?
3. Full vs partial dependency?
4. Explain 1NF, 2NF, 3NF and BCNF precisely.
5. Give the formal 3NF vs BCNF difference.
6. Lossless join vs dependency preservation?
7. Why can BCNF lose dependency preservation?
8. What problem does 4NF solve?
9. When is denormalization justified?
10. Why does adding an ID column not automatically fix poor normalization?

## 60-second recall

- FD is a rule across all valid states.
- Closure tests implied attributes/superkeys.
- Candidate keys are minimal superkeys.
- 2NF removes partial non-prime dependencies; 3NF handles transitive-style dependencies; BCNF requires every determinant to be a superkey.
- Decomposition should be lossless and preferably dependency-preserving.
- Denormalize only with a measured reason and consistency owner.

