# 18 — Views and Temporary Results

> **Priority:** Reference  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Choose views, CTEs, derived tables, and temporary tables based on lifetime, reuse, and indexing needs.

## Syntax template

```sql
CREATE VIEW active_employees AS
SELECT employee_id, department_id, salary
FROM employees
WHERE active = 1
WITH CHECK OPTION;

CREATE TEMPORARY TABLE recent_orders AS
SELECT * FROM orders WHERE ordered_at >= CURRENT_DATE - INTERVAL 30 DAY;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| View | Stores a query definition, not normally materialized rows. | Exposes query-produced nulls. |
| `WITH CHECK OPTION` | Rejects writes through the view that make rows invisible to it. | UNKNOWN view predicate is not accepted for visibility. |
| Temporary table | Session-scoped physical table. | Normal null rules from its definition. |
| CTE | One-statement named result. | Preserves nulls. |
| Derived table | One-query-block result in `FROM`. | Preserves nulls and requires alias. |

## Revision notes

- Simple views may be updatable; aggregation, `DISTINCT`, grouping, set operations, and similar constructs generally make views non-updatable.
- `SQL SECURITY DEFINER` uses definer privileges; `INVOKER` uses caller privileges.
- A view is not a security boundary unless its privileges and definer are designed carefully.
- Temporary tables can be indexed and reused across statements but require lifecycle and connection-pool care.
- MySQL has no native materialized view; use summary tables plus refresh logic when needed.
- A CTE is best for readability inside one statement; it does not persist.

## Examples

### Restricted view

```sql
CREATE VIEW public_employee AS
SELECT employee_id, department_id
FROM employees
WHERE active = 1;
```

**Expected behavior:** Consumers see only selected columns and active rows, subject to view security configuration.

### Index a temp result

```sql
CREATE TEMPORARY TABLE ids (id BIGINT PRIMARY KEY);
INSERT INTO ids VALUES (1), (2), (3);
```

**Expected behavior:** A reusable session-local keyed set.

## Tricky parts

- Assuming views store snapshots.
- Expecting every view to be updatable.
- Leaking temp tables across pooled-session reuse.
- Using definer security without managing account lifecycle.
- Expecting indexes directly on an ordinary view.

## Interview checks

1. View versus materialized view?
2. CTE versus temporary table?
3. What makes a view non-updatable?

## 30-second recap

- Choose views, CTEs, derived tables, and temporary tables based on lifetime, reuse, and indexing needs.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Views and Temporary Results](https://dev.mysql.com/doc/refman/8.4/en/view-syntax.html)
