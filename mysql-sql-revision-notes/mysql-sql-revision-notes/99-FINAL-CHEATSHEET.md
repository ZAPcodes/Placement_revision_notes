# MySQL Final Cheatsheet

> MySQL 8.4 LTS · foundational syntax + dates + window functions + `NULL` behavior

## 1. Query skeleton and logical order

```sql
SELECT [DISTINCT] expressions
FROM tables
JOIN other_table ON join_condition
WHERE row_condition
GROUP BY grouping_expressions
HAVING group_condition
WINDOW window_name AS (...)
ORDER BY expressions
LIMIT count OFFSET offset;
```

**Logical processing:** `FROM/JOIN` → `WHERE` → `GROUP BY` → `HAVING` → windows → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`.

## 2. Foundational syntax

```sql
-- filter
WHERE salary >= 50000
  AND department_id IN (10, 20)
  AND name LIKE 'A%'
  AND deleted_at IS NULL

-- sort/page
ORDER BY created_at DESC, id DESC
LIMIT 20 OFFSET 40

-- conditional value
CASE
  WHEN score IS NULL THEN 'unknown'
  WHEN score >= 90 THEN 'A'
  ELSE 'other'
END

-- aggregate
SELECT department_id, COUNT(*) AS employees, AVG(salary) AS avg_salary
FROM employees
WHERE active = 1
GROUP BY department_id
HAVING COUNT(*) >= 5;
```

### Join patterns

```sql
-- matched only
FROM a INNER JOIN b ON b.a_id = a.id

-- keep every a
FROM a LEFT JOIN b ON b.a_id = a.id

-- a rows with no b
FROM a
WHERE NOT EXISTS (SELECT 1 FROM b WHERE b.a_id = a.id)
```

Keep an optional-side filter in `ON` when left rows must survive:

```sql
FROM departments AS d
LEFT JOIN employees AS e
  ON e.department_id = d.department_id
 AND e.active = 1
```

### Subquery and CTE patterns

```sql
-- scalar: no rows -> NULL; >1 row -> error
WHERE salary > (SELECT AVG(salary) FROM employees)

-- CTE
WITH totals AS (
  SELECT customer_id, SUM(amount) AS total
  FROM orders
  GROUP BY customer_id
)
SELECT * FROM totals WHERE total >= 1000;
```

## 3. `NULL` master table

| Expression | Result / rule |
|---|---|
| `NULL = NULL` | `NULL` |
| `NULL <=> NULL` | `1` |
| `5 <=> NULL` | `0` |
| `x IS NULL` | Always `0` or `1` |
| `1 AND NULL` / `0 AND NULL` | `NULL` / `0` |
| `1 OR NULL` / `0 OR NULL` | `1` / `NULL` |
| `NOT NULL` | `NULL` |
| `WHERE predicate` | Keeps TRUE only; FALSE and NULL are removed |
| `COUNT(*)` | Counts rows; empty input → `0` |
| `COUNT(col)` | Counts non-null values; empty input → `0` |
| `SUM/AVG/MIN/MAX(col)` | Ignore nulls; no known values → `NULL` |
| `GROUP BY nullable_col` | Null keys form one group |
| `ORDER BY col ASC/DESC` | Nulls first / last in MySQL |
| `CONCAT(a,b)` | Null if either argument is null |
| `CONCAT_WS(sep,a,b)` | Skips null values after separator |
| `COALESCE(a,b,...)` | First non-null; all null → null |
| `CASE ...` without matching branch/`ELSE` | `NULL` |
| `NOT IN` when candidates include null | Can become `NULL`; prefer `NOT EXISTS` |
| Scalar subquery with no row | `NULL` |
| `LEFT JOIN` with no match | Right-side columns become `NULL` |

## 4. Date and time essentials

| Need | MySQL syntax | Important behavior |
|---|---|---|
| Current date | `CURDATE()` / `CURRENT_DATE` | Date only |
| Current timestamp | `NOW()` / `CURRENT_TIMESTAMP` | Constant within a statement |
| Extract part | `EXTRACT(YEAR FROM dt)` or `YEAR(dt)` | Null input → null |
| Add interval | `DATE_ADD(dt, INTERVAL 7 DAY)` | Null `dt` → null |
| Subtract interval | `DATE_SUB(dt, INTERVAL 1 MONTH)` | Month-end adjustment may occur |
| Days difference | `DATEDIFF(end,start)` | Ignores time portion |
| Unit difference | `TIMESTAMPDIFF(MONTH,start,end)` | Completed units, signed |
| Format | `DATE_FORMAT(dt, '%Y-%m-%d')` | Returns string |
| Parse | `STR_TO_DATE(text, format)` | Format mismatch → null + warning; invalid ranges may error in 8.4; zero/incomplete dates depend on SQL mode |
| Month end | `LAST_DAY(dt)` | Null/invalid date → null |

### Safe date-range patterns

```sql
-- one day for DATETIME/TIMESTAMP
WHERE created_at >= '2026-09-15'
  AND created_at <  '2026-09-16'

-- one month
WHERE created_at >= '2026-09-01'
  AND created_at <  '2026-10-01'

-- sargable year
WHERE created_at >= '2026-01-01'
  AND created_at <  '2027-01-01'
