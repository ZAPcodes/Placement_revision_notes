# 02 — Basic SELECT Queries

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Project columns and expressions, name results, and understand when duplicate removal happens.

## Syntax template

```sql
SELECT [DISTINCT]
       expression [AS alias], ...
FROM table_name [AS t];

SELECT 2 + 3 AS answer;
SELECT CURRENT_DATE AS today;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `*` | All visible columns in table order. | Preserves column `NULL`s. |
| Expression alias | Changes the output label, not stored schema. | Alias may label a `NULL` result. |
| `DISTINCT` | One row per unique selected tuple. | Duplicate `NULL` values are treated as duplicates. |
| No `FROM` | One computed row in MySQL. | A null expression returns one row containing `NULL`. |
| Scalar function | One value per input row. | Function-specific; check its null contract. |

## Revision notes

- List required columns instead of `SELECT *` in production code.
- A table alias is visible to later clauses; after aliasing a table, qualify it using the alias, not the original name.
- A select-list alias is normally available to `GROUP BY`, `ORDER BY`, and `HAVING`, but not to `WHERE` because `WHERE` is evaluated earlier.
- `DISTINCT` applies to the entire selected row, not just the first column.
- Column order in the result is the select-list order.

## Examples

### Projection and calculation

```sql
SELECT e.employee_id,
       CONCAT(e.first_name, ' ', e.last_name) AS employee_name,
       e.salary * 12 AS annual_salary
FROM employees AS e;
```

**Expected behavior:** One output row per employee; `annual_salary` is `NULL` when `salary` is `NULL`.

### Distinct combinations

```sql
SELECT DISTINCT department_id, job_title
FROM employees;
```

**Expected behavior:** One row per distinct `(department_id, job_title)` pair.

## Tricky parts

- Believing `DISTINCT(a), b` deduplicates only `a`.
- Using a select alias in `WHERE`.
- Assuming `DISTINCT` sorts results.
- Forgetting that `CONCAT()` returns `NULL` if any argument is `NULL`.

## Interview checks

1. What is the difference between `DISTINCT` and `GROUP BY`?
2. Can an alias defined in `SELECT` be used in `WHERE`? Why?
3. What does `SELECT NULL;` return?

## 30-second recap

- Project columns and expressions, name results, and understand when duplicate removal happens.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Basic SELECT Queries](https://dev.mysql.com/doc/refman/8.4/en/select.html)
