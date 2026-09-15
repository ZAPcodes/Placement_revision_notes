# 06 — Built-in MySQL Functions

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Recall common string, numeric, and date functions together with their return and null behavior.

## Syntax template

```sql
SELECT CONCAT(first_name, ' ', last_name);
SELECT SUBSTRING(text_col, 1, 5);
SELECT ROUND(amount, 2);
SELECT DATE_ADD(order_date, INTERVAL 7 DAY);
SELECT TIMESTAMPDIFF(YEAR, birth_date, CURDATE());
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `CONCAT(a,b,...)` | Concatenated string. | Returns `NULL` if any argument is `NULL`. |
| `CONCAT_WS(sep,...)` | Joined non-null arguments. | Skips null values after separator; null separator → `NULL`. |
| `LENGTH(s)` | Bytes; `CHAR_LENGTH(s)` returns characters. | Null input → `NULL`. |
| `GREATEST` / `LEAST` | Largest/smallest argument after coercion. | Any null argument → `NULL`. |
| `ROUND`, `ABS`, `DATE_ADD` | Transformed value. | Null required input → `NULL`. |
| `RAND()` | Pseudo-random value in `[0,1)`. | Takes no nullable input. |

## Revision notes

- Use `CONCAT_WS(' ', first_name, middle_name, last_name)` when nullable name parts should be skipped.
- `SUBSTRING(str,pos,len)` uses 1-based positions; negative positions count from the end.
- `ROUND()` and exact/approximate types can differ; use `DECIMAL` for exact financial rounding.
- `DATEDIFF(a,b)` returns whole date boundaries as `DATE(a)-DATE(b)` and ignores time-of-day.
- `TIMESTAMPDIFF(unit,start,end)` returns `end-start` in complete requested units.
- Functions on an indexed column in a predicate often make the predicate non-sargable.

## Rapid function reference

### Strings

| Function | Return | Important `NULL`/edge behavior |
|---|---|---|
| `LOWER(s)`, `UPPER(s)` | Case-converted string | Null input → `NULL`; collation affects comparisons |
| `LEFT(s,n)`, `RIGHT(s,n)` | Up to `n` characters | Null argument → `NULL` |
| `TRIM(s)`, `LTRIM(s)`, `RTRIM(s)` | Trimmed string | Null input → `NULL`; `TRIM` does not remove arbitrary inner spaces |
| `REPLACE(s,from,to)` | Replaced string | Any required null argument → `NULL`; case-sensitive matching |
| `INSTR(s,sub)`, `LOCATE(sub,s)` | 1-based position; `0` if absent | Null argument → `NULL` |
| `LPAD(s,n,pad)`, `RPAD(s,n,pad)` | Exactly/truncated to `n` characters | Null argument → `NULL` |
| `REVERSE(s)`, `REPEAT(s,n)` | Transformed string | Null argument → `NULL` |

### Numbers

| Function | Return | Important `NULL`/edge behavior |
|---|---|---|
| `ABS(x)` | Absolute value | Null → `NULL` |
| `ROUND(x,d)` | Rounded to `d` decimal places | Null → `NULL`; exact/approximate inputs can round differently |
| `CEIL(x)`, `FLOOR(x)` | Nearest integer upward/downward | Null → `NULL` |
| `MOD(a,b)` or `a % b` | Remainder | Null operand or zero divisor → `NULL` |
| `POWER(a,b)`, `SQRT(x)` | Power / square root | Null → `NULL`; domain/overflow rules matter |

### Dates

| Function | Return | Important `NULL`/edge behavior |
|---|---|---|
| `CURDATE()`, `CURTIME()`, `NOW()` | Current session date/time | No nullable input; current-time functions are stable within one statement |
| `DATE(x)`, `TIME(x)`, `YEAR(x)` | Extracted component | Null → `NULL`; invalid-date handling depends on SQL mode |
| `DATE_ADD`, `DATE_SUB` | Shifted temporal value | Null date → `NULL`; month-end adjustment can occur |
| `LAST_DAY(d)` | Last date in month | Null/invalid date → `NULL` |
| `DATE_FORMAT(d,fmt)` | Formatted text | Null required argument → `NULL` |
| `STR_TO_DATE(s,fmt)` | Parsed temporal value | Null → `NULL`; format mismatch yields `NULL` plus warning; invalid ranges can error in MySQL 8.4, while zero/incomplete-date acceptance depends on SQL mode |

## Examples

### Null-safe full name

```sql
SELECT CONCAT_WS(' ', first_name, middle_name, last_name) AS full_name
FROM employees;
```

**Expected behavior:** Missing name components are skipped; the separator is not repeated for them.

### Age as completed years

```sql
SELECT TIMESTAMPDIFF(YEAR, birth_date, CURDATE()) AS age
FROM employees;
```

**Expected behavior:** `NULL` for a missing `birth_date`; otherwise completed years.

## Tricky parts

- Confusing bytes with characters.
- Assuming `CONCAT` skips nulls.
- Reversing arguments to `DATEDIFF`/`TIMESTAMPDIFF`.
- Comparing `YEAR(date_col)=2026` instead of a range when index use matters.
- Using `RAND()` to sort a large table.

## Interview checks

1. `LENGTH` versus `CHAR_LENGTH`?
2. How does `CONCAT` behave with one null argument?
3. Difference between `DATEDIFF` and `TIMESTAMPDIFF`?

## 30-second recap

- Recall common string, numeric, and date functions together with their return and null behavior.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Built-in MySQL Functions](https://dev.mysql.com/doc/refman/8.4/en/function-reference.html)
