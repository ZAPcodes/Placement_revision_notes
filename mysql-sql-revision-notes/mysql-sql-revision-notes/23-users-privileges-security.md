# 23 — Users, Privileges, and Security Queries

> **Priority:** Reference  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Apply least privilege and prevent injection through parameterized values and controlled identifiers.

## Syntax template

```sql
CREATE USER 'app_user'@'10.%' IDENTIFIED BY 'strong-secret';
GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'app_user'@'10.%';
SHOW GRANTS FOR 'app_user'@'10.%';
REVOKE UPDATE ON app_db.* FROM 'app_user'@'10.%';
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| Account | Combination of user name and host. | User/host are not nullable SQL values. |
| `GRANT` | Adds named privileges. | No query result set; success/status metadata. |
| `REVOKE` | Removes named privileges. | No effect from data nulls. |
| Prepared value placeholder | Binds a value separately from SQL text. | Can safely bind SQL null as a value. |
| Identifier | Table/column syntax element. | Cannot normally be bound with a value placeholder; allowlist it. |
| Definer object | May execute with definer privileges. | Orphan/overprivileged definers are operational risks. |

## Revision notes

- Grant only required actions on required schemas/tables; separate read-only, write, migration, and admin roles.
- Use application-driver prepared statements. Do not build SQL by concatenating untrusted input.
- Placeholders protect values, not SQL keywords, table names, column names, or sort direction. Map those from allowlists.
- Do not store production passwords in SQL scripts or repositories; examples use placeholders only.
- Audit `SHOW GRANTS`, role membership, and stored-object definers.
- Restrict network origin through the MySQL account host component and infrastructure controls.

## Examples

### Safe value query

```sql
SELECT order_id FROM orders WHERE customer_id = ? AND status = ?;
```

**Expected behavior:** The driver binds typed values; a bound null follows SQL null comparison rules, so generate `IS NULL` when that is the intended predicate.

### Allowlisted ordering concept

```sql
-- application maps input "newest" to this fixed fragment
ORDER BY ordered_at DESC, order_id DESC
```

**Expected behavior:** User input never becomes raw SQL syntax.

## Tricky parts

- Binding null to `col = ?` and expecting null rows.
- Using placeholders for identifiers.
- Granting global privileges to an application.
- Committing credentials.
- Ignoring definer privilege drift.

## Interview checks

1. Why does a prepared statement prevent value injection?
2. Why can identifiers not be bound like values?
3. What does least privilege mean for an application account?

## 30-second recap

- Apply least privilege and prevent injection through parameterized values and controlled identifiers.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Users, Privileges, and Security Queries](https://dev.mysql.com/doc/refman/8.4/en/access-control.html)