```

Avoid `DATE(created_at) = ...` or `YEAR(created_at) = ...` on a normally indexed column when a range expresses the same requirement.

### Date traps

- `BETWEEN` is inclusive. `BETWEEN '2026-09-01' AND '2026-09-30'` misses most of September 30 for a datetime column.
- `DATEDIFF(end,start)` and `TIMESTAMPDIFF(unit,start,end)` use different signatures.
- `TIMESTAMP` is time-zone converted; `DATETIME` is not.
- Month arithmetic can clamp to the last valid day.
- Invalid/zero-date behavior depends on SQL mode.

## 5. Window-function syntax

```sql
function(args) OVER (
  PARTITION BY group_col
  ORDER BY sort_col, unique_id
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
```

| Function | Meaning | Null / edge behavior |
|---|---|---|
| `ROW_NUMBER()` | Unique sequence | Nondeterministic among ties unless order is unique |
| `RANK()` | Peers share rank; gaps | Null sort keys can be peers |
| `DENSE_RANK()` | Peers share rank; no gaps | Null sort keys can be peers |
| `LAG(x,n,default)` | Previous value | No prior row → default (`NULL` by default); existing null stays null |
| `LEAD(x,n,default)` | Next value | No next row → default |
| `FIRST_VALUE(x)` | First value in frame | Can be null |
| `LAST_VALUE(x)` | Last value in frame | Default frame often ends at current peers |
| `SUM(x) OVER` | Window total | Ignores null x; all-null frame → null |
| `COUNT(x) OVER` | Known-value count | Ignores null; returns zero when none |
| `NTILE(n)` | Bucket number `1..n` | Bucket sizes differ by at most one |

### Ranking

```sql
SELECT employee_id, department_id, salary,
       ROW_NUMBER() OVER (
         PARTITION BY department_id
         ORDER BY salary DESC, employee_id
       ) AS rn,
       RANK() OVER (
         PARTITION BY department_id
         ORDER BY salary DESC
       ) AS salary_rank
FROM employees;
```

### Top N per group

```sql
WITH ranked AS (
  SELECT e.*,
         ROW_NUMBER() OVER (
           PARTITION BY department_id
           ORDER BY salary DESC, employee_id
         ) AS rn
  FROM employees AS e
)
SELECT * FROM ranked WHERE rn <= 3;
```

Use `RANK`/`DENSE_RANK` instead when all ties at the boundary should remain.

### Running total

```sql
SELECT order_id, ordered_at, amount,
       SUM(amount) OVER (
         ORDER BY ordered_at, order_id
         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM orders
ORDER BY ordered_at, order_id;
```

### Moving average

```sql
AVG(amount) OVER (
  ORDER BY ordered_at, order_id
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
) AS moving_avg_3_rows
```

### Previous-row comparison

```sql
WITH x AS (
  SELECT order_id, ordered_at, amount,
         LAG(amount) OVER (ORDER BY ordered_at, order_id) AS prev_amount
  FROM orders
)
SELECT *, amount - prev_amount AS change_amount
FROM x;
```

The first row has no predecessor, so `prev_amount` and `change_amount` are null.

### Correct partition-wide `LAST_VALUE`

```sql
LAST_VALUE(amount) OVER (
  PARTITION BY customer_id
  ORDER BY ordered_at, order_id
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

## 6. High-frequency interview patterns

### Second distinct highest salary

```sql
WITH x AS (
  SELECT employee_id, salary,
         DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
  FROM employees
  WHERE salary IS NOT NULL
)
SELECT employee_id, salary FROM x WHERE rnk = 2;
```

### Above department average

```sql
SELECT employee_id, department_id, salary
FROM (
  SELECT e.*, AVG(salary) OVER (PARTITION BY department_id) AS dept_avg
  FROM employees AS e
) AS x
WHERE salary > dept_avg;
```

### Latest row per entity

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

### Conditional aggregation

```sql
SELECT department_id,
       COUNT(*) AS total,
       COUNT(CASE WHEN salary >= 100000 THEN 1 END) AS high_paid,
       COUNT(*) - COUNT(salary) AS missing_salary
FROM employees
GROUP BY department_id;
```

## 7. Last-minute traps

1. `NOT IN` + any null candidate can reject everything.
2. A right-side filter in `WHERE` can destroy left-join preservation.
3. `COUNT(col)` ignores null; `COUNT(*)` counts rows.
4. `SUM` on no rows is null, not zero.
5. No `ORDER BY` means no guaranteed order.
6. Add a unique tie-breaker to ranking and pagination.
7. MySQL has no `QUALIFY`; filter window output in an outer query.
8. Use an explicit window frame for running totals and `LAST_VALUE`.
9. `BETWEEN` is inclusive; use half-open timestamp ranges.
10. `CONCAT` propagates null; `CONCAT_WS` skips null arguments.
11. `CHECK (x > 0)` accepts null/unknown unless `x` is also `NOT NULL`.
12. `UNIQUE` permits multiple nulls in MySQL.

## Official references

- [Working with NULL](https://dev.mysql.com/doc/refman/8.4/en/working-with-null.html)
- [Date and time functions](https://dev.mysql.com/doc/refman/8.4/en/date-and-time-functions.html)
- [Window-function syntax](https://dev.mysql.com/doc/refman/8.4/en/window-functions-usage.html)
- [SELECT statement](https://dev.mysql.com/doc/refman/8.4/en/select.html)
