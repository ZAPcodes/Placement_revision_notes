# 09 — Indexing and Hashing

> Goal: choose and explain an index, predict when it helps, and recognize when it will not be used.

## 1. What an index is

An index is an auxiliary data structure that maps search keys to rows or data pages. It improves selected reads by spending extra storage and adding work to `INSERT`, `UPDATE`, and `DELETE`.

- **Search key:** indexed column(s); it need not be a candidate key.
- **Dense index:** one entry per search-key value or record.
- **Sparse index:** entries for only some values/pages; requires ordered data.
- **Primary index:** built on the ordering key of the file (textbooks sometimes use this differently from a primary-key index).
- **Secondary index:** does not define the physical order of rows.

**Core trade-off:** an index is not “free speed.” Optimize the workload, not a single query.

## 2. Clustered vs non-clustered

| Property | Clustered | Non-clustered |
|---|---|---|
| Leaf order | Corresponds to row/data-page order | Separate from row order |
| Range scan | Usually excellent | May require many scattered row fetches |
| Count per table | Usually one physical ordering | Usually many possible |
| Leaf payload | Rows or references to data pages, engine-dependent | Row locator plus indexed data |

Terminology is engine-specific. In PostgreSQL, `CLUSTER` physically rewrites a table once; the order is not maintained automatically. In InnoDB, the primary key is the clustered index.

## 3. B-tree and B+ tree

DBMS indexes commonly use a high-fan-out balanced tree, typically described as a **B+ tree**.

- Internal nodes store separator keys and child pointers.
- Leaf nodes contain search keys and row/page references (or rows, depending on the engine).
- Leaves are linked, making ordered and range scans efficient.
- All root-to-leaf paths have the same length.
- Insert/delete may cause split, merge, or redistribution.

Because each node is sized around a page and fan-out is high, the tree stays shallow. Search, insert, and delete are logarithmic in the number of entries, but actual performance is dominated by page access and caching.

### B-tree vs B+ tree interview distinction

In the standard textbook distinction, B-tree records may appear in internal and leaf nodes; B+ tree records/references appear at leaves, while internal nodes only guide search. B+ trees therefore have greater fan-out and better sequential leaf traversal.

## 4. Hash indexes

A hash index applies a hash function to find a bucket.

- Strong for equality lookup: `WHERE user_id = ?`.
- Poor for range/order/prefix operations: `<`, `BETWEEN`, `ORDER BY`, `LIKE 'ab%'`.
- Collisions require overflow handling.
- Static hashing struggles as the file grows; extendible or linear hashing grows dynamically.

Do not claim that hashing is always O(1): collisions, overflow pages, resizing, and I/O matter.

## 5. Composite indexes and the leftmost-prefix idea

For an index on `(a, b, c)`, the physical ordering is first by `a`, then `b`, then `c`.

Commonly useful predicates include:

```sql
WHERE a = ?
WHERE a = ? AND b = ?
WHERE a = ? AND b = ? AND c > ?
```

`WHERE b = ?` alone normally cannot perform a simple ordered seek using this index. Optimizers and engines can have exceptions, so call this a strong design rule—not a universal law.

### Equality before range

For `WHERE tenant_id = ? AND created_at >= ?`, `(tenant_id, created_at)` usually supports the equality lookup followed by a narrow range scan. Columns used only after the first range condition often cannot further narrow the contiguous scan, though they may still filter or cover the query.

## 6. Covering, included, partial, and expression indexes

- **Covering index:** contains everything a query needs, potentially avoiding a base-table lookup. “Covering” is query-relative.
- **Included columns:** stored at leaves for coverage but not part of key ordering, where supported.
- **Partial/filtered index:** indexes only rows satisfying a predicate, e.g. active jobs. Smaller and cheaper, but only helps compatible queries.
- **Expression/function index:** indexes a computed expression such as `LOWER(email)`; the query must match a usable expression.
- **Unique index:** enforces uniqueness while enabling lookup; exact `NULL` semantics are DB-specific.

## 7. Selectivity and cardinality

- **Cardinality:** number of distinct values.
- **Selectivity:** fraction of rows selected by a predicate; smaller fractions are more selective.

An index on a low-cardinality column such as `is_active` may be unhelpful if most rows match. A partial index, composite index, or bitmap-style access in analytical engines may still help. The optimizer compares estimated index access plus row fetches against a sequential scan.

## 8. Why an index may not be used

- The query returns a large part of the table.
- Statistics are stale or estimates are wrong.
- A function/cast on the indexed column makes the predicate non-sargable.
- A leading wildcard is used: `LIKE '%phone%'`.
- Composite-index order does not match the predicate/order.
- Implicit type conversion prevents a useful seek.
- The table is tiny; a scan is cheaper.
- The index does not cover the query and random row lookups cost too much.

```sql
-- Often non-sargable
WHERE DATE(created_at) = DATE '2026-09-11'

-- Range form is usually index-friendly
WHERE created_at >= TIMESTAMP '2026-09-11 00:00:00'
  AND created_at <  TIMESTAMP '2026-09-12 00:00:00'
```

## 9. Index design checklist

1. Start with real frequent/expensive query patterns.
2. Identify equality, range, join, ordering, and grouping columns.
3. Choose composite order deliberately; consider equality before range.
4. Add coverage only when the read benefit justifies size/write cost.
5. Check data distribution, not just distinct count.
6. Inspect the actual execution plan and runtime statistics.
7. Remove redundant or unused indexes carefully.

## 10. Common traps

- A primary key is a constraint; its index implementation is DB-specific.
- A foreign key is **not automatically indexed in every DBMS**.
- More indexes can slow writes and increase locking, WAL, cache pressure, and storage.
- `COUNT(*)` does not necessarily scan every full row; plans vary.
- Indexing every column separately does not replace a suitable composite index.
- An index can accelerate `ORDER BY` only when its ordering is compatible with the query.
- Index fragmentation is not the first explanation for every slow query; validate with plans and measurements.

## Interview checks

1. Why are B+ trees preferred over binary search trees for disk-based indexes?
2. Design an index for “latest 20 successful payments for one user.”
3. Why might the optimizer choose a sequential scan despite a matching index?
4. Can `(a, b)` and `(b, a)` serve the same workload? Explain.
5. When is a covering index harmful?

## 60-second recall

- B+ tree: equality + range + ordered traversal.
- Hash: equality; no natural range order.
- Clustered: row order; usually only one.
- Composite order matters; leftmost prefix is the default mental model.
- Coverage reduces row lookups; selectivity determines usefulness.
- Every index taxes writes, storage, WAL, and cache.

## Sources for deeper revision

- [PostgreSQL: Index Types](https://www.postgresql.org/docs/current/indexes-types.html)
- [PostgreSQL: Multicolumn Indexes](https://www.postgresql.org/docs/current/indexes-multicolumn.html)
- [CMU 15-445 course schedule](https://15445.courses.cs.cmu.edu/fall2025/schedule.html)

