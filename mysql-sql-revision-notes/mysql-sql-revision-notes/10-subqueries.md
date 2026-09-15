# 10 — Subqueries

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Choose scalar, table, correlated, membership, or existence subqueries safely.

## Syntax template

```sql
SELECT ...
FROM table_name AS t
WHERE t.value > (SELECT AVG(value) FROM table_name)
  AND EXISTS (SELECT 1 FROM child AS c WHERE c.parent_id = t.id);
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Scalar subquery | One value. | No rows → `NULL`; more than one row → error. |
| `EXISTS(subquery)` | `1` if any row exists, else `0`. | Selected values—including null—do not matter. |
| `IN(subquery)` | Membership test. | No match plus a null candidate → `NULL`. |
| `NOT EXISTS` | `1` when subquery returns no rows. | Safe from projected null values. |
| `ANY` / `SOME` | True if comparison is true for at least one row. | Can return `NULL` when no comparison is true and some are unknown. |
| `ALL` | True if comparison holds for every row; true on empty set. | Unknown comparisons can yield `NULL`. |

## Revision notes

- A correlated subquery references the outer row and is logically reevaluated per outer row, though the optimizer may transform it.
- `EXISTS` checks row existence; write `SELECT 1` to signal intent.
- Prefer `NOT EXISTS` over `NOT IN` when nullability is possible.
- A subquery in `FROM` is a derived table and needs an alias.
- MySQL supports lateral derived tables so a derived table can reference earlier `FROM` items.
- Error 1093 can occur when modifying a table while directly selecting from the same table; an extra derived-table level or a join/CTE rewrite can help.

## Examples

### Above global average

```sql
SELECT employee_id, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**Expected behavior:** Only known salaries above the average of known salaries. If all salaries are null, the scalar average is null and no row passes.

### Above department average

```sql
SELECT e.employee_id, e.department_id, e.salary
FROM employees AS e
WHERE e.salary > (
  SELECT AVG(x.salary)
  FROM employees AS x
  WHERE x.department_id = e.department_id
);
```

**Expected behavior:** A correlated comparison within each department.

## Tricky parts

- Scalar subquery returning multiple rows.
- `NOT IN` with nullable output.
- Forgetting the derived-table alias.
- Using a correlated subquery when a grouped join/window is clearer.
- Assuming `EXISTS` cares about its select list.

## Interview checks

1. What does an empty scalar subquery return?
2. Why is `NOT EXISTS` safer than `NOT IN`?
3. Correlated versus non-correlated subquery?

## 30-second recap

- Choose scalar, table, correlated, membership, or existence subqueries safely.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Subqueries](https://dev.mysql.com/doc/refman/8.4/en/subqueries.html)
