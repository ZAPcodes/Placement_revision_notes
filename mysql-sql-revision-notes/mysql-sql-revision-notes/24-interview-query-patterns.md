# 24 — Essential Interview Query Patterns

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Recognize reusable shapes behind common placement SQL questions.

## Syntax template

```sql
-- Solve in this order:
-- 1. Define one output row precisely.
-- 2. Identify grain/cardinality of each input.
-- 3. Choose join/group/window/anti-join pattern.
-- 4. Decide tie and NULL policy.
-- 5. Add deterministic ordering and test edge cases.
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Nth distinct value | `DENSE_RANK` then filter rank. | Decide whether null is a candidate; usually exclude it. |
| Top N rows/group | `ROW_NUMBER` for exactly N; `RANK`/`DENSE_RANK` to include ties. | Descending order places null measures last. |
| Anti join | `NOT EXISTS`. | Safe with nulls in child key projection. |
| Conditional pivot | `SUM(CASE...)` or `COUNT(CASE...)`. | Choose zero versus null intentionally. |
| Gaps and islands | Stable grouping key from value/date minus row sequence. | Null/missing values need explicit exclusion or bucket. |
| Latest per entity | Rank by timestamp plus unique tie-breaker. | Null timestamps usually sort last in descending order. |

## Revision notes

- Second-highest salary usually means second distinct salary; clarify whether ties should produce multiple employees.
- Relational division (“customers who bought every product”) uses double `NOT EXISTS` or grouped count matched to target count.
- For running totals, specify `ROWS` and a deterministic order.
- For consecutive-day streaks, first reduce multiple events per user/day to distinct dates.
- For median, rank/count rows and average the middle one or two values; define null exclusion.
- Always test empty input, one row, duplicates/ties, nulls, missing parents, and boundary dates.

## Pattern catalog

| Problem shape | Primary MySQL pattern | Decision to state |
|---|---|---|
| Nth/second-highest value | `DENSE_RANK()` | Distinct value? Return all ties? Exclude null? |
| Top N per group | `ROW_NUMBER`, `RANK`, or `DENSE_RANK` | Exactly N rows or include boundary ties? |
| Above group average | `AVG() OVER (PARTITION BY ...)` | `AVG` ignores null measures |
| Duplicate detection | `GROUP BY business_key HAVING COUNT(*) > 1` | Which columns define a duplicate? |
| Duplicate removal | `ROW_NUMBER` then delete by primary key | Which row should survive? |
| Present in A, absent in B | `NOT EXISTS` | Prefer over nullable `NOT IN` |
| Bought every product | Double `NOT EXISTS` or matched-count division | What is the target product set? |
| Manager/report count | Self join + grouping | Keep managers with zero reports? |
| Running total | Window `SUM` with explicit `ROWS` frame | Deterministic row order |
| Month-over-month change | Aggregate by month, then `LAG` | Missing months and divide-by-zero policy |
| Percent of total | `value / SUM(value) OVER (...)` | Decimal division and zero total |
| First/last row per group | `ROW_NUMBER` | Tie-breaker and null date placement |
| Latest record per entity | Descending `ROW_NUMBER` | Stable unique tie-breaker |
| Consecutive dates | Date minus `ROW_NUMBER` grouping key | Deduplicate multiple events per day first |
| Gaps and islands | Boundary flag + running sum, or shifted-key method | Required adjacency definition |
| Missing dates/numbers | Recursive CTE/calendar table + anti join | Inclusive boundaries |
| Previous/next comparison | `LAG` / `LEAD` | No-neighbor default versus stored null |
| Median | Rank/count, average middle position(s) | Exclude nulls; even-row definition |
| Overlapping ranges | `a.start <= b.end AND b.start <= a.end` | Closed versus half-open intervals |
| Conditional pivot | `SUM(CASE...)` / `COUNT(CASE...)` | Zero versus null output |
| Unpivot | `UNION ALL` | Preserve duplicates and source labels |
| Hierarchy | Recursive CTE | Roots, maximum depth, cycles |
| Sessionization | New-session flag from `LAG`, then cumulative `SUM` | Exact inactivity threshold |
| Cohort/retention | First event + elapsed-period grouping | Time zone and cohort definition |
| Merge intervals | Running maximum end + boundary grouping | Touching intervals count as merged? |
| Longest streak | Islands + grouped count | Distinct day/value grain |

## Additional canonical patterns

### Customers who bought every product

```sql
SELECT c.customer_id
FROM customers AS c
WHERE NOT EXISTS (
  SELECT 1
  FROM products AS p
  WHERE NOT EXISTS (
    SELECT 1
    FROM orders AS o
    JOIN order_items AS oi ON oi.order_id = o.order_id
    WHERE o.customer_id = c.customer_id
      AND oi.product_id = p.product_id
  )
);
```

If `products` is empty, every customer qualifies (vacuous truth). Clarify whether that is desired.

### Consecutive login-day streaks

```sql
WITH days AS (
  SELECT DISTINCT user_id, DATE(login_at) AS login_date
  FROM logins
), numbered AS (
  SELECT user_id, login_date,
         ROW_NUMBER() OVER (
           PARTITION BY user_id ORDER BY login_date
         ) AS rn
  FROM days
), islands AS (
  SELECT user_id, login_date,
         DATE_SUB(login_date, INTERVAL rn DAY) AS island_key
  FROM numbered
)
SELECT user_id, MIN(login_date) AS streak_start,
       MAX(login_date) AS streak_end, COUNT(*) AS streak_days
FROM islands
GROUP BY user_id, island_key;
```

Deduplicating user/day before numbering is essential.

### Conditional pivot

```sql
SELECT department_id,
       SUM(CASE WHEN job_title = 'Engineer' THEN 1 ELSE 0 END) AS engineers,
       SUM(CASE WHEN job_title = 'Manager'  THEN 1 ELSE 0 END) AS managers,
       COUNT(CASE WHEN salary IS NULL THEN 1 END) AS missing_salary
FROM employees
GROUP BY department_id;
```

`ELSE 0` makes a nonempty group's conditional sum zero rather than null.

## Examples

### Second distinct highest salary

```sql
WITH ranked AS (
  SELECT employee_id, salary,
         DENSE_RANK() OVER (ORDER BY salary DESC) AS salary_rank
  FROM employees
  WHERE salary IS NOT NULL
)
SELECT employee_id, salary
FROM ranked
WHERE salary_rank = 2;
```

**Expected behavior:** All employees tied at the second distinct known salary; no rows if fewer than two distinct salaries.

### Latest order per customer

```sql
WITH x AS (
  SELECT o.*,
         ROW_NUMBER() OVER (
           PARTITION BY customer_id
           ORDER BY ordered_at DESC, order_id DESC
         ) AS rn
  FROM orders AS o
)
SELECT * FROM x WHERE rn = 1;
```

**Expected behavior:** Exactly one deterministic latest order for each customer.

## Tricky parts

- Solving before clarifying tie behavior.
- Using `MAX` and selecting unrelated columns.
- Applying window functions before deduplicating the correct grain.
- Dividing integers or by zero without a policy.
- Testing only the happy path.

## Interview checks

1. Second highest with and without ties?
2. Customers who bought every product?
3. Longest consecutive login streak?

## 30-second recap

- Recognize reusable shapes behind common placement SQL questions.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Essential Interview Query Patterns](https://dev.mysql.com/doc/refman/8.4/en/example-maximum-column-group-row.html)
