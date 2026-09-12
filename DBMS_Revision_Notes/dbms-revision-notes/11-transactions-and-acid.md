# 11 — Transactions and ACID

> Goal: define transaction guarantees precisely and design correct transaction boundaries.

## 1. Transaction mental model

A **transaction** is a logical unit of work that the DBMS treats according to its atomicity, consistency, isolation, and durability guarantees.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;
```

If the second update fails, the application should roll back rather than expose a half-transfer.

## 2. ACID—precise meanings

| Property | Meaning | Common misconception |
|---|---|---|
| Atomicity | All transactional changes commit or none take effect | It does not mean the code executes as one CPU instruction |
| Consistency | A correct transaction takes the DB from one valid state to another | The DBMS cannot infer every business rule automatically |
| Isolation | Concurrent execution behaves according to the chosen isolation guarantee | It does not always mean full serial execution |
| Durability | Once commit succeeds, effects survive failures covered by the DB guarantee | It does not replace backup or disaster recovery |

Consistency is shared responsibility: constraints and the DBMS enforce declared rules; application logic must implement the rest.

## 3. Transaction states

Typical conceptual states:

- **Active:** executing operations.
- **Partially committed:** final statement executed but commit is not yet durable.
- **Committed:** commit completed.
- **Failed:** cannot proceed.
- **Aborted:** effects rolled back; it may be retried or terminated.

Exact internal state names vary by DBMS.

## 4. Transaction boundaries

- `BEGIN`/`START TRANSACTION` opens an explicit transaction.
- `COMMIT` makes its changes permanent.
- `ROLLBACK` cancels its uncommitted changes.
- `SAVEPOINT` allows partial rollback within the transaction.
- **Autocommit** commonly treats each statement as its own transaction unless an explicit transaction is open.

DDL transactional behavior varies by engine. Never assume all schema operations roll back everywhere.

## 5. Atomicity and durability mechanisms

Atomicity and durability are supported by recovery mechanisms such as write-ahead logging, undo/redo information, and controlled buffer flushing. The log describing a change must become durable before a related dirty data page is written under WAL.

Commit typically means the required commit record/log state has reached the durability boundary—not that every modified data page has been written immediately.

## 6. Choosing a transaction boundary

Place all database operations that must succeed/fail together in one transaction, but keep it as short as correctness permits.

Avoid while holding a transaction open:

- waiting for user input;
- slow external API calls;
- large CPU work unrelated to the DB;
- unnecessary reads or network round trips.

Long transactions retain locks or old row versions, increase contention, delay cleanup, and make recovery/replication harder.

## 7. External side effects

A database transaction usually cannot atomically roll back an email, HTTP request, or message sent to an unrelated system.

For “update DB and publish event,” common approaches include:

- **Transactional outbox:** write business state and an outbox row in one DB transaction; a relay publishes later.
- **Idempotent consumer:** safely process repeated delivery.
- **Saga/compensation:** coordinate a multi-step distributed workflow without pretending it is one local transaction.

## 8. Idempotency is not atomicity

- **Atomic:** partial effects are not committed.
- **Idempotent:** repeating the operation has the same intended effect as applying it once.

A payment request may use an idempotency key with a unique constraint. That handles client retries; the transaction still needs atomic updates.

```sql
INSERT INTO payment_requests(idempotency_key, status)
VALUES (?, 'STARTED');
-- unique(idempotency_key) prevents duplicate request creation
```

## 9. Optimistic application pattern

A version column can prevent silent lost updates:

```sql
UPDATE documents
SET body = ?, version = version + 1
WHERE id = ? AND version = ?;
```

If affected rows = 0, another writer changed the row; reload or retry. This is application-level optimistic concurrency, not a replacement for DB transaction semantics.

## 10. Transaction failure and retry

Transactions can abort because of deadlocks, serialization failures, timeouts, constraint violations, or infrastructure failures.

A safe retry policy needs:

- a clearly retryable error class;
- bounded attempts with backoff/jitter;
- an idempotent surrounding operation;
- a fresh transaction per retry;
- observability so retry storms are visible.

After many DB errors, the current transaction is invalid until rollback; continuing statements may fail.

## 11. The uncertain commit problem

If the client loses its connection after sending `COMMIT`, it may not know whether the server committed. Blindly repeating a non-idempotent business action can duplicate it.

Use a stable request/idempotency identifier and query the recorded outcome. “Network error” does not imply rollback.

## 12. Common traps

- ACID consistency is not the same as CAP consistency.
- Isolation level names do not guarantee identical behavior across DBMSs.
- A transaction covering one DB does not automatically cover Redis, Kafka, or an HTTP service.
- Rollback does not undo already-observed external side effects.
- Durability does not mean zero data loss under every catastrophe; check replication/fsync/storage guarantees.
- `SELECT`, compute in application, then `UPDATE` can lose an update without locking, atomic SQL, or version checking.
- Holding locks while calling another service is a contention and deadlock hazard.

## Interview checks

1. Explain ACID using a money transfer without vague definitions.
2. When can a committed transaction’s pages still be absent from the main data file?
3. How would you atomically update an order and later publish `OrderCreated`?
4. What happens if the client times out during commit?
5. Why should deadlock retries wrap the complete transaction?

## 60-second recall

- Transaction = smallest all-or-nothing database unit.
- Atomicity uses undo/recovery; durability uses durable log/redo.
- Isolation is a spectrum governed by implementation and level.
- Keep transactions correct and short.
- DB + external message: outbox, idempotency, or workflow coordination.
- Unknown commit outcome requires request identity, not blind replay.

## Sources for deeper revision

- [PostgreSQL: Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [PostgreSQL: WAL](https://www.postgresql.org/docs/current/wal-intro.html)
- [Berkeley CS 186](https://cs186berkeley.net/sp26/)

