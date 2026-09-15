# 14 — Updating and Deleting Data

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Modify exactly the intended rows, including join-based changes and safe duplicate removal.

## Syntax template

```sql
UPDATE table_name
SET c1 = expression, c2 = expression
WHERE condition
ORDER BY ...
LIMIT n;

DELETE FROM table_name
WHERE condition
ORDER BY ...
LIMIT n;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `UPDATE` | Matched rows are considered; returns affected-row metadata to client. | Setting null fails for `NOT NULL` in strict mode. |
| Single-table assignments | Generally evaluated left-to-right in MySQL. | Later assignments can see an earlier null assignment. |
| `DELETE` | Removes matching rows. | Predicate UNKNOWN does not match. |
| Join update/delete | Changes rows selected by joins. | Outer-join nulls require careful predicates. |
| No `WHERE` | Targets every row. | Null behavior is irrelevant—the scope is the whole table. |
| `LIMIT` | Caps rows for single-table forms. | Use deterministic `ORDER BY` when row choice matters. |

## Revision notes

- Preview the exact `WHERE` and joins with `SELECT` before mutation.
- Wrap related changes in a transaction and check affected-row counts.
- Single-table `UPDATE`/`DELETE` can use `ORDER BY` and `LIMIT`; multi-table forms have different restrictions.
- Use null-safe comparison when detecting changes: `NOT (old_value <=> new_value)`.
- For duplicate deletion, rank rows in a CTE/derived result and join back by a stable primary key.
- `TRUNCATE` is DDL-like and not a row-by-row substitute for transactional `DELETE`.

## Examples

### Conditional update

```sql
UPDATE employees
SET bonus = CASE
  WHEN performance_rating >= 4 THEN salary * 0.10
  WHEN performance_rating IS NULL THEN NULL
  ELSE salary * 0.03
END
WHERE active = 1;
```

**Expected behavior:** Only active rows change; missing ratings produce a null bonus.

### Delete orphans

```sql
DELETE oi
FROM order_items AS oi
LEFT JOIN orders AS o ON o.order_id = oi.order_id
WHERE o.order_id IS NULL;
```

**Expected behavior:** Deletes order items whose parent row is absent.

## Tricky parts

- Running without first validating `WHERE`.
- Expecting `col = NULL` to match nulls.
- Updating the wrong side of a multi-table join.
- Relying on row order without `ORDER BY`.
- Assuming all DDL can be rolled back like DML.

## Interview checks

1. How do you safely remove duplicates?
2. How do you update one table using another?
3. Why use `<=>` when comparing nullable old/new values?

## 30-second recap

- Modify exactly the intended rows, including join-based changes and safe duplicate removal.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Updating and Deleting Data](https://dev.mysql.com/doc/refman/8.4/en/update.html)
