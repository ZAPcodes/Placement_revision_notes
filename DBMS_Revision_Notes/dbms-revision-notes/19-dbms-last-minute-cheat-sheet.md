# 19 — DBMS Last-Minute Cheat Sheet

> Read this in 15–20 minutes before an OA/interview. If a line feels vague, jump to the matching numbered note.

## 1. Core vocabulary

| Term | Recall |
|---|---|
| Schema / instance | Definition / data at a moment |
| Degree / cardinality | Number of attributes / number of tuples (relational-theory usage) |
| Physical / logical independence | Change storage / change conceptual design without disrupting higher layers |
| OLTP / OLAP | Short concurrent operations / analytical scans and aggregation |
| Row / column store | Whole rows together / values of a column together |

## 2. ER and keys

- Entity = object; relationship = association; weak entity needs owner key + partial key.
- Cardinality = maximum (`1:1`, `1:N`, `M:N`); participation = minimum (optional/mandatory).
- `1:N`: foreign key normally on N side. `M:N`: junction table. Multivalued attribute: separate table.
- Superkey uniquely identifies; candidate key is minimal; primary key is selected candidate; alternate keys remain candidates.
- Surrogate key does **not** replace business uniqueness constraints.
- Entity integrity: primary key unique/non-null. Referential integrity: foreign key matches a permitted referenced key or is null.
- Foreign keys are not automatically indexed in every DBMS.

## 3. Relational model and algebra

- Selection `σ`: rows. Projection `π`: columns. Join: matching tuples. Division: “for all.”
- Cartesian product needs a later predicate to become a meaningful join.
- Algebra is procedural; calculus is declarative. Safe relational calculus and algebra have equivalent expressive power.
- Classical relations are sets; SQL tables/results commonly use bag semantics unless `DISTINCT`.

## 4. SQL order and NULL

```text
FROM/JOIN → WHERE → GROUP BY → aggregate → HAVING
→ window → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET
```

- `WHERE` filters rows; `HAVING` filters groups.
- `COUNT(*)` counts rows; `COUNT(x)` ignores null `x`.
- `NULL` comparisons produce `UNKNOWN`; use `IS NULL`.
- `NOT IN` + any `NULL` can surprise; prefer a correct `NOT EXISTS`.
- `UNION` deduplicates; `UNION ALL` keeps duplicates and is usually cheaper.
- Right-table predicate in `WHERE` after `LEFT JOIN` can remove null-extended rows; put match conditions in `ON` when appropriate.

## 5. Window functions

| Function | Tie behavior |
|---|---|
| `ROW_NUMBER()` | Unique sequence; ties broken by ordering |
| `RANK()` | Same rank for ties, then gaps |
| `DENSE_RANK()` | Same rank for ties, no gaps |

- Window functions retain rows; `GROUP BY` collapses them.
- `PARTITION BY` resets the window; `ORDER BY` determines sequence.
- Specify a deterministic tie-breaker and explicit frame when results depend on it.
- Running total: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.
- Top N per group: `ROW_NUMBER()` partitioned by group, then filter rank.

## 6. Functional dependencies and normalization

- `X → Y`: equal `X` implies equal `Y` in every valid instance.
- Closure `X+`: attributes implied by `X`; if it covers all attributes, `X` is a superkey.
- Armstrong: reflexivity, augmentation, transitivity.
- Minimal cover: singleton RHS, remove extraneous LHS attributes, remove redundant FDs.

| NF | Fast test |
|---|---|
| 1NF | Atomic domains/no repeating groups |
| 2NF | 1NF + no partial dependency of non-prime attribute on candidate key |
| 3NF | Every `X→A`: `X` superkey or `A` prime |
| BCNF | Every nontrivial determinant is a superkey |
| 4NF | Every nontrivial MVD determinant is a superkey |
| 5NF | No nontrivial join dependency except those implied by keys |

- BCNF is stronger than 3NF but may lose dependency preservation.
- Good decomposition goals: lossless join + preferably dependency preservation.
- Normalize for correctness; denormalize deliberately after measurement.

## 7. Storage and buffer pool

- Page = primary I/O/cache unit; record = logical row representation.
- Slotted page supports variable records with stable slot IDs.
- Heap: fast inserts; sorted: ranges/order but expensive maintenance; hash: equality.
- Buffer pool: page table, frames, pin count, dirty bit, replacement, flushing.
- Dirty = changed in memory since durable page version.
- Sequential access is friendly to prefetch/cache; random page reads are costly.
- DB buffer pool and OS page cache are different layers; designs may bypass or coordinate them.

## 8. Index decision table

| Need | Likely candidate |
|---|---|
| Equality + range/order | B+ tree |
| Equality only | Hash or B+ tree |
| Rows in physical key order | Clustered organization |
| Avoid table lookup | Covering index |
| Small important subset | Partial/filtered index |
| Query on computed expression | Expression index |

- B+ tree: high fan-out, shallow, linked leaves.
- `(a,b,c)` naturally supports leading-column searches; order matters.
- Equality columns usually precede first range column.
- Covering is query-relative and increases size/write cost.
- Selectivity = fraction matched; low matched fraction usually favors index access.
- More indexes → slower writes, more storage/WAL/cache pressure.
- Primary key constraint and clustered storage are not universally identical.

### Why an index is ignored

Large result fraction, tiny table, stale/wrong stats, non-sargable function/cast, leading wildcard, mismatched composite order, costly random lookups, or parameter skew.

