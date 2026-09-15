# 20 — Query Performance and Optimization

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Diagnose plans and write predicates, joins, grouping, and pagination that indexes can support.

## Syntax template

```sql
EXPLAIN FORMAT=TREE
SELECT ...;

EXPLAIN ANALYZE
SELECT ...;

SHOW INDEX FROM table_name;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `EXPLAIN` | Optimizer plan estimate; query normally is not executed. | Plan explains null predicates like any others. |
| `EXPLAIN ANALYZE` | Executes statement and reports actual timing/rows. | Do not use casually on costly or modifying statements. |
| `key` | Chosen index, or `NULL` if none. | Here null means no selected index, not SQL data null. |
| `rows` | Estimated examined rows. | Estimate can be inaccurate. |
| `Using index` | Covering-index access. | Not the same as merely using an index lookup. |
| `Using filesort` | Extra sorting algorithm. | May be in memory; does not necessarily mean disk file. |

## Revision notes

- Sargable: `ordered_at >= '2026-01-01'`; often non-sargable: `YEAR(ordered_at)=2026` on a plain index.
- Match composite-index order to equality predicates first, then useful range/order columns, validated against real workload.
- Selectivity, table size, statistics, and returned fraction determine whether an index is beneficial.
- `EXISTS` versus `IN` has no universal winner; modern optimizers transform both. Measure.
- `UNION ALL` can outperform `OR` in some plans, but it can change duplicate semantics.
- Optimize correctness and row counts first; then inspect execution plans and measure.

## Examples

### Sargable year filter

```sql
WHERE ordered_at >= '2026-01-01'
  AND ordered_at <  '2027-01-01'
```

**Expected behavior:** Can use a range on a plain index on `ordered_at`.

### Avoid deep offset

```sql
WHERE (ordered_at, order_id) < (?, ?)
ORDER BY ordered_at DESC, order_id DESC
LIMIT 50
```

**Expected behavior:** Keyset page cost does not grow linearly with page number.

## Tricky parts

- Treating every full scan as bad.
- Reading `possible_keys` as the chosen key.
- Adding redundant indexes.
- Optimizing without representative data.
- Ignoring semantic differences during rewrites.

## Interview checks

1. What makes a predicate sargable?
2. Interpret `Using index` versus `Using filesort`.
3. Why can an optimizer ignore an available index?

## 30-second recap

- Diagnose plans and write predicates, joins, grouping, and pagination that indexes can support.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Query Performance and Optimization](https://dev.mysql.com/doc/refman/8.4/en/optimization.html)
