# 04. Relational Algebra and Calculus

> Relational algebra explains how relations can be transformed; relational calculus describes which tuples satisfy a condition.

## Algebra vs calculus vs SQL

| Relational algebra | Relational calculus | SQL |
|---|---|---|
| Procedural expression of operations | Declarative logical specification | Declarative practical language |
| Basis for logical query plans | Foundation for query meaning | Bag semantics, NULLs and extensions |
| Set-oriented | Predicate-oriented | Optimizer chooses physical execution |

## Fundamental relational-algebra operators

| Operator | Meaning | Important requirement |
|---|---|---|
| Selection `σ` | Filter rows | Predicate over attributes |
| Projection `π` | Choose/derive columns | Pure algebra removes duplicates |
| Union `∪` | Tuples in either relation | Union-compatible schemas |
| Difference `−` | Tuples in first not second | Union-compatible schemas |
| Cartesian product `×` | All tuple pairs | Attribute names must be distinguishable |
| Rename `ρ` | Rename relation/attributes | Useful for self-joins |

Derived operators include intersection, joins, division and aggregation extensions.

## Selection vs projection

- Selection reduces **rows**.
- Projection reduces **columns**.
- Selection predicates can often be pushed near base relations to reduce intermediate rows.
- Projection can be pushed down while retaining columns required by later joins/predicates.

## Joins

- **Theta join:** product filtered by any comparison predicate.
- **Equi-join:** theta join using equality.
- **Natural join:** automatically equates all same-named attributes and removes duplicate join columns.
- **Outer join:** preserves unmatched rows with NULL padding.
- **Semi-join:** returns only matching tuples from one side.
- **Anti-join:** returns tuples from one side with no match.

Natural join is concise but fragile when schemas gain same-named columns that were not intended as join keys. Production SQL should usually state explicit join conditions.

## Division: “for all” queries

Division models queries such as:

> Find students who completed every required course.

SQL often expresses this through double `NOT EXISTS`, grouping with counts, or relational-division patterns. Carefully handle duplicates and NULLs.

## Set compatibility

Union/intersection/difference require corresponding attributes to have compatible domains. Attribute names may require renaming.

SQL distinctions:

- `UNION` removes duplicates.
- `UNION ALL` retains duplicates and is usually cheaper.
- SQL tables/results have no guaranteed order without `ORDER BY`.

## Query equivalence and rewriting

Common logical transformations:

- Push selections below joins when predicates reference one input.
- Push projections while retaining required attributes.
- Reorder inner joins because they are logically associative/commutative under appropriate conditions.
- Replace product + equality selection with an equi-join.
- Simplify redundant predicates.

Outer joins are not freely reorderable like inner joins because unmatched-row preservation changes semantics.

## Tuple relational calculus (TRC)

Describes result tuples using predicates over tuple variables:

> Return tuples `t` such that condition `P(t)` holds.

## Domain relational calculus (DRC)

Uses variables ranging over individual attribute-domain values rather than whole tuples.

## Free, bound and safe expressions

- **Free variable:** contributes to output.
- **Bound variable:** quantified inside formula.
- **Safe/range-restricted query:** returns values derived from the finite database domain rather than an unbounded universe.

Safety is essential for implementable finite results.

## Set vs bag semantics

Pure relational algebra normally treats relations as sets. SQL often preserves duplicates because duplicate elimination requires work and duplicates can be meaningful. This affects projection, joins, aggregation and equivalence rules.

## NULL complication

Classical relational theory assumes ordinary values and two-valued predicates. SQL adds NULL and three-valued logic, so theoretical equivalences can require care in real SQL.

## Common traps

- Projection in relational algebra removes duplicates; SQL `SELECT` does not unless `DISTINCT`.
- Natural join may match every same-named column.
- `R − S` is directional.
- Cartesian product itself has no join condition.
- Outer-join predicates pushed from `WHERE` to `ON` can change results.
- Inner joins can often reorder; outer joins cannot be arbitrarily reordered.
- Relational calculus being declarative does not mean it can return an infinite unsafe result.

## Interview checks

1. Selection vs projection?
2. Theta join, equi-join and natural join?
3. Semi-join vs inner join?
4. Express a “for every” requirement conceptually.
5. Why is `UNION ALL` normally faster than `UNION`?
6. Set semantics vs SQL bag semantics?
7. Why can selections be pushed down?
8. Why is natural join risky?
9. Why are outer joins harder to reorder?
10. Relational algebra vs relational calculus?

## 60-second recall

- Algebra says how to transform relations; calculus states predicates results satisfy.
- Selection = rows; projection = columns.
- Join = related tuple combinations; semi/anti joins return only one side.
- Division represents “for all.”
- SQL defaults to bags and adds NULL, so pure algebra rules need care.
- Push filters/projections down; inner joins are more freely reorderable than outer joins.

