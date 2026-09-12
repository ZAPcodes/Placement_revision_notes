# 06. Advanced SQL

> Advanced SQL questions usually test whether you can preserve row detail while calculating comparisons, rankings and sequences.

## Window functions

Window functions compute across related rows without collapsing them into one row per group.

```sql
SELECT employee_id,
       department_id,
       salary,
       RANK() OVER (
         PARTITION BY department_id
         ORDER BY salary DESC
       ) AS salary_rank
FROM employees;
```

### Window anatomy

- `PARTITION BY`: independent groups.
- `ORDER BY`: ordering inside each partition.
- Frame: subset relative to current row used by frame-sensitive functions.

## Ranking functions

| Function | Ties | Sequence after tie |
|---|---|---|
| `ROW_NUMBER` | Arbitrarily unique unless tie-breaker supplied | No gaps |
| `RANK` | Same rank | Gaps |
| `DENSE_RANK` | Same rank | No gaps |

For deterministic `ROW_NUMBER`, add a unique final sort key.

## Common window patterns

- Top N per group: rank partitioned by group, filter outer query.
- Previous/next row: `LAG`, `LEAD`.
- Running total: ordered `SUM` with explicit frame.
- Moving average: frame over preceding/current rows.
- First/last event: careful ordering and window frames.
- Gap/island problems: compare row number, previous value or cumulative boundary flag.

### Default-frame trap

With window `ORDER BY`, default frames are DBMS/function dependent and can include peer rows. Write an explicit `ROWS BETWEEN ...` frame when row-by-row semantics matter.

## CTEs

```sql
WITH paid_orders AS (
  SELECT * FROM orders WHERE status = 'PAID'
)
SELECT customer_id, COUNT(*)
FROM paid_orders
GROUP BY customer_id;
```

- Improve decomposition/readability.
- Can be recursive.
- Do not assume a CTE is always materialized or always inlined; optimizer behavior/version matters.

## Recursive CTEs

Contain:

1. Anchor query.
2. Recursive term referencing accumulated result.
3. Termination when recursive term returns no new rows.

Uses: hierarchies, reachability and sequences. Protect against cycles and runaway recursion.

## Conditional aggregation

```sql
SELECT customer_id,
       SUM(CASE WHEN status = 'PAID' THEN amount ELSE 0 END) AS paid_amount,
       COUNT(CASE WHEN status = 'FAILED' THEN 1 END) AS failed_count
FROM orders
GROUP BY customer_id;
```

Be precise: `COUNT(CASE ... ELSE 0 END)` counts the zeros too because they are non-NULL.

## Duplicate handling

To detect duplicates, group by the business key and use `HAVING COUNT(*) > 1`.

To delete duplicates safely:

1. Define exactly which row survives.
2. Assign deterministic `ROW_NUMBER` within each duplicate group.
3. Delete rows with number greater than one using DBMS-supported syntax.
4. Add a unique constraint to prevent recurrence.

## Views vs materialized views

| View | Materialized view |
|---|---|
| Stores query definition | Stores query result |
| Reflects base changes at query time | Can become stale |
| Normal query cost still applies | Faster reads after refresh |
| Useful for interface/security | Useful for expensive repeated computation |

Refresh strategy determines consistency and cost.

## Procedures, functions and triggers

- **Procedure:** invoked operation; may support transaction/control behavior depending on DBMS.
- **Function:** returns value/table and can be used in expressions according to rules.
- **Trigger:** automatically fires on specified events.

Triggers can enforce cross-cutting rules but hide behavior, complicate debugging and create recursion/order/performance surprises.

## Upsert and race safety

Upsert combines insert with conflict handling atomically according to a unique/exclusion constraint. It is safer than application logic: “check if exists, then insert,” which races under concurrency.

Upsert still needs intentional conflict semantics: ignore, update selected fields, compare versions or reject.

## Pagination

### Offset pagination

Simple: `LIMIT size OFFSET n`. Deep pages may scan/discard many rows and can shift when concurrent changes occur.

### Keyset/cursor pagination

Filter after the last stable ordered key, such as `(created_at, id)`. Scales better for sequential navigation and is more stable, but does not directly jump to arbitrary page numbers.

Always define a total deterministic order.

## Prepared statements and dynamic SQL

- Prepared/parameterized statements separate SQL structure from values and prevent value-based injection.
- They can enable plan reuse, but parameter-sensitive data distributions may make one generic plan poor.
- Dynamic table/column identifiers cannot normally be passed as ordinary value parameters; whitelist and quote safely.

## Common traps

- Window functions keep rows; `GROUP BY` collapses groups.
- `RANK` can have gaps; `DENSE_RANK` cannot.
- `LAST_VALUE` may only see through the current frame unless explicitly widened.
- CTE is not guaranteed optimization barrier/materialization.
- Trigger-enforced behavior may run once per row or statement depending on type/product.
- Offset pagination can miss/duplicate items during concurrent updates.
- Upsert depends on the correct unique conflict target.

## Interview checks

1. `ROW_NUMBER` vs `RANK` vs `DENSE_RANK`?
2. Window function vs `GROUP BY`?
3. Find top three salaries per department conceptually.
4. Why should window frames sometimes be explicit?
5. CTE vs subquery? Is a CTE always materialized?
6. View vs materialized view?
7. Why is check-then-insert unsafe?
8. Offset vs keyset pagination?
9. Risks of triggers?
10. How do prepared statements prevent SQL injection?

## 60-second recall

- Windows compute across related rows without collapsing detail.
- Ranking tie behavior differs; deterministic order needs a unique tie-breaker.
- CTE improves structure but has no universal materialization guarantee.
- Upsert needs a real uniqueness constraint.
- Keyset pagination scales/stabilizes sequential pages.
- Prepared values prevent injection; identifiers require controlled construction.

