# 05. SQL Fundamentals

> SQL interviews test semantics more than syntax. Always reason about duplicates, NULLs, join cardinality and logical execution order.

## Core statement families

- DDL: `CREATE`, `ALTER`, `DROP`
- DML/query: `SELECT`, `INSERT`, `UPDATE`, `DELETE`
- Privileges: `GRANT`, `REVOKE`
- Transactions: `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`

Product behavior for transactional DDL, `TRUNCATE`, generated columns and types varies.

## Logical SELECT processing order

Conceptually:

1. `FROM` / joins
2. `WHERE`
3. `GROUP BY`
4. `HAVING`
5. `SELECT`
6. `DISTINCT`
7. `ORDER BY`
8. `LIMIT` / `OFFSET`

The optimizer can execute an equivalent physical plan in another order. Logical order explains why a `SELECT` alias is commonly unavailable in `WHERE`.

## Filtering and NULL

SQL predicates return `TRUE`, `FALSE` or `UNKNOWN`. `WHERE` keeps only `TRUE`.

```sql
WHERE manager_id IS NULL       -- correct NULL test
WHERE manager_id = NULL        -- never TRUE
```

Useful tools:

- `IS NULL`, `IS NOT NULL`
- `COALESCE(a, fallback)`
- `NULLIF(a, b)`
- `CASE WHEN ... THEN ... END`

Do not replace NULL blindly: unknown salary is not necessarily zero salary.

## Aggregates and grouping

| Expression | Behavior |
|---|---|
| `COUNT(*)` | Counts rows |
| `COUNT(col)` | Counts non-NULL values |
| `COUNT(DISTINCT col)` | Counts distinct non-NULL values |
| `SUM/AVG(col)` | Ignore NULL inputs; all-NULL may return NULL |

- `WHERE` filters rows before grouping.
- `HAVING` filters groups after aggregation.
- Every selected nonaggregate expression must be compatible with grouping rules.

## Join semantics

### Inner join

Returns matching row combinations. If one left row matches three right rows, it appears three times.

### Left outer join

Returns all left rows; unmatched right values become NULL.

```sql
-- Keeps every customer
SELECT c.id, o.id
FROM customers c
LEFT JOIN orders o
  ON o.customer_id = c.id
 AND o.status = 'PAID';
```

Putting `o.status = 'PAID'` in `WHERE` would remove NULL-extended rows and commonly turn the result into inner-join behavior.

### Self join

Same table used with different aliases, such as employee-to-manager.

### Cross join

Every pair of rows. An accidental missing join condition can multiply result size dramatically.

## Subqueries

- **Scalar:** expected to return one value.
- **Uncorrelated:** can be evaluated independently.
- **Correlated:** refers to outer row.
- **Derived table:** subquery in `FROM`.

The optimizer may transform subqueries into joins/semi-joins; write the clearest correct form and inspect plans for performance.

## `IN`, `EXISTS`, `NOT IN`, `NOT EXISTS`

- `EXISTS` checks whether at least one qualifying row exists.
- `IN` compares against a set/list of values.
- Modern optimizers can produce similar plans for equivalent cases.
- `NOT IN (subquery)` is dangerous if the subquery can return NULL: comparisons may become UNKNOWN and return no rows.
- `NOT EXISTS` usually expresses anti-join intent safely when correlated correctly.

## Set operators

| Operator | Duplicates |
|---|---|
| `UNION` | Removes |
| `UNION ALL` | Preserves |
| `INTERSECT` | Commonly removes unless ALL supported |
| `EXCEPT` | Commonly removes unless ALL supported |

Inputs need the same number of compatible columns. `UNION ALL` avoids duplicate-elimination work.

## `DELETE`, `TRUNCATE`, `DROP`

| Statement | Meaning |
|---|---|
| `DELETE` | Remove qualifying rows; supports predicate |
| `TRUNCATE` | Quickly remove all rows using engine-specific behavior |
| `DROP` | Remove the database object definition |

Logging, triggers, identity reset, locking and rollback behavior differ by DBMS; avoid universal claims such as “TRUNCATE can never roll back.”

## Views

- Stored query/interface, normally without storing result rows itself.
- Supports abstraction, permissions and compatibility.
- Does not automatically improve performance.
- Updatability depends on query and DBMS rules.

## Common traps

- SQL result order is undefined without `ORDER BY`.
- `DISTINCT` can hide a wrong join rather than fix it.
- Outer-join filtering in `WHERE` can discard preserved rows.
- `COUNT(col)` ignores NULL; `COUNT(*)` does not.
- `NOT IN` plus NULL is a classic correctness bug.
- `BETWEEN` is commonly inclusive at both ends; date/timestamp boundaries still need care.
- `LIKE '%abc'` generally cannot use an ordinary ordered index efficiently.

## Interview checks

1. Explain SQL logical execution order.
2. `WHERE` vs `HAVING`?
3. `COUNT(*)` vs `COUNT(column)`?
4. How can a left join accidentally become an inner join?
5. `UNION` vs `UNION ALL`?
6. `IN` vs `EXISTS`—correctness and performance?
7. Why can `NOT IN` fail with NULL?
8. `DELETE` vs `TRUNCATE` vs `DROP` without overgeneralizing?
9. Why can joins create duplicates?

## 60-second recall

- Think `FROM -> WHERE -> GROUP -> HAVING -> SELECT -> ORDER/LIMIT`.
- NULL yields UNKNOWN; use `IS NULL`.
- Join output cardinality depends on matches, not input row count alone.
- `WHERE` filters rows; `HAVING` filters groups.
- `COUNT(*)` counts rows; `COUNT(col)` counts known values.
- Prefer `UNION ALL` when duplicate removal is unnecessary.

## Reference

- [PostgreSQL SQL language documentation](https://www.postgresql.org/docs/current/sql.html)

