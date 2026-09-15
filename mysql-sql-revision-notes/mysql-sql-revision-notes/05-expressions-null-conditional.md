# 05 — Expressions, Conditional Logic, and NULL

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Master the exact return behavior of conditional expressions and SQL unknown values.

## Syntax template

```sql
CASE
  WHEN condition THEN result
  WHEN condition THEN result
  ELSE fallback
END

IF(condition, true_result, false_result)
IFNULL(expr, fallback)
COALESCE(expr1, expr2, ...)
NULLIF(expr1, expr2)
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `CASE` | First matching branch; `ELSE`; otherwise `NULL`. | A simple `CASE x WHEN NULL` does not match null—use `WHEN x IS NULL`. |
| `IF(p,a,b)` | `a` if `p` is true; otherwise `b`. | A `NULL` predicate chooses `b`. |
| `IFNULL(a,b)` | `a` unless it is `NULL`; then `b`. | Both null → `NULL`. |
| `COALESCE(...)` | First non-null argument. | All arguments null → `NULL`. |
| `NULLIF(a,b)` | `NULL` if `a=b`; otherwise `a`. | `NULLIF(NULL,x)` returns `NULL`. |
| Arithmetic | Calculated value. | Most operators propagate `NULL`. |

## Revision notes

- `NULL` means missing/unknown, not zero, false, empty text, or the word `NULL`.
- `WHERE` and `HAVING` retain only TRUE; both FALSE and UNKNOWN are removed.
- Use `COALESCE` for portable fallback logic and `IFNULL` when a MySQL-specific two-argument form is clearer.
- Do not replace missing data with zero unless that matches the business meaning.
- Result type is inferred from all branches/arguments; mixing types may cause coercion.
- Null-safe equality `<=>` is MySQL-specific and never returns `NULL`.

## Examples

### Three-valued logic

```sql
SELECT NULL = NULL, NULL <=> NULL,
       1 AND NULL, 0 AND NULL,
       1 OR NULL, 0 OR NULL;
```

**Expected behavior:** Returns `NULL, 1, NULL, 0, 1, NULL`.

### Bucket salaries

```sql
SELECT employee_id,
       CASE
         WHEN salary IS NULL THEN 'unknown'
         WHEN salary >= 100000 THEN 'high'
         ELSE 'standard'
       END AS salary_band
FROM employees;
```

**Expected behavior:** Every employee receives a non-null text label.

## Tricky parts

- Testing null with `=`.
- Writing `CASE col WHEN NULL`.
- Using `COALESCE(col, 0)` without checking semantics.
- Forgetting that an omitted `ELSE` returns `NULL`.
- Expecting short-circuit behavior to protect every invalid expression in every optimization context.

## Interview checks

1. Give the truth table for `AND`/`OR` with `NULL`.
2. Difference between `IFNULL`, `COALESCE`, and `NULLIF`?
3. Why does `CASE x WHEN NULL` fail to identify nulls?

## 30-second recap

- Master the exact return behavior of conditional expressions and SQL unknown values.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Expressions, Conditional Logic, and NULL](https://dev.mysql.com/doc/refman/8.4/en/working-with-null.html)
