# 04 — Sorting, Limiting, and Pagination

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Produce deterministic ranked output and choose safe pagination patterns.

## Syntax template

```sql
SELECT columns
FROM table_name
ORDER BY expr1 [ASC|DESC], expr2 [ASC|DESC]
LIMIT row_count OFFSET offset;

-- equivalent MySQL form
LIMIT offset, row_count;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `ORDER BY ... ASC` | Ascending order; ASC is default. | MySQL places `NULL` first. |
| `ORDER BY ... DESC` | Descending order. | MySQL places `NULL` last. |
| `LIMIT n` | At most `n` rows. | Does not change column values. |
| `OFFSET n` | Skips `n` sorted rows. | Without deterministic ordering pages may drift. |
| `FIELD()` sort | Position in supplied list; `0` if absent. | `NULL` normally produces `0`. |

## Revision notes

- Always add a unique tie-breaker to pagination order, for example `ORDER BY created_at DESC, id DESC`.
- `ORDER BY` can use a column, select alias, expression, or ordinal; ordinals are brittle and best avoided.
- To force nulls last in ascending order: `ORDER BY col IS NULL, col ASC`.
- Large offsets still require the server to find and skip earlier rows. Keyset pagination scales better.
- `LIMIT` without `ORDER BY` returns an arbitrary subset.

## Examples

### Top five salaries

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC, employee_id ASC
LIMIT 5;
```

**Expected behavior:** At most five deterministic rows; employees with null salary appear after known salaries.

### Keyset pagination

```sql
SELECT order_id, ordered_at
FROM orders
WHERE (ordered_at, order_id) < ('2026-09-15 10:00:00', 900)
ORDER BY ordered_at DESC, order_id DESC
LIMIT 20;
```

**Expected behavior:** The next page after the supplied composite cursor.

## Tricky parts

- Using `LIMIT` without `ORDER BY`.
- Omitting a tie-breaker.
- Assuming standard `NULLS LAST` syntax exists in MySQL.
- Using deep offset pagination on a large table.

## Interview checks

1. How do you emulate `NULLS LAST` in MySQL?
2. Why can offset pagination show duplicates?
3. Write keyset pagination for descending timestamps.

## 30-second recap

- Produce deterministic ranked output and choose safe pagination patterns.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Sorting, Limiting, and Pagination](https://dev.mysql.com/doc/refman/8.4/en/select.html)