## 9. Query optimization

```text
SQL → parse/bind → logical plan → rewrite → cost optimization → physical execution
```

| Join | Best mental case |
|---|---|
| Index nested loop | Small/selective outer + inner index |
| Hash | Large equi-join; enough memory |
| Sort-merge | Ordered inputs or order useful; large joins |

- Push filters/projections where semantics allow; optimizer chooses join order.
- Cardinality errors cascade into wrong access path, join order/type, and memory.
- Stats: row count, distinct count, null fraction, histograms/common values, correlation.
- Sargable: index-search-friendly predicate; rewrite functions into ranges when possible.
- In an actual plan check estimated vs actual rows, loops, filters, spills, buffers/I/O.
- First major row-estimate divergence is often more informative than the slow final node.

## 10. Transactions and ACID

| Letter | Precise meaning |
|---|---|
| A | All effects commit or none |
| C | Correct transaction preserves declared/business invariants |
| I | Concurrent behavior follows selected isolation guarantee |
| D | Acknowledged commit survives covered failures |

- Keep transactions as short as correctness permits.
- Avoid external API calls/user waits while locks/versions are held.
- Savepoint = partial rollback point, not a separate durable transaction.
- Autocommit commonly makes each statement a transaction.
- Atomicity ≠ idempotency. Retried APIs often need both.
- DB state + external message: transactional outbox + idempotent relay/consumer.
- Connection loss during commit = possibly unknown outcome; use request identity.

## 11. Schedules and anomalies

- Conflict: different transactions + same item + at least one write.
- Precedence graph acyclic **iff conflict-serializable**; topological order gives serial order.
- View-serializable is broader; blind writes create cases not conflict-serializable.
- `Strict ⇒ Cascadeless ⇒ Recoverable` (not conversely).

| Anomaly | Recall |
|---|---|
| Dirty read/write | Observe/overwrite uncommitted data |
| Non-repeatable read | Same row later has different committed value |
| Phantom | Predicate result set changes |
| Lost update | One same-value update overwrites another |
| Read skew | Related values from inconsistent times |
| Write skew | Disjoint writes violate an invariant after common snapshot |

Serializable and recoverable are different properties.

## 12. Locks, deadlocks, and MVCC

- `S` compatible with `S`; `X` conflicts with `S` and `X`.
- Intention locks coordinate multigranularity (`IS`, `IX`, `SIX`).
- 2PL: acquire in growing phase, release in shrinking phase → conflict serializable.
- Strict 2PL holds write locks until commit/abort → strict schedule.
- Deadlock = wait-for cycle; handle by detection/victim, prevention, or timeout.
- Wait-die: older waits, younger dies. Wound-wait: older wounds, younger waits.
- Consistent lock order + short transactions reduce, not eliminate, deadlocks.
- Retry complete transaction with bounded backoff.

### MVCC

- Snapshot chooses visible row versions; ordinary readers often avoid blocking writers.
- Writers still conflict; locks remain; old versions require cleanup.
- Long snapshots delay reclamation.
- Snapshot isolation can permit write skew.
- Serializable may detect dependency danger and abort; applications must retry.
- Isolation-level names/behavior differ across DBMSs—state the engine.

## 13. Recovery

- WAL rule: log change durable before dirty data page; commit log durable before success acknowledgement.
- **Steal ⇒ UNDO. No-force ⇒ REDO.** Common: steal + no-force.
- Checkpoint bounds recovery work; fuzzy checkpoint does not flush everything.
- ARIES: **analysis → redo (“repeat history”) → undo losers**.
- CLR records undo progress; redoable, not undone again.
- Group commit shares log flush latency across transactions.
- PITR = base backup + archived log replay to time/LSN.
- Replication supports availability; backup/PITR supports historical recovery. One does not replace the other.

## 14. Ten answers that separate strong candidates

1. “It depends on the DBMS and execution plan” followed by the exact dependency—not used as an escape.
2. Indexes optimize access patterns but tax writes, storage, WAL, and cache.
3. A foreign-key constraint does not universally create its own index.
4. Candidate keys preserve business uniqueness even when a surrogate PK is used.
5. SQL `NULL` uses three-valued logic; `NOT IN` is a classic consequence.
6. Serializable execution can be concurrent and may abort/retry.
7. MVCC reduces read/write blocking; it is not lock-free or automatically serializable.
8. Most disastrous plan choices begin with a bad row estimate.
9. Database transactions do not automatically include an external service.
10. Replication is not backup, and backup is unproven until restore is tested.

## 15. Project answer template

For any database choice or optimization, answer in this order:

1. **Requirement:** query/write pattern, scale, latency, consistency, failure tolerance.
2. **Choice:** schema, DB, index, or transaction mechanism.
3. **Why:** which requirement it satisfies.
4. **Trade-off:** what became more expensive or complex.
5. **Evidence:** dataset/load, plan/metrics, p95/p99, failure test.
6. **Next limit:** the bottleneck or risk that remains.

## Final self-test

Can you explain, without notes:

- why `(tenant_id, created_at)` fits “recent events for one tenant”;
- why a left join can accidentally become an inner join;
- a BCNF versus 3NF trade-off;
- index nested loop versus hash join;
- write skew under snapshot isolation;
- unknown commit outcome and idempotency;
- steal/no-force and the ARIES phases;
- why a Redis/Cassandra/MongoDB choice fits your project’s exact access patterns?

If any answer takes more than 90 seconds or lacks a trade-off, revise that source file once more.

