# 01 — SQL and MySQL Fundamentals

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Read MySQL statements correctly: command families, data types, literals, identifiers, truth values, and logical query processing.

## Syntax template

```sql
-- DQL
SELECT column_list FROM table_name;

-- DML
INSERT INTO t (c1) VALUES (v1);
UPDATE t SET c1 = v1 WHERE condition;
DELETE FROM t WHERE condition;

-- DDL / TCL / DCL
CREATE TABLE t (id INT PRIMARY KEY);
START TRANSACTION; COMMIT;
GRANT SELECT ON db_name.* TO user_name;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `SELECT` | A result set; zero or more rows. | Expressions can return `NULL`. |
| `WHERE` | Keeps only rows whose predicate is TRUE (`1`). | FALSE (`0`) and UNKNOWN (`NULL`) are both discarded. |
| Arithmetic | Numeric value after type conversion. | Most arithmetic with `NULL` returns `NULL`. |
| Boolean context | `0` is false; nonzero is true. | `NULL` is unknown and behaves as false for filtering. |
| String literal | Text enclosed in single quotes. | SQL `NULL` is not the string `'NULL'`. |

## Revision notes

- Logical processing model for a typical query: `FROM/JOIN` → `WHERE` → `GROUP BY` → `HAVING` → window functions → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`.
- Use backticks for an unavoidable identifier such as ``order``; use single quotes for strings. Do not use quotes as identifier delimiters.
- `DECIMAL(p,s)` is exact and preferred for money; `FLOAT`/`DOUBLE` are approximate.
- `DATE` stores a date; `DATETIME` stores a wall-clock date/time; `TIMESTAMP` is converted between the session time zone and UTC and has a narrower range.
- `BOOLEAN` is a synonym for `TINYINT(1)`; it is not a separate storage type.
- Implicit conversion can silently change comparison meaning and prevent index use. Match parameter types to column types.
- SQL mode matters. Strict mode converts many silent adjustments into errors; `ONLY_FULL_GROUP_BY` enforces deterministic grouping.

## Examples

### Truth and `NULL`

```sql
SELECT 0 AS false_value, 7 AS true_value,
       1 = NULL AS unknown_value,
       NULL IS NULL AS is_null;
```

**Expected behavior:** `false_value=0`, `true_value=7`, `unknown_value=NULL`, `is_null=1`.

### Inspect the environment

```sql
SELECT VERSION(), @@sql_mode, @@session.time_zone;
```

**Expected behavior:** One row describing the server and current session.

## Tricky parts

- Writing `column = NULL` instead of `column IS NULL`.
- Using floating point for exact currency calculations.
- Assuming displayed row order without `ORDER BY`.
- Relying on implicit numeric/string conversion.
- Confusing an empty string or zero with `NULL`.

## Interview checks

1. Why does `WHERE NULL` return no rows?
2. State the logical processing order of a query.
3. When would you choose `DATETIME` over `TIMESTAMP`?

## 30-second recap

- Read MySQL statements correctly: command families, data types, literals, identifiers, truth values, and logical query processing.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — SQL and MySQL Fundamentals](https://dev.mysql.com/doc/refman/8.4/en/language-structure.html)
