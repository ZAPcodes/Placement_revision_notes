# 18 — DBMS Interview Question Bank

> Use this for active recall. Hide the answer, speak for 30–90 seconds, then verify. Questions marked **Trap** deserve extra repetition.

## A. Fundamentals and architecture

### 1. Why use a DBMS instead of files?

Controlled concurrency, transactions/recovery, constraints, flexible querying, centralized security, reduced duplication, and data independence. Files can still be right for simple immutable/sequential data.

### 2. DBMS vs RDBMS?

DBMS is the broad category; an RDBMS models data as relations and uses relational constraints/operators. Every RDBMS is a DBMS, not every DBMS is relational.

### 3. Schema vs instance?

Schema is the database definition/structure; instance is its data at a particular time.

### 4. Logical vs physical data independence?

Logical: change conceptual schema with minimal external-view/application changes. Physical: change storage/indexing without changing the logical schema. Physical is generally easier to achieve.

### 5. What is a catalog?

System metadata describing tables, columns, constraints, indexes, statistics, users, and privileges; the optimizer and administrative tools consult it.

### 6. OLTP vs OLAP?

OLTP has frequent small concurrent transactions and operational correctness; OLAP has large scans/aggregations for analysis. Their storage, indexing, and schema preferences often differ.

### 7. Row store vs column store?

Row storage keeps a record together, favoring point access and updates. Column storage keeps values of one column together, favoring compression and analytical scans over selected columns.

### 8. Two-tier vs three-tier architecture?

Two-tier clients access the DB directly. Three-tier adds an application/service layer for business logic, validation, pooling, security, and API boundaries.

## B. ER design, keys, and constraints

### 9. Entity, entity set, and relationship?

An entity is one distinguishable object; an entity set is similar entities; a relationship associates entities.

### 10. Strong vs weak entity?

A strong entity has its own identifying key. A weak entity depends on an owner’s key plus a partial key and normally has total participation in its identifying relationship.

### 11. Cardinality vs participation?

Cardinality gives the maximum relationship count (1:1, 1:N, M:N); participation gives the minimum requirement (optional/partial versus mandatory/total).

### 12. Map a many-to-many relationship to tables.

Create a junction table with foreign keys to both entities; commonly use their pair as a composite key and store relationship attributes there.

### 13. Superkey vs candidate key?

A superkey uniquely identifies rows but may contain redundant attributes. A candidate key is a minimal superkey.

### 14. Primary vs alternate key?

One candidate key is selected as primary; the remaining candidate keys are alternate keys. Both are logically unique.

### 15. Natural vs surrogate key?

Natural keys carry domain meaning; surrogate keys are artificial identifiers. Surrogates provide stable compact references but do not remove the need for unique constraints on real business identifiers.

### 16. **Trap:** Does a foreign key have to reference a primary key?

It must reference a candidate/unique key allowed by that DBMS, not necessarily the designated primary key.

### 17. **Trap:** Is a foreign-key column automatically indexed?

Not universally. Many workloads need an explicit index on the referencing columns for joins, deletes/updates of the parent, and lock efficiency.

### 18. `PRIMARY KEY` vs `UNIQUE`?

Both enforce uniqueness, but a primary key is the chosen row identity and is non-null; a table has one primary-key constraint but can have multiple unique constraints. `NULL` behavior for unique constraints is DB-specific.

### 19. Entity integrity vs referential integrity?

Entity integrity requires a valid unique primary key; referential integrity requires each non-null foreign key to match a referenced key, subject to the configured action.

### 20. When should deletion cascade?

When the child is truly owned and has no meaning without the parent. Avoid it across loosely coupled or valuable records where deletion should be explicit/audited.

## C. Relational theory and normalization

### 21. What is relational algebra?

A procedural formal query language whose operators consume relations and return relations: selection, projection, product, join, union, difference, rename, etc.

### 22. Selection vs projection?

Selection filters rows; projection chooses columns. Pure relational algebra projection removes duplicates because relations are sets.

### 23. What is division used for?

“For all” queries, such as students who completed every required course.

### 24. Relational algebra vs relational calculus?

Algebra specifies an operator procedure; calculus declaratively specifies properties of desired tuples/domains. Safe forms are equivalent in expressive power for relational queries.

