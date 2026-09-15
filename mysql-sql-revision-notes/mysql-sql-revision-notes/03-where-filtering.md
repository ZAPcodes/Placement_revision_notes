# 03 — Filtering with WHERE

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Build correct predicates using comparison, ranges, membership, patterns, regular expressions, and three-valued logic.

## Syntax template

```sql
SELECT columns
FROM table_name
WHERE predicate;

-- common predicates
WHERE x BETWEEN low AND high
  AND y IN (v1, v2)
  AND name LIKE 'A%'
  AND deleted_at IS NULL;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `=`, `<>`, `<`, `>` | `1` or `0` for known operands. | Returns `NULL` if either operand is `NULL`. |
| `<=>` | Null-safe equality: always `1` or `0`. | Returns `1` when both operands are `NULL`. |
| `BETWEEN a AND b` | Inclusive range test. | Usually `NULL` if needed comparison is unknown. |
| `IN (...)` | `1` on a match; otherwise `0`. | If no match and a tested value is `NULL`, result can be `NULL`. |
| `LIKE` / `REGEXP` | `1` on pattern match, else `0`. | Returns `NULL` when an operand is `NULL`. |
| `IS NULL` | Always `1` or `0`. | The correct test for SQL `NULL`. |

## Revision notes

- Precedence relevant to interviews: `NOT` binds before `AND`; `AND` binds before `OR`. Parenthesize mixed conditions.
- `BETWEEN` includes both endpoints. For timestamps, prefer a half-open range: `ts >= start AND ts < next_day`.
- `LIKE` uses `%` for any sequence and `_` for exactly one character.
- Collation controls case/accent sensitivity. Use an appropriate collation rather than wrapping indexed columns unnecessarily.
- `NOT IN` is dangerous when the list or subquery can contain `NULL`; prefer `NOT EXISTS`.
- Truth table: `0 AND NULL = 0`, `1 AND NULL = NULL`, `1 OR NULL = 1`, `0 OR NULL = NULL`, `NOT NULL = NULL`.

## Examples

### Half-open date filter

```sql
SELECT *
FROM orders
WHERE ordered_at >= '2026-09-01'
  AND ordered_at <  '2026-10-01';
```

**Expected behavior:** All September timestamps, including those with a time component.

### Safe nullable equality

```sql
SELECT *
FROM employees
WHERE manager_id <=> NULL;
```

**Expected behavior:** Rows whose `manager_id` is `NULL`; clearer form is `IS NULL`.

## Tricky parts

- `NOT IN (1, 2, NULL)` can never be TRUE.
- `WHERE col <> NULL` keeps nothing.
- Missing parentheses around `OR`.
- Using `BETWEEN '2026-09-01' AND '2026-09-30'` for a `DATETIME` column.
- Leading wildcard `LIKE '%abc'` usually prevents a B-tree range lookup.

## Interview checks

1. Explain the `NOT IN`/`NULL` trap.
2. Is `BETWEEN` inclusive?
3. What does `NULL <=> NULL` return?

## 30-second recap

- Build correct predicates using comparison, ranges, membership, patterns, regular expressions, and three-valued logic.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Filtering with WHERE](https://dev.mysql.com/doc/refman/8.4/en/comparison-operators.html)
