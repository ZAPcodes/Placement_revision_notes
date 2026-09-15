# 11 — Common Table Expressions

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Name intermediate results and solve hierarchical or iterative problems with recursive CTEs.

## Syntax template

```sql
WITH cte_name AS (
  SELECT ...
)
SELECT ... FROM cte_name;

WITH RECURSIVE seq AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n + 1 FROM seq WHERE n < 10
)
SELECT n FROM seq;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Nonrecursive CTE | Named statement-scoped result. | Preserves nulls produced by its query. |
| Recursive anchor | Initial rows. | Null column values influence inferred column types. |
| Recursive member | Adds rows until it produces none or hits limit. | Null predicates in its `WHERE` do not pass. |
| `UNION ALL` | Retains repeated recursive rows. | Can loop if termination/cycle handling is wrong. |
| `UNION DISTINCT` | Deduplicates rows per recursive accumulation. | Treats duplicate nulls as equal. |

## Revision notes

- A CTE exists only for the statement and is not a stored view or temporary table.
- The anchor determines recursive-column names and types; cast wide enough to avoid truncation.
- A recursive CTE needs a termination condition and often explicit cycle protection.
- Use `WITH RECURSIVE` once even when only one of several CTEs is recursive.
- CTEs improve readability but do not guarantee materialization or better performance.
- MySQL limits recursion through `cte_max_recursion_depth`.

## Examples

### Number sequence

```sql
WITH RECURSIVE seq AS (
  SELECT 1 AS n
  UNION ALL
  SELECT n + 1 FROM seq WHERE n < 5
)
SELECT n FROM seq ORDER BY n;
```

**Expected behavior:** Rows `1,2,3,4,5`.

### Organization tree

```sql
WITH RECURSIVE org AS (
  SELECT employee_id, manager_id, 0 AS depth
  FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.employee_id, e.manager_id, o.depth + 1
  FROM employees AS e
  JOIN org AS o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

**Expected behavior:** One row per reachable employee with depth from a root.

## Tricky parts

- Missing termination condition.
- Anchor string type too short for a growing path.
- Using `UNION` accidentally and hiding valid duplicate paths.
- Expecting `ORDER BY` inside recursion to define final order.
- Forgetting cycle handling for general graphs.

## Interview checks

1. Anchor versus recursive member?
2. Why might you cast a path in the anchor?
3. CTE versus derived table versus temp table?

## 30-second recap

- Name intermediate results and solve hierarchical or iterative problems with recursive CTEs.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Common Table Expressions](https://dev.mysql.com/doc/refman/8.4/en/with.html)