### 25. What is a functional dependency `X → Y`?

Any two valid tuples agreeing on `X` must agree on `Y`; it is a semantic constraint, not merely a pattern in current sample data.

### 26. What is attribute closure used for?

Compute all attributes implied by an attribute set; use it to test superkeys/candidate keys and FD implication.

### 27. Partial vs transitive dependency?

Partial: a non-prime attribute depends on a proper subset of a composite candidate key. Transitive: a key determines a non-key determinant that determines another non-prime attribute.

### 28. 1NF, 2NF, 3NF in one line each.

1NF: atomic relation-valued domains/no repeating groups. 2NF: 1NF plus no partial dependency of non-prime attributes on a candidate key. 3NF: for every `X → A`, `X` is a superkey or `A` is prime.

### 29. 3NF vs BCNF?

BCNF requires every nontrivial FD determinant to be a superkey; 3NF also permits a non-superkey determinant when the dependent attribute is prime. BCNF is stronger.

### 30. Lossless vs dependency-preserving decomposition?

Lossless means joining decomposed relations recreates exactly the original relation. Dependency preservation means original constraints can be enforced locally without joining decomposed tables.

### 31. Does BCNF always preserve dependencies?

No. A BCNF decomposition can be lossless yet lose direct dependency preservation; 3NF synthesis is often chosen when preserving all FDs matters.

### 32. What anomalies does normalization reduce?

Insertion, update, and deletion anomalies caused by storing multiple independent facts in one row structure.

### 33. When would you denormalize?

After measuring a read bottleneck where duplication/precomputation materially helps and the team can define ownership, update logic, consistency expectations, rebuilds, and monitoring.

### 34. **Trap:** Is storing an address in one text field a 1NF violation?

Not automatically. Atomicity depends on the intended domain and queries; if components must be independently constrained/searched, separate attributes may be better.

## D. SQL

### 35. Logical SQL query order?

`FROM/JOIN → WHERE → GROUP BY → aggregate → HAVING → window functions → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET` (exact formal details vary, but this mental order explains most behavior).

### 36. `WHERE` vs `HAVING`?

`WHERE` filters rows before grouping; `HAVING` filters groups after aggregation. Prefer `WHERE` for pre-aggregate predicates.

### 37. `COUNT(*)` vs `COUNT(column)`?

`COUNT(*)` counts rows; `COUNT(column)` counts non-null values of that expression.

### 38. Why is `NULL = NULL` not true?

`NULL` represents unknown/missing under SQL three-valued logic; equality is unknown. Use `IS NULL`, or a DB-supported null-safe comparison.

### 39. **Trap:** Why can `NOT IN` return no rows unexpectedly?

If its subquery/list contains `NULL`, comparisons can become unknown. `NOT EXISTS` with a correct correlation is often safer.

### 40. `ON` vs `WHERE` with a left join?

`ON` controls which right rows match while preserving unmatched left rows. A `WHERE` predicate on right-side columns can reject the resulting null-extended rows and effectively turn it into an inner join.

### 41. `UNION` vs `UNION ALL`?

`UNION` removes duplicates; `UNION ALL` preserves them and generally avoids duplicate-elimination work.

### 42. `EXISTS` vs `IN`?

Both can express membership and optimizers may transform them. `EXISTS` naturally expresses existence/correlation and avoids `NOT IN`’s null trap; measure actual plans.

### 43. Correlated subquery?

A subquery referencing the outer row. Conceptually evaluated per outer row, though the optimizer may decorrelate it into a join/semi-join.

### 44. View vs materialized view?

A normal view stores a query definition; a materialized view stores results and needs refresh/maintenance. The latter trades freshness/write work/storage for read speed.

### 45. Stored procedure vs function?

Both encapsulate database-side logic, but invocation, transaction control, return behavior, and side-effect rules vary by DBMS. Answer in the named engine’s terms.

### 46. Trigger risks?

Hidden side effects, recursion, ordering ambiguity, write amplification, difficult testing/debugging, and cross-row concurrency issues. Use when centralized DB enforcement outweighs those costs.

### 47. Window function vs `GROUP BY`?

`GROUP BY` collapses rows into groups. A window function computes across related rows while retaining each input row.

