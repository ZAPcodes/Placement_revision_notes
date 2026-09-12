# 08. Database Storage and Buffer Management

> A DBMS moves fixed-size pages between storage and memory, then interprets records inside those pages.

## Storage hierarchy perspective

- CPU cache and RAM are fast/volatile.
- SSD/HDD storage is slower/persistent.
- Random I/O and sequential I/O have different costs.
- DBMS organizes data to reduce expensive transfers and exploit locality.

## Pages, records and files

- **Page/block:** basic unit commonly read from/written to storage.
- **Record/tuple:** logical row representation within pages.
- **File:** collection of pages belonging to a table/index/segment.
- Database page size and OS virtual-memory page size are separate concepts even if values sometimes coincide.

## Slotted-page layout

```mermaid
flowchart LR
    H["Page header"] --> S["Slot directory"]
    S --> F["Free space"]
    F --> R["Variable-length records"]
```

- Slot entry identifies record location/length.
- Records can move during compaction while logical slot ID remains stable.
- Supports variable-length rows and deletion without changing every external reference.

## Record representation

- Fixed/variable-length attributes
- NULL bitmap
- Record header/status
- Inline vs overflow/toast-style large values
- Alignment/padding
- Record identifier often contains page + slot information

Wide/large rows can increase I/O and reduce rows per page.

## File organizations

| Organization | Strength | Weakness |
|---|---|---|
| Heap file | Fast unsorted inserts | Search needs scan/index |
| Sorted file | Efficient ordered/range access | Expensive arbitrary insert/reorganization |
| Hash organization | Equality access | Poor range access and bucket growth issues |
| Log-structured | Sequential writes | Read/compaction amplification |

“Heap file” means unordered database pages, not a program's dynamic-memory heap.

## Row vs column storage

| Row-oriented | Column-oriented |
|---|---|
| Entire row stored together | Values of one column stored together |
| Point lookup/update friendly | Analytical scan/aggregation friendly |
| Reads unused columns in wide scans | Reads selected columns only |
| Mixed values reduce compression | Similar values compress well |

Hybrid and vectorized systems blur the boundary.

## Compression

- Dictionary encoding
- Run-length encoding
- Delta encoding
- Bit packing
- Prefix compression

Compression reduces I/O and can improve speed even with CPU decompression. Effectiveness depends on data distribution and access granularity.

## Buffer pool

The buffer pool caches database pages in RAM.

Each frame may track:

- Page identifier
- Pin/reference count
- Dirty state
- Usage/reference information
- Latch/protection state

### Pinning

A pinned page cannot be evicted while an operator actively uses it. Forgetting to unpin leaks buffer capacity.

### Dirty page

Modified in memory but not yet reflected in its persistent home location. Dirty does not mean corrupt.

### Replacement

DBMS often uses Clock/LRU approximations and workload-aware policies. Sequential scans can pollute a naive cache by evicting hot pages.

## Database buffer pool vs OS page cache

DBMS may prefer explicit buffering because it knows:

- Transaction/recovery state
- Pinning and query access patterns
- Which pages are dirty or temporary
- Prefetch and eviction importance

Using both DBMS and OS caches can create double caching. Other systems intentionally rely more on memory mapping/OS cache. There is no universal design.

## Force/no-force and steal/no-steal

| Policy | Meaning | Recovery implication |
|---|---|---|
| Force | Flush transaction's pages at commit | Reduces redo need, expensive commit |
| No-force | Commit without all data pages flushed | Requires redo capability |
| Steal | Evict dirty uncommitted page | Requires undo capability |
| No-steal | Keep uncommitted dirty pages in memory | Avoids undo for disk pages; memory pressure |

WAL enables practical **steal + no-force** designs by logging enough information before page writes.

## Common traps

- Buffer pool is not the same as query-result cache.
- Dirty page can contain committed and uncommitted changes depending on architecture.
- A table scan can be faster than many random index lookups.
- Page pin and transaction lock solve different problems.
- Row store vs column store concerns physical layout, not SQL vs NoSQL.
- Memory mapping is not automatically faster than explicit buffering.
- No-force does not mean no durability; the log can make committed work recoverable.

## Interview checks

1. Why does a DBMS use pages?
2. Explain a slotted page.
3. Heap file vs application heap?
4. Row store vs column store?
5. Buffer pool vs OS page cache?
6. What does pinning mean?
7. Dirty page vs dirty read?
8. Force/no-force and steal/no-steal?
9. Why can a sequential scan beat an index scan?
10. How can a large scan damage a naive buffer cache?

## 60-second recall

- Storage moves pages; pages contain records/slots.
- Heap files favor insert; sorted/hash/log layouts favor different access patterns.
- Buffer pool tracks residency, pins and dirty pages.
- Row storage favors OLTP; column storage favors analytical scans/compression.
- No-force needs redo; steal needs undo; WAL supports both safely.

## References

- [CMU 15-445: Storage and Buffer Management](https://15445.courses.cs.cmu.edu/fall2025/schedule.html)
- [Berkeley CS 186: Disks, Files and Buffers](https://cs186berkeley.net/sp26/)

