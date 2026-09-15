# 19 — Stored SQL Objects

> **Priority:** Reference  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Recognize routines, triggers, cursors, handlers, and events without turning interview preparation into server administration.

## Syntax template

```sql
DELIMITER //
CREATE PROCEDURE raise_salary(IN p_id BIGINT, IN p_pct DECIMAL(5,2))
BEGIN
  UPDATE employees
  SET salary = salary * (1 + p_pct / 100)
  WHERE employee_id = p_id;
END//
DELIMITER ;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Procedure | Invoked with `CALL`; may emit result sets and OUT values. | Input null is allowed unless routine logic rejects it. |
| Stored function | Returns exactly one value. | May return null. |
| Trigger | Runs per affected row at defined timing/event. | Use `OLD`/`NEW`; null rules still apply. |
| Cursor `FETCH` | Copies next row into variables. | Fetched column null becomes variable null. |
| `NOT FOUND` handler | Runs when cursor/query finds no row. | Signals absence; do not confuse with a row containing null. |
| Event | Runs scheduled SQL when event scheduler is enabled. | Null behavior depends on its statements. |

## Revision notes

- `DELIMITER` is a client command, not server SQL; it lets clients send stored-program bodies containing semicolons.
- Declare local variables, conditions, cursors, and handlers near the beginning of a block in required order.
- Triggers are row-level in MySQL and can hide important side effects; keep them small and documented.
- A `SELECT ... INTO` finding no row raises a not-found condition; a row whose selected value is null is still a found row.
- Prefer set-based SQL over cursors where practical.
- Dynamic SQL uses `PREPARE`, `EXECUTE`, and placeholders; identifiers cannot be bound as value parameters.

## Examples

### Null-aware routine guard

```sql
IF p_pct IS NULL OR p_pct < 0 THEN
  SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid percent';
END IF;
```

**Expected behavior:** Raises an application error for missing or negative input.

### Audit trigger concept

```sql
CREATE TRIGGER employee_salary_audit
AFTER UPDATE ON employees
FOR EACH ROW
INSERT INTO salary_audit(employee_id, old_salary, new_salary)
VALUES (NEW.employee_id, OLD.salary, NEW.salary);
```

**Expected behavior:** Writes one audit row per updated employee; null old/new salaries are recorded if permitted.

## Tricky parts

- Confusing `DELIMITER` with SQL syntax.
- Using `= NULL` inside routines.
- Cursor loops without a not-found handler.
- Business logic hidden across many triggers.
- String-concatenating untrusted values into dynamic SQL.

## Interview checks

1. Procedure versus function?
2. What does a not-found handler mean?
3. Why prefer set-based SQL over a cursor?

## 30-second recap

- Recognize routines, triggers, cursors, handlers, and events without turning interview preparation into server administration.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Stored SQL Objects](https://dev.mysql.com/doc/refman/8.4/en/stored-objects.html)