### 48. `ROW_NUMBER`, `RANK`, and `DENSE_RANK`?

`ROW_NUMBER` always increments uniquely; `RANK` ties share rank and leave gaps; `DENSE_RANK` ties share rank without gaps. Add deterministic tie-breakers when one row is required.

### 49. Why can the default window frame surprise you?

With `ORDER BY`, defaults may include peer rows and produce a running/peer-aware result rather than the full partition. Specify `ROWS BETWEEN ...` explicitly when correctness depends on it.

### 50. CTE advantages and caveat?

Improves decomposition/readability and enables recursion. It is not universally materialized or an optimization barrier; engine/version behavior matters.

### 51. Keyset vs offset pagination?

Offset is simple but large offsets scan/discard work and concurrent changes cause duplicates/skips. Keyset uses a stable ordered cursor and scales better, but random page jumps are harder.

### 52. Delete duplicate rows but keep the newest—approach?

Rank per business key using `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY created_at DESC, id DESC)`, inspect, then delete rows with rank > 1 inside a safe transaction. Add a unique constraint to prevent recurrence.

## E. Storage, indexes, and query plans

### 53. Why do DBMSs organize storage into pages?

Pages are the core transfer/cache unit, amortize I/O, and let the buffer manager track fixed-size chunks efficiently.

### 54. What is a slotted page?

A page with header and slot directory pointing to variable-length records; records can move within the page without changing the stable slot identifier.

### 55. Heap file vs sorted file?

Heap gives fast insertion and full scans but weak unindexed search. Sorted files help ranges/order but make maintaining order expensive.

### 56. What does the buffer pool do?

Caches database pages, pins in-use pages, tracks dirty state, chooses victims, and coordinates flushing/recovery constraints.

### 57. Dirty page?

A cached page modified since its durable data-file version. It must eventually be flushed, subject to WAL ordering.

### 58. Why a B+ tree instead of a binary search tree?

High fan-out makes the tree shallow in page accesses; page-sized nodes and linked leaves support efficient point and range scans.

### 59. B+ tree vs hash index?

B+ tree supports equality, ranges, and ordered traversal. Hash supports equality well but not natural ordering/range queries.

### 60. Clustered vs non-clustered index?

A clustered organization determines/corresponds to row storage order, aiding ranges; non-clustered indexes are separate structures with row locators. Exact terminology/leaf contents vary by engine.

### 61. What is a covering index?

An index containing all columns needed by a particular query, potentially avoiding base-table fetches. Coverage is query-specific and increases index size/write cost.

### 62. Explain leftmost prefix for `(a,b,c)`.

The entries are primarily ordered by `a`, then `b`, then `c`; predicates starting with leading columns usually support a contiguous seek. `b` alone normally cannot use the ordering for a simple seek.

### 63. Equality before range—why?

Equality on leading columns narrows to a contiguous section; after the first range, later keys often cannot further narrow that section, though they can filter or cover.

### 64. Why might an index not be used?

Too many rows match, table is tiny, statistics are wrong, predicate is non-sargable, composite order mismatches, implicit casts occur, or random row fetches exceed scan cost.

### 65. Is a low-cardinality index useless?

Not always. It depends on distribution, selected fraction, composite/partial design, coverage, and engine features—not the distinct count alone.

### 66. What is sargability?

Whether a predicate can become an efficient index search condition. Avoid unnecessary functions/casts on indexed columns and leading wildcards; rewrite to ranges or use a suitable expression index.

### 67. Sequential scan vs index scan?

Sequential scan reads pages in order and is efficient for broad access; an index scan narrows selective access but may add tree traversal and random table fetches.

### 68. Pick a join algorithm.

Index nested loop for small/selective outer plus indexed inner; hash for large equality joins; sort-merge when inputs are/will be ordered or merge semantics fit. Validate size, memory, and plan estimates.

### 69. What causes hash/sort spill?

The operator exceeds allocated memory and uses temporary storage, adding I/O. Causes include insufficient memory, excessive rows, wide rows, or bad cardinality estimates.

### 70. What is cardinality estimation?

Predicting rows emitted by plan operators using statistics and assumptions. Errors cascade into wrong join orders, access paths, and memory grants.

