# 25 — MySQL Traps and Dialect Differences

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Avoid high-frequency mistakes caused by nulls, clause order, outer joins, MySQL syntax, and nondeterminism.

## Syntax template

```sql
-- MySQL choices
LIMIT 10 OFFSET 20              -- not TOP
IFNULL(value, fallback)
value <=> other_value           -- null-safe equality
ORDER BY value IS NULL, value   -- nulls last ascending

-- unsupported direct syntax/patterns
-- FULL OUTER JOIN: emulate
-- QUALIFY: filter in outer query/CTE
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `NULL = NULL` | `NULL`, not true. | Use `IS NULL` or `<=>`. |
| `NOT IN` with null candidate | Often `NULL`/not true. | Prefer `NOT EXISTS`. |
| Outer-join unmatched row | Preserved side plus null-extended optional side. | A `WHERE` predicate on optional side may remove it. |
| `COUNT(col)` | Counts known values. | Ignores nulls; `COUNT(*)` counts rows. |
| `CONCAT` | Joined string. | Any null argument → `NULL`; `CONCAT_WS` skips null arguments. |
| `LAST_VALUE` | Last value in current frame. | Default frame often does not reach partition end. |
| No final `ORDER BY` | Unspecified order. | Indexes/current plan do not guarantee ordering. |

## Revision notes

- MySQL supports `LIMIT`, not SQL Server `TOP`; `IFNULL`, not SQL Server `ISNULL`.
- MySQL 8.4 supports `INTERSECT`/`EXCEPT`, but older installations may not.
- MySQL lacks direct `FULL OUTER JOIN`, `QUALIFY`, and ordinary materialized views.
- `ONLY_FULL_GROUP_BY` exposes nondeterministic grouping that permissive modes might accept.
- Alias visibility follows processing order: select aliases are unavailable to `WHERE`.
- Use `<=>` to compare nullable values as values, but do not indiscriminately make null keys join each other.

## Examples

### The `NOT IN` surprise

```sql
SELECT 3 NOT IN (1, 2, NULL) AS result;
```

**Expected behavior:** Returns `NULL`, not `1`; a `WHERE` using it rejects the row.

### Preserve left rows

```sql
SELECT d.department_id, e.employee_id
FROM departments AS d
LEFT JOIN employees AS e
  ON e.department_id = d.department_id
 AND e.active = 1;
```

**Expected behavior:** All departments survive; inactive employees simply do not match.

## Tricky parts

- Every item in this chapter is a trap—practice explaining the reason, not memorizing only the fix.
- Assuming behavior from PostgreSQL, SQL Server, or Oracle applies unchanged.
- Ignoring server version and SQL mode.
- Equating an empty result with a result containing one null row.

## Interview checks

1. Name five MySQL-versus-other-dialect differences.
2. Empty result versus scalar subquery returning null?
3. Why can moving a predicate from `ON` to `WHERE` change output?

## 30-second recap

- Avoid high-frequency mistakes caused by nulls, clause order, outer joins, MySQL syntax, and nondeterminism.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — MySQL Traps and Dialect Differences](https://dev.mysql.com/doc/refman/8.4/en/differences-from-ansi.html)
