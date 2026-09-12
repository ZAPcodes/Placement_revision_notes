# 12 — Schedules, Serializability, and Anomalies

> Goal: analyze interleavings correctly and separate serializability, recoverability, and isolation anomalies.

## 1. Schedule vocabulary

- **Schedule/history:** ordering of operations from one or more transactions, preserving each transaction’s internal order.
- **Serial schedule:** one transaction finishes before the next begins.
- **Non-serial schedule:** operations interleave.
- **Serializable schedule:** non-serial but equivalent to some serial execution under a stated equivalence notion.

Concurrency improves utilization and throughput, but unsafe interleavings can violate correctness.

## 2. Conflicting operations

Two operations conflict when they:

1. belong to different transactions;
2. access the same data item; and
3. at least one is a write.

| Pair | Conflict? |
|---|---|
| `R1(X), R2(X)` | No |
| `R1(X), W2(X)` | Yes |
| `W1(X), R2(X)` | Yes |
| `W1(X), W2(X)` | Yes |

Nonconflicting adjacent operations can be swapped without changing conflict behavior.

## 3. Conflict serializability

A schedule is conflict-serializable if it can be transformed into a serial schedule by swapping nonconflicting operations.

### Precedence/serialization graph test

- Create one node per transaction.
- Add edge `Ti → Tj` if an operation of `Ti` occurs before and conflicts with an operation of `Tj` on the same item.
- The schedule is conflict-serializable **iff the graph is acyclic**.
- A topological ordering gives an equivalent serial order.

### Tiny example

```text
R1(X), W1(X), R2(X), W2(X)
```

Conflicts on `X` add `T1 → T2`; no reverse edge, so it is conflict-serializable as `T1, T2`.

## 4. View serializability

Two schedules are view-equivalent when:

- every read obtains its value from the same transaction (or the initial value); and
- the same transaction performs the final write on each item.

View serializability includes all conflict-serializable schedules and can also include schedules with blind writes. Testing it is harder; practical protocols generally target conflict serializability.

**Trap:** “Serializable” and “conflict-serializable” are not logically identical terms, even though interview problems often intend the latter.

## 5. Recoverability properties

These are about commit/abort safety, not just serial equivalence.

### Recoverable schedule

If `Tj` reads a value written by `Ti`, then `Tj` commits only after `Ti` commits.

### Cascadeless schedule

Transactions read only committed values. If `Tj` reads from `Ti`, then `Ti` must already have committed before the read. Avoids cascading rollback.

### Strict schedule

After a transaction writes `X`, no other transaction may read **or write** `X` until the writer commits/aborts. Simplifies recovery.

```text
Strict ⇒ Cascadeless ⇒ Recoverable
```

The converses do not necessarily hold.

## 6. Major concurrency anomalies

| Anomaly | What happens |
|---|---|
| Dirty read | Read data written by an uncommitted transaction |
| Dirty write | Overwrite data written by an uncommitted transaction |
| Non-repeatable read | Re-reading a row returns a newer committed value |
| Phantom | Re-running a predicate query returns a different matching row set |
| Lost update | One write silently overwrites another transaction’s update |
| Read skew | Related values are observed from inconsistent points in time |
| Write skew | Transactions read a shared condition, then update different rows and violate it |

## 7. Lost update example

```text
Initial balance = 100
T1 reads 100
T2 reads 100
T1 writes 80
T2 writes 130
```

T1’s deduction disappears. Remedies include an atomic update (`balance = balance - 20`), row locking, version checking, or an appropriate serializable implementation.

## 8. Phantom vs non-repeatable read

- Non-repeatable read: an existing row’s value changes or disappears between reads.
- Phantom: the set of rows satisfying a predicate changes, commonly due to insert/delete or an update entering/leaving the range.

Preventing phantoms is about predicate/range behavior, not merely locking previously returned rows.

## 9. Write skew—the favorite tricky case

Rule: at least one of two doctors must remain on call.

1. T1 reads both rows and sees two doctors on call.
2. T2 reads the same valid state.
3. T1 turns doctor A off.
4. T2 turns doctor B off.
5. They update different rows, so simple write-write conflict detection may not stop both.

The final state violates the rule. Snapshot-style isolation can permit this; true serializable mechanisms, explicit locking of the logical constraint, or schema redesign may prevent it.

## 10. Serializability vs serial execution

Serializable execution need not run transactions one at a time. A DBMS may interleave or run them in parallel as long as the result obeys its serializable guarantee. Concurrency control discovers/prevents dangerous dependencies.

## 11. Serializability vs recoverability

A schedule can be conflict-serializable but unrecoverable if a reader commits before the transaction whose uncommitted value it read. Conversely, recoverability alone does not imply serializability.

Treat them as separate axes:

- **Serializability:** equivalence/correctness of concurrent outcome.
- **Recoverability:** safe commit and abort dependencies.

## 12. Common traps

- Conflicts require the same data item and at least one write.
- Preserve transaction-internal order when analyzing a schedule.
- Acyclic graph ⇒ conflict-serializable; a cycle ⇒ not conflict-serializable.
- A graph can have multiple topological orders and therefore multiple equivalent serial orders.
- Strictness does not by itself prove serializability.
- Repeatable read names differ by DBMS; reason from documented anomalies/implementation.
- MVCC can eliminate dirty/non-repeatable reads yet still allow write skew.
- Serializable transactions may abort and require retry; correctness is not “free waiting.”

## Interview checks

1. Build a precedence graph from a schedule and give every valid serial order.
2. Can a serializable schedule be unrecoverable? Why?
3. Differentiate dirty read, non-repeatable read, and phantom with examples.
4. Why does row-level locking not automatically prevent every phantom?
5. Explain write skew and why it differs from a lost update.

## 60-second recall

- Conflict: different transactions, same item, one write.
- Precedence graph acyclic ⇔ conflict-serializable.
- View serializability is broader but harder to test.
- Strict ⇒ cascadeless ⇒ recoverable.
- Serializable and recoverable answer different questions.
- Write skew is the key snapshot-isolation trap.

## Sources for deeper revision

- [PostgreSQL: Transaction Isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [Berkeley CS 186](https://cs186berkeley.net/sp26/)
- [CMU 15-445 course schedule](https://15445.courses.cs.cmu.edu/fall2025/schedule.html)