### 71. How do you read an execution plan?

Compare estimated and actual rows, inspect scans/predicates and join inputs, multiply per-loop work, detect spills, examine buffers/I/O, and locate the earliest major divergence or row explosion.

### 72. Why can the same SQL become slow only for some parameters?

Skew and parameter-sensitive plan reuse: a plan suitable for a rare value can be poor for a common one, or vice versa.

## F. Transactions and concurrency

### 73. Define ACID precisely.

Atomicity: all or none. Consistency: valid-to-valid when rules and transaction are correct. Isolation: concurrent outcomes follow the selected guarantee. Durability: acknowledged commits survive covered failures.

### 74. Atomicity vs idempotency?

Atomicity prevents partial commit; idempotency makes replay have the intended effect of one execution. Robust APIs often need both.

### 75. Why keep transactions short?

Long transactions hold locks/versions, increase contention and deadlocks, delay cleanup, consume resources, and complicate recovery/replication.

### 76. What if the network fails during commit?

Outcome may be unknown, not necessarily rolled back. Use a stable request/idempotency key and query the recorded result before repeating the business action.

### 77. DB update plus message publication—how?

Transactional outbox: atomically write business data and an outbox event, then asynchronously publish with retry and idempotent handling.

### 78. What makes two operations conflict?

Different transactions, same data item, and at least one write.

### 79. Test conflict serializability.

Build a precedence graph with an edge for each ordered conflict. It is conflict-serializable iff the graph is acyclic; topological orders give equivalent serial orders.

### 80. View vs conflict serializability?

View equivalence preserves reads-from and final writers and is broader due to blind writes. Conflict serializability is easier to test/enforce.

### 81. Recoverable, cascadeless, strict?

Recoverable delays a reader’s commit until its source writer commits. Cascadeless reads only committed values. Strict prevents reading or writing an item written by an unfinished transaction. `Strict ⇒ cascadeless ⇒ recoverable`.

### 82. Lost update vs write skew?

Lost update overwrites another write to the same logical value. Write skew updates different items after reading a shared invariant, so the combined state becomes invalid.

### 83. Non-repeatable read vs phantom?

Non-repeatable read changes an already observed row; phantom changes the set satisfying a predicate.

### 84. What is 2PL?

A growing phase acquires locks and a shrinking phase releases them. It gives conflict serializability; strict 2PL holds write locks through commit/abort for strictness.

### 85. Why can 2PL deadlock?

Transactions can acquire locks in different orders and form a circular wait even though the resulting completed histories would be controlled.

### 86. Wait-die vs wound-wait?

Wait-die: older requester waits for younger holder; younger requester aborts against older. Wound-wait: older requester aborts younger holder; younger requester waits for older.

### 87. OCC vs locking?

OCC works privately then validates, favoring low contention; locking blocks/prevents conflicting access earlier, favoring workloads where wasted retries would be costly.

### 88. What is MVCC?

Multiple row versions plus snapshot visibility let reads commonly avoid blocking writes. Writers still conflict, versions need cleanup, and guarantees depend on isolation level.

### 89. Snapshot isolation vs serializable?

Snapshot isolation offers a consistent snapshot and typically prevents same-item concurrent writes, but can permit write skew. Serializable additionally prevents dangerous dependency cycles, sometimes by aborting.

### 90. Why can an MVCC read-only transaction hurt performance?

A long snapshot can keep old versions visible, delaying cleanup and causing storage, cache, undo/table bloat, or replication pressure.

### 91. When use `SELECT ... FOR UPDATE`?

When a decision depends on rows that must not change before a following write. Prefer atomic conditional SQL when possible and remember row locks may not protect an entire predicate.

### 92. Should a deadlock retry rerun only the last statement?

Usually no. The transaction was aborted and its prior reads/decisions are invalid; start a fresh transaction and rerun the whole unit with bounded retry.

## G. Recovery

### 93. What is WAL?

Log records describing a change must become durable before the changed data page; the required commit log must be durable before acknowledging commit.

### 94. Steal/no-steal and force/no-force?

Steal permits flushing uncommitted changes, requiring undo. No-force permits committed dirty pages to remain unwritten, requiring redo. Many systems use steal + no-force.

