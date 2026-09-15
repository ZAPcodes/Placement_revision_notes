# 13 — Inserting Data

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Insert rows, copy query results, and handle duplicate keys without confusing update and replacement semantics.

## Syntax template

```sql
INSERT INTO table_name (c1, c2)
VALUES (v1, v2), (v3, v4);

INSERT INTO target (c1, c2)
SELECT x, y FROM source;

INSERT INTO t (id, value) VALUES (1, 'x')
AS new
ON DUPLICATE KEY UPDATE value = new.value;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Plain `INSERT` | Creates rows or errors. | Explicit null accepted only if column permits it. |
| Omitted column | Uses default, generated/auto value, or possibly errors. | Not automatically equivalent to explicit `NULL`. |
| `INSERT IGNORE` | Skips/adjusts certain ignorable errors with warnings. | A forbidden null may be adjusted to implicit default depending on mode/context. |
| Upsert | Insert, or update row causing unique/primary conflict. | Incoming null can overwrite unless guarded. |
| `REPLACE` | Insert, deleting conflicting old row first when needed. | Delete/insert semantics affect triggers, FKs, and auto-increment. |
| `LAST_INSERT_ID()` | Session-specific generated auto-increment value. | Returns `0` if no applicable value in the session. |

## Revision notes

- Always name columns so schema changes do not silently reorder values.
- Multi-row insert is usually more efficient than many single-row statements.
- An explicit `NULL` can trigger an auto-increment value for an auto-increment column, but do not generalize this to ordinary defaults.
- `INSERT IGNORE` can hide data-quality problems; inspect warnings.
- `ON DUPLICATE KEY UPDATE` can be ambiguous when multiple unique indexes conflict.
- `REPLACE` is not an update. It can delete the old row and insert a new one.

## Examples

### Bulk insert

```sql
INSERT INTO departments (department_id, name)
VALUES (10, 'Engineering'),
       (20, 'Finance');
```

**Expected behavior:** Two rows inserted atomically as one statement unless an error aborts it.

### Null-preserving upsert choice

```sql
INSERT INTO profiles (user_id, display_name)
VALUES (7, NULL) AS new
ON DUPLICATE KEY UPDATE
  display_name = COALESCE(new.display_name, profiles.display_name);
```

**Expected behavior:** On conflict, a null incoming name does not erase the stored name.

## Tricky parts

- Omitting the column list.
- Treating `INSERT IGNORE` as universal error handling.
- Assuming `REPLACE` preserves the original row identity/effects.
- Overwriting known values with null during an upsert.
- Using deprecated `VALUES(col)` patterns instead of a row alias in new code.

## Interview checks

1. `REPLACE` versus upsert?
2. What is the difference between omitting a column and inserting null?
3. Why inspect warnings after `INSERT IGNORE`?

## 30-second recap

- Insert rows, copy query results, and handle duplicate keys without confusing update and replacement semantics.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Inserting Data](https://dev.mysql.com/doc/refman/8.4/en/insert.html)
