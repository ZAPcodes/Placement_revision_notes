# 15 — Database and Table Definition Queries

> **Priority:** Reference  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Create and evolve schemas while understanding defaults, generated columns, and implicit commits.

## Syntax template

```sql
CREATE TABLE employees (
  employee_id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  email VARCHAR(255) NOT NULL UNIQUE,
  salary DECIMAL(12,2) NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

ALTER TABLE employees ADD COLUMN active BOOLEAN NOT NULL DEFAULT TRUE;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `CREATE` | Creates an object or errors if it exists. | Column nullability comes from definition. |
| `DEFAULT` | Used when column is omitted or `DEFAULT` is specified. | Explicit null usually remains null or errors; it does not generally request the default. |
| `AUTO_INCREMENT` | Generates a numeric key. | Explicit null commonly requests generation. |
| Generated column | Expression-derived value. | Usually null when its expression null-propagates. |
| `TRUNCATE` | Quickly empties table and resets auto-increment behavior. | No row predicate; causes implicit commit. |
| `DROP` | Removes object definition and data. | Causes implicit commit. |

## Revision notes

- Choose the narrowest correct data type but leave realistic growth room.
- Use `utf8mb4` for full Unicode; collation controls comparison and sort semantics.
- `CREATE TABLE ... SELECT` copies query data but does not copy all indexes/constraints/defaults.
- `CREATE TABLE ... LIKE` copies structure more faithfully but not data.
- Many DDL statements cause an implicit commit before and/or after execution.
- Online DDL capabilities vary by operation; production migrations require operational planning.

## Examples

### Generated total

```sql
CREATE TABLE invoice_lines (
  qty INT NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL,
  total DECIMAL(12,2)
    GENERATED ALWAYS AS (qty * unit_price) STORED
);
```

**Expected behavior:** `total` is computed automatically and cannot be supplied as an arbitrary stored value.

### Add nullable then backfill

```sql
ALTER TABLE employees ADD COLUMN nickname VARCHAR(100) NULL;
-- backfill safely, then optionally enforce NOT NULL later
```

**Expected behavior:** A migration-friendly two-phase pattern.

## Tricky parts

- Assuming explicit null invokes a default.
- Using `VARCHAR` without considering character set byte size.
- Expecting `CREATE TABLE ... SELECT` to copy keys.
- Running DDL inside a transaction expecting normal rollback.
- Confusing `TRUNCATE` with `DELETE`.

## Interview checks

1. `DELETE` versus `TRUNCATE` versus `DROP`?
2. `DATETIME` versus `TIMESTAMP`?
3. What is copied by `CREATE TABLE ... SELECT`?

## 30-second recap

- Create and evolve schemas while understanding defaults, generated columns, and implicit commits.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Database and Table Definition Queries](https://dev.mysql.com/doc/refman/8.4/en/data-definition-statements.html)