### 95. ARIES phases?

Analysis reconstructs state, redo repeats necessary history, and undo rolls back loser transactions.

### 96. Why redo a loser before undoing it?

ARIES first reconstructs the exact logged crash-time state (“repeat history”), then performs a uniform logged undo; this supports idempotent restart even across another crash.

### 97. What is a CLR?

A compensation log record records an undo action. It can be redone after another crash and is not undone again, preventing repeated rollback work.

### 98. What does a checkpoint guarantee?

It records recovery state and bounds log scanning; a fuzzy checkpoint does not imply all dirty pages are flushed or old logs are immediately disposable.

### 99. Group commit?

Several commits share a durable log flush, amortizing latency and increasing throughput while preserving the required durability boundary.

### 100. Why is replication not backup?

Replication can copy deletes/corruption immediately and may lack historical retention. Backups plus archived logs enable restoration to earlier states; replication primarily improves availability/read scale.

## H. Project/resume cross-questions

### 101. Why MongoDB rather than PostgreSQL for a project?

Answer from access patterns and data shape: document aggregation, schema evolution, embedded ownership, write/read scale, and team constraints. Also state trade-offs: joins, relational constraints, transaction patterns, analytics, and operational complexity. “MongoDB is faster” is not a valid general answer.

### 102. MongoDB embedding vs referencing?

Embed when data is owned, bounded, and commonly read/updated together. Reference when it grows independently, is shared, needs separate access, or embedding would cause duplication/unbounded documents.

### 103. What is a MongoDB discriminator?

In ODMs such as Mongoose, discriminators store related document subtypes in one collection with a type key and subtype schemas. Discuss validation convenience versus heterogeneous documents and index/query complexity.

### 104. How would you make a Redis-based rate limiter correct?

Use an atomic server-side operation/script for check-and-increment with expiry, choose algorithm (fixed/sliding/token bucket), define failure behavior, key scope, clock/cluster issues, and distinguish approximate from strict limits.

### 105. How do Redis idempotency keys work?

Atomically reserve a request key with TTL, store processing/result state, and return the prior result for retries. Define collision scope, expiry, crash recovery, payload mismatch handling, and whether Redis persistence is adequate for the business risk.

### 106. Why Cassandra for a workload?

Use when query-driven denormalized tables, high write throughput, horizontal scale, and multi-node availability fit. State costs: limited ad-hoc joins/transactions, partition-key design, eventual/tunable consistency, hot partitions, tombstones, and operational burden.

### 107. Design a jobs table for a worker queue.

Include stable ID, status, payload/reference, priority, available time, attempt count, lease owner/expiry, deduplication key, timestamps, and error metadata. Use an atomic claim pattern, short transactions, indexes matching “next eligible job,” lease recovery, and idempotent processing.

### 108. Prevent two workers from processing the same job.

Atomically claim via conditional update or locking query (engine permitting `SKIP LOCKED`), assign a lease, commit quickly, and make the handler idempotent because leases/timeouts can still cause redelivery.

### 109. How would you prove a database optimization on your resume?

Give baseline workload and p50/p95/p99 latency, dataset size, query/plan evidence, exact schema/index/query change, write/storage trade-off, controlled measurement, and post-change result—not an unexplained percentage.

### 110. “Designed a scalable database”—what follow-ups should you expect?

Schema and invariants, access patterns, index rationale, transaction boundaries, hot keys/partitions, consistency choice, failure/retry behavior, migration strategy, observability, load-test method, backup/restore, and the next bottleneck.

## Final rapid-fire traps

1. Candidate key is minimal; primary key is selected.
2. Foreign key indexing is not universal.
3. SQL uses bags and three-valued logic; relational algebra traditionally uses sets.
4. `NOT IN` plus `NULL` is dangerous.
5. Right-side `WHERE` filters can collapse a left join.
6. BCNF can lose dependency preservation.
7. Composite-index order matters.
8. Index existence does not guarantee index use.
9. Serializable can mean abort-and-retry, not only waiting.
10. MVCC does not mean lock-free.
11. Snapshot isolation can allow write skew.
12. Steal needs undo; no-force needs redo.
13. Checkpoint is not backup.
14. Replication is not backup.

