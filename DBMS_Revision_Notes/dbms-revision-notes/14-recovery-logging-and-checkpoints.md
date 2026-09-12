# 14 — Recovery, Logging, and Checkpoints

> Goal: explain how a DB recovers committed work and removes incomplete work after failures.

## 1. Failure types

- **Transaction failure:** constraint violation, deadlock victim, explicit abort, logic error.
- **System crash:** process/OS/power failure loses volatile memory but durable storage remains.
- **Media failure:** disk/page/storage loss or corruption.
- **Site/disaster failure:** loss of a larger failure domain.

Crash recovery is not enough for media or site failure; backups, archived logs, replication, and tested restore procedures address different risks.

## 2. Stable storage is an abstraction

Recovery theory assumes some log information can survive a crash. Real durability depends on the complete stack: DB settings, filesystem, storage controller, device cache, cloud volume, and replication policy.

## 3. Log records and LSNs

A log record may contain:

- transaction ID;
- data item/page identifier;
- before-image or logical undo information;
- after-image or logical redo information;
- a monotonically ordered **log sequence number (LSN)**;
- links to a transaction’s previous log record.

Exact contents vary. Physical logging records byte/page changes; logical logging records higher-level operations; physiological logging mixes page identity with logical-within-page change.

## 4. Write-ahead logging (WAL)

Two central rules:

1. The log record describing a change reaches durable storage before the changed data page is written.
2. The commit record (and required preceding log) reaches the durability boundary before commit success is acknowledged.

Therefore dirty data pages can be flushed later. On restart, the log supplies information needed to redo committed/effective work and undo incomplete work.

## 5. Buffer policies and recovery needs

| Policy | Meaning | Consequence |
|---|---|---|
| Steal | A dirty page from an uncommitted transaction may be evicted | Need UNDO |
| No-steal | Such a page cannot be written before commit | Avoids that UNDO need; harder buffer management |
| Force | All transaction pages are forced at commit | Reduces REDO need; slow commits |
| No-force | Pages may remain dirty after commit | Need REDO; enables fast commits |

Common high-performance systems use **steal + no-force**, so they need both undo and redo support.

## 6. Undo and redo

- **UNDO:** remove effects of transactions that did not commit.
- **REDO:** reapply logged effects that should be present but may not have reached data pages.

Operations should be designed so repeating recovery is safe. Page LSNs and log metadata help determine whether a change is already reflected.

## 7. Checkpoints

A checkpoint limits how far recovery must inspect. A naive checkpoint might pause work and flush everything; production systems generally use **fuzzy checkpoints**, allowing transactions and writes to continue while recording enough state to start recovery efficiently.

A checkpoint does not necessarily mean:

- every dirty page is on disk;
- the log before it can immediately be deleted;
- recovery starts with an empty transaction table.

Log retention must also consider backups, replication, and point-in-time recovery.

## 8. ARIES overview

ARIES is a canonical WAL recovery design summarized as:

1. **Analysis:** reconstruct active transactions and dirty-page information from the latest checkpoint.
2. **Redo:** “repeat history,” reapplying necessary actions, including those of transactions that will later be undone.
3. **Undo:** roll back loser transactions in reverse order.

### Compensation log records (CLRs)

When recovery undoes an action, it logs the undo as a CLR. If another crash occurs during recovery, CLRs show what undo work was already performed and guide continued undo. CLRs are redoable and are not undone again.

### Winners and losers

- **Winner:** committed before the crash.
- **Loser:** active/uncommitted at crash and must be undone.

Commit status and durable log records—not whether a data page happened to be flushed—determine the outcome.

## 9. Group commit

Multiple transactions can share one durable log flush. This amortizes storage latency while preserving each transaction’s durability requirement. It explains why batching/concurrency may improve transaction throughput.

## 10. Shadow paging

Shadow paging maintains a stable old page table and writes changed pages elsewhere; commit atomically switches the root pointer. It can avoid conventional undo/redo for basic recovery, but page copying, fragmentation, concurrency, and garbage collection make it less attractive for many general-purpose systems.

## 11. Backup and point-in-time recovery

- **Full backup:** complete database copy at a point/reference state.
- **Incremental/differential backup:** only changes since a defined earlier backup.
- **Log archiving:** retains WAL/log stream.
- **Point-in-time recovery (PITR):** restore a base backup, then replay logs until a target time/LSN.

Two objectives:

- **RPO:** maximum tolerable data loss measured in time.
- **RTO:** maximum tolerable restoration time.

A backup is useful only if restoration has been tested.

## 12. Replication is not backup

Replication improves availability and may reduce data loss from hardware failure, but it can quickly copy accidental deletes, corruption, or malicious changes. Backups provide retained historical recovery points; use both according to requirements.

## 13. Common traps

- WAL does not require every data page to be flushed at commit.
- “Committed” is not determined by whether the table file already contains the new page image.
- No-force implies possible redo; steal implies possible undo.
- Checkpoints shorten recovery but do not remove the need for logs.
- ARIES redo includes history that may later be undone.
- A transaction whose commit record was not durable at crash is not safely assumed committed.
- `fsync`/durability settings traded for speed can change guarantees.
- Read replicas are not inherently backups.
- Crash-consistent snapshot and application-consistent backup are not always the same.

## Interview checks

1. Why does steal require undo and no-force require redo?
2. What exactly must be durable before acknowledging commit under WAL?
3. Why does ARIES redo loser transactions before undoing them?
4. What problem do CLRs solve if recovery crashes?
5. How do checkpoint, backup, replication, and PITR differ?
6. Explain group commit’s benefit and trade-off.

## 60-second recall

- WAL first, data page later.
- Steal ⇒ UNDO; no-force ⇒ REDO.
- Checkpoint bounds recovery work; fuzzy does not flush everything.
- ARIES: analysis → redo history → undo losers.
- CLRs make undo progress recoverable.
- Replication gives availability; backups/PITR give historical recovery.

## Sources for deeper revision

- [PostgreSQL: WAL Introduction](https://www.postgresql.org/docs/current/wal-intro.html)
- [PostgreSQL: WAL Configuration](https://www.postgresql.org/docs/current/wal-configuration.html)
- [PostgreSQL: Continuous Archiving and PITR](https://www.postgresql.org/docs/current/continuous-archiving.html)
- [CMU 15-445 course schedule](https://15445.courses.cs.cmu.edu/fall2025/schedule.html)

