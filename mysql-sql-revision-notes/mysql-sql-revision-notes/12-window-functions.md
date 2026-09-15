# 12 — Window Functions

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Compute rankings, offsets, running measures, and per-group analytics without collapsing rows.

## Syntax template

```sql
window_function(args) OVER (
  [PARTITION BY expr, ...]
  [ORDER BY expr [ASC|DESC], ...]
  [ROWS|RANGE frame_extent]
)
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `ROW_NUMBER()` | Unique sequential number per partition. | Null sort keys follow MySQL ordering; add tie-breakers. |
| `RANK()` | Same rank for peers; leaves gaps. | Null peers tie when sort keys compare equal. |
| `DENSE_RANK()` | Same rank for peers; no gaps. | Null peers tie. |
| `LAG` / `LEAD` | Value from offset row. | No such row → default, which is `NULL` unless supplied; existing null remains null. |
| `SUM` / `AVG OVER` | Window aggregate per row. | Ignore null expression values; all-null frame → `NULL`. |
| `COUNT(expr) OVER` | Non-null values in frame. | Empty/all-null frame → `0`. |
| `FIRST_VALUE` / `LAST_VALUE` | Value in current frame. | Can return null; MySQL effectively respects nulls. |

## Revision notes

- Window functions retain input-row granularity; `GROUP BY` collapses groups.
- They are allowed in the select list and `ORDER BY`, not directly in `WHERE`, `GROUP BY`, or `HAVING`. Filter in an outer query/CTE.
- Window `ORDER BY` controls calculation order; final `ORDER BY` controls display order.
- With window `ORDER BY` and no explicit frame, aggregate value functions commonly use a peer-aware running frame. Write the frame explicitly.
- Use `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` for a row-by-row running total.
- `LAST_VALUE` often surprises because the default frame ends at the current peer group. Use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` for the partition last value.
- MySQL has no `QUALIFY`; use a CTE or derived table.

## Examples

### Top three per department

```sql
WITH ranked AS (
  SELECT e.*,
         ROW_NUMBER() OVER (
           PARTITION BY department_id
           ORDER BY salary DESC, employee_id
         ) AS rn
  FROM employees AS e
)
SELECT * FROM ranked WHERE rn <= 3;
```

**Expected behavior:** At most three deterministic employees per department. Null salaries rank after known salaries in descending order.

### Running total

```sql
SELECT order_id, ordered_at, amount,
       SUM(amount) OVER (
         ORDER BY ordered_at, order_id
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_amount
FROM orders;
```

**Expected behavior:** One row per order; `SUM` skips a null amount and keeps the prior running sum.

## Tricky parts

- Filtering a window alias in the same query block.
- Omitting a deterministic tie-breaker.
- Confusing `RANK` with `DENSE_RANK`.
- Forgetting an explicit frame for `LAST_VALUE` or running totals.
- Assuming calculation order also sorts final output.

## Interview checks

1. Top N per group?
2. Explain `ROWS` versus `RANGE`.
3. Why can `LAST_VALUE` return the current row?

## 30-second recap

- Compute rankings, offsets, running measures, and per-group analytics without collapsing rows.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Window Functions](https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html)
