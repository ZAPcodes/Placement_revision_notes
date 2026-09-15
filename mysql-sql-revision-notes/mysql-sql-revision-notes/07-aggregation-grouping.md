# 07 — Aggregate Functions and Grouping

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Summarize rows correctly, distinguish pre/post-group filters, and understand empty/all-null groups.

## Syntax template

```sql
SELECT group_col,
       COUNT(*) AS rows_count,
       COUNT(value_col) AS known_count,
       SUM(value_col) AS total
FROM table_name
WHERE row_condition
GROUP BY group_col
HAVING group_condition;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `COUNT(*)` | Number of rows; empty input → `0`. | Counts rows even when every column is null. |
| `COUNT(expr)` | Non-null expression count; empty input → `0`. | Ignores nulls. |
| `SUM` / `AVG` | Aggregate value. | Ignore nulls; no non-null values → `NULL`. |
| `MIN` / `MAX` | Smallest/largest known value. | Ignore nulls; no non-null values → `NULL`. |
| `GROUP_CONCAT` | Concatenated non-null values. | No non-null values → `NULL`. |
| `GROUP BY` | One row per group. | All null grouping keys form one group. |

## Revision notes

- `WHERE` filters individual rows before grouping; `HAVING` filters completed groups.
- `COUNT(*) - COUNT(col)` counts nulls in `col`.
- Conditional count: `SUM(condition)` works in MySQL but returns `NULL` on empty input; `COUNT(CASE WHEN condition THEN 1 END)` returns `0`.
- `COUNT(DISTINCT a,b)` counts distinct non-null pairs and ignores a row if either expression is null.
- `WITH ROLLUP` adds super-aggregate rows. Use `GROUPING(col)` to distinguish a rollup null from a data null.
- With `ONLY_FULL_GROUP_BY`, every unaggregated selected expression must be grouped or functionally dependent.

## Examples

### Null audit

```sql
SELECT COUNT(*) AS total_rows,
       COUNT(manager_id) AS known_managers,
       COUNT(*) - COUNT(manager_id) AS missing_managers
FROM employees;
```

**Expected behavior:** Always one row; counts are zero on an empty table.

### Conditional aggregation

```sql
SELECT department_id,
       SUM(salary >= 100000) AS high_paid,
       AVG(salary) AS avg_known_salary
FROM employees
GROUP BY department_id;
```

**Expected behavior:** `AVG` ignores null salaries; `SUM` counts TRUE values within each nonempty group.

## Tricky parts

- Selecting non-grouped columns under lax SQL mode.
- Using `WHERE COUNT(*) > 5`.
- Expecting `SUM` on no rows to return zero.
- Using `COUNT(col)` when row count was intended.
- Confusing a rollup-generated null with stored null.

## Interview checks

1. How do `COUNT(*)` and `COUNT(col)` differ?
2. What does `AVG` do with nulls?
3. Find groups with more than five members.

## 30-second recap

- Summarize rows correctly, distinguish pre/post-group filters, and understand empty/all-null groups.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Aggregate Functions and Grouping](https://dev.mysql.com/doc/refman/8.4/en/aggregate-functions-and-modifiers.html)
