# 16 — Keys, Constraints, and Index Queries

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Enforce data rules and design indexes that match real query predicates and ordering.

## Syntax template

```sql
CREATE TABLE child (
  id BIGINT PRIMARY KEY,
  parent_id BIGINT NULL,
  code VARCHAR(50) UNIQUE,
  amount DECIMAL(10,2) CHECK (amount >= 0),
  CONSTRAINT fk_child_parent FOREIGN KEY (parent_id)
    REFERENCES parent(id) ON DELETE SET NULL
);

CREATE INDEX idx_emp_dept_salary
ON employees (department_id, salary DESC);
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Primary key | Unique and non-null row identity. | Null is rejected. |
| Unique index | Rejects duplicate non-null key tuples. | MySQL permits multiple nulls because nulls are not equal for uniqueness. |
| Foreign key | Requires matching parent for known key. | A nullable FK can bypass the parent check when a component is null. |
| `CHECK` | Rejects rows where expression is FALSE. | TRUE or UNKNOWN passes; use `NOT NULL` too when null is invalid. |
| B-tree index | Accelerates matching prefixes/ranges/order. | Can store nulls and may support `IS NULL`. |
| `ON DELETE SET NULL` | Preserves child and clears FK. | FK columns must permit null. |

## Revision notes

- A composite index `(a,b,c)` efficiently supports leftmost prefixes such as `(a)` and `(a,b)`; skipping `a` usually loses direct lookup power.
- After a range condition on one key part, later parts often cannot narrow the index range, though they can still help filtering/covering.
- A covering index contains every column needed by a query, avoiding extra table lookups.
- Index selectivity and workload matter; more indexes increase write cost and storage.
- Foreign-key columns should be indexed; InnoDB requires suitable indexes.
- A `CHECK (amount >= 0)` alone allows null because the result is UNKNOWN.

## Examples

### Enforce nonnegative known amount

```sql
amount DECIMAL(10,2) NOT NULL CHECK (amount >= 0)
```

**Expected behavior:** Both missing amounts and negative amounts are rejected.

### Match query order

```sql
CREATE INDEX idx_orders_customer_date
ON orders (customer_id, ordered_at DESC);
```

**Expected behavior:** Supports equality on customer and range/order on date.

## Tricky parts

- Assuming `UNIQUE` allows only one null.
- Using `CHECK` without `NOT NULL`.
- Indexing every column independently.
- Wrong composite-index order.
- Forgetting write overhead and redundant indexes.

## Interview checks

1. Explain the leftmost-prefix rule.
2. Can a unique column contain multiple nulls?
3. Why can a check constraint accept null?

## 30-second recap

- Enforce data rules and design indexes that match real query predicates and ordering.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Keys, Constraints, and Index Queries](https://dev.mysql.com/doc/refman/8.4/en/create-table.html)
