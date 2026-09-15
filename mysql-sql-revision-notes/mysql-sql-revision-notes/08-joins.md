# 08 — Joins

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Combine tables without losing required rows or multiplying data unexpectedly.

## Syntax template

```sql
SELECT ...
FROM left_table AS l
[INNER|LEFT|RIGHT] JOIN right_table AS r
  ON r.key_col = l.key_col;

SELECT ... FROM a CROSS JOIN b;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `INNER JOIN` | Only matching combinations. | Null join keys do not match with `=`. |
| `LEFT JOIN` | All left rows plus matches. | No match → every right-side output column is `NULL`. |
| `RIGHT JOIN` | All right rows plus matches. | No match → left-side output columns are `NULL`. |
| `CROSS JOIN` | Cartesian product. | Null values are simply carried into combinations. |
| `ON a <=> b` | Null-safe match. | Null keys match each other—use only when semantically correct. |
| `USING(k)` | Joins equal same-named keys and emits one key column. | Coalesced key behavior matters in outer joins. |

## Revision notes

- Cardinality drives output size: one-to-many joins repeat the one-side row once per match.
- A right-table filter in `WHERE` can remove null-extended rows and turn a `LEFT JOIN` effectively into an inner join.
- Place match restrictions in `ON`; place final-result restrictions in `WHERE`.
- For existence tests use `EXISTS`; for nonexistence use `NOT EXISTS`.
- MySQL has no direct `FULL OUTER JOIN`; emulate carefully with two outer-join branches and `UNION ALL`.
- Aggregate each many-side table before joining when joining multiple one-to-many relations would multiply measures.

## Examples

### Keep departments with no employees

```sql
SELECT d.department_id, d.name, COUNT(e.employee_id) AS employee_count
FROM departments AS d
LEFT JOIN employees AS e
  ON e.department_id = d.department_id
GROUP BY d.department_id, d.name;
```

**Expected behavior:** `COUNT(e.employee_id)` yields `0` for an unmatched department.

### Anti join

```sql
SELECT c.customer_id
FROM customers AS c
WHERE NOT EXISTS (
  SELECT 1 FROM orders AS o
  WHERE o.customer_id = c.customer_id
);
```

**Expected behavior:** Customers with no order; safe even if `orders.customer_id` can be null.

## Tricky parts

- Filtering the optional side in `WHERE`.
- Missing a join condition.
- Joining on incomplete composite keys.
- Counting `*` after a left join when unmatched rows should count as zero.
- Using `DISTINCT` to hide an incorrect join.

## Interview checks

1. Why can a join increase row count?
2. How do you find unmatched rows safely?
3. How do `ON` and `WHERE` differ for a left join?

## 30-second recap

- Combine tables without losing required rows or multiplying data unexpectedly.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Joins](https://dev.mysql.com/doc/refman/8.4/en/join.html)
