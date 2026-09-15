# 09 — Set Operations

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Combine compatible result sets and control duplicate elimination.

## Syntax template

```sql
query_block
UNION [ALL|DISTINCT]
query_block;

query_block INTERSECT [ALL|DISTINCT] query_block;
query_block EXCEPT [ALL|DISTINCT] query_block;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `UNION` | Rows from both inputs, duplicates removed. | Nulls compare equal for duplicate elimination. |
| `UNION ALL` | All rows from both inputs. | Preserves every null and duplicate row. |
| `INTERSECT` | Rows present in both inputs. | Nulls participate in set duplicate semantics. |
| `EXCEPT` | Rows in left input but not right. | Null rows can cancel under set semantics. |
| Final `ORDER BY` | Sorts the combined output. | Normal MySQL null ordering applies. |

## Revision notes

- Corresponding query blocks must return the same number of columns with compatible types.
- Output column names come from the first query block.
- `UNION ALL` is normally faster because it avoids global duplicate elimination.
- In MySQL 8.4, `INTERSECT` has higher precedence than `UNION` and `EXCEPT`; use parentheses for clarity.
- A branch-local `ORDER BY` matters only with a branch-local `LIMIT`; use the final `ORDER BY` for output order.
- Use joins when combining columns; use set operations when stacking/comparing rows.

## Examples

### Stack active IDs

```sql
SELECT customer_id FROM retail_customers
UNION
SELECT customer_id FROM wholesale_customers;
```

**Expected behavior:** One row per distinct customer ID across both sources.

### Keep duplicates intentionally

```sql
SELECT email, 'customer' AS source FROM customers
UNION ALL
SELECT email, 'lead' FROM leads;
```

**Expected behavior:** Every source row is retained.

## Tricky parts

- Using `UNION` when duplicates are meaningful.
- Expecting output names from the second branch.
- Different column counts.
- Putting the only `ORDER BY` in a branch.
- Forgetting version compatibility when using `INTERSECT`/`EXCEPT` on older MySQL.

## Interview checks

1. `UNION` versus `UNION ALL`?
2. How are nulls treated during duplicate removal?
3. When should you use a join instead?

## 30-second recap

- Combine compatible result sets and control duplicate elimination.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Set Operations](https://dev.mysql.com/doc/refman/8.4/en/set-operations.html)
