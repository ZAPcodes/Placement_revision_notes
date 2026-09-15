# 17 — Transactions and Concurrency Queries

> **Priority:** Core  
> **Dialect:** MySQL 8.4 LTS  
> **Purpose:** Fast placement revision; examples favor clarity and interview correctness.

## What you must know

Protect multi-step invariants using transactions, isolation, and locking reads.

## Syntax template

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;

SELECT balance
FROM accounts
WHERE account_id = 1
FOR UPDATE;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;
```

## Keyword and `NULL` behavior

| Construct | Returns / effect | When `NULL` is involved |
|---|---|---|
| `START TRANSACTION` | Begins explicit transaction. | Does not change stored nulls. |
| `COMMIT` | Makes transaction changes durable/visible per engine rules. | — |
| `ROLLBACK` | Undoes uncommitted transactional changes. | — |
| `SAVEPOINT` | Marks partial rollback point. | — |
| `FOR UPDATE` | Returns rows and takes exclusive-style row locks. | Rows not found are not returned; range/gap locking depends on plan/isolation. |
| `FOR SHARE` | Returns rows and takes shared locks. | Selected null column values remain null. |
| `SKIP LOCKED` | Returns only immediately available rows. | Result is intentionally incomplete. |

## Revision notes

- InnoDB default isolation is commonly `REPEATABLE READ`; verify rather than assume.
- Use a single atomic `UPDATE ... WHERE ...` when possible instead of read-then-write.
- Lock rows in a consistent order to reduce deadlocks; still retry deadlock victims.
- `NOWAIT` fails immediately on a conflicting lock; `SKIP LOCKED` is useful for queues, not general consistent reporting.
- DDL and various administrative statements cause implicit commits.
- A transaction does not make application side effects such as emails roll back.

## Examples

### Atomic stock decrement

```sql
UPDATE products
SET stock = stock - 1
WHERE product_id = 7
  AND stock > 0;
```

**Expected behavior:** Affected rows `1` means success; `0` means unavailable/not found without a race-prone preliminary read.

### Partial rollback

```sql
START TRANSACTION;
SAVEPOINT before_optional;
-- optional work
ROLLBACK TO SAVEPOINT before_optional;
COMMIT;
```

**Expected behavior:** Keeps work before the savepoint and discards work after it.

## Tricky parts

- Holding transactions open during user interaction.
- Assuming deadlocks never occur.
- Using `SKIP LOCKED` for a complete report.
- Mixing nontransactional side effects without an outbox/idempotency plan.
- Expecting DDL rollback like ordinary DML.

## Interview checks

1. Lost update and how to prevent it?
2. `FOR UPDATE` versus `FOR SHARE`?
3. Why must deadlocks be retried?

## 30-second recap

- Protect multi-step invariants using transactions, isolation, and locking reads.
- State the output grain before writing the query.
- Decide explicitly how missing values, ties, duplicates, and empty inputs should behave.
- Add deterministic ordering whenever row choice or pagination depends on order.

## Official reference

- [MySQL 8.4 Reference Manual — Transactions and Concurrency Queries](https://dev.mysql.com/doc/refman/8.4/en/innodb-transaction-model.html)
