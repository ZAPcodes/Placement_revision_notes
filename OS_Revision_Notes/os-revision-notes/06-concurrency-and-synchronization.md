# 06. Concurrency and Synchronization

> Concurrency bugs occur because correct-looking operations can interleave in unexpected ways.

## Core vocabulary

- **Race condition:** result depends on execution timing/interleaving.
- **Data race:** unsynchronized conflicting memory accesses, at least one a write; language definitions matter.
- **Critical section:** code accessing shared state that must obey an invariant.
- **Mutual exclusion:** at most one participant in a protected region.
- **Atomicity:** an operation appears indivisible relative to observers.
- **Visibility:** when another processor/thread can observe a write.
- **Ordering:** constraints on the observed order of memory operations.

Mutual exclusion alone is not the entire correctness story. A solution also needs meaningful progress and should avoid indefinite postponement.

## Why `count++` is not atomic

Conceptually it performs read, modify and write. Two threads can read the same old value and overwrite one increment. Making the variable `volatile` does not make the compound operation atomic.

## Hardware building blocks

- Atomic test-and-set
- Compare-and-swap (CAS)
- Exchange
- Fetch-and-add
- Memory fences/barriers

These support higher-level locks and lock-free algorithms. An atomic instruction on one variable does not make a multi-step invariant involving other variables atomic.

## Primitive comparison

| Primitive | Ownership | Wait behavior | Good fit |
|---|---|---|---|
| Mutex | Locked/unlocked by owner | Usually sleeps/blocks under contention | General critical sections |
| Spinlock | Ownership-like | Busy-waits | Very short sections where sleeping is impossible/too costly |
| Binary semaphore | Token count 0/1; no strict ownership concept | Blocks | Signalling or mutual exclusion |
| Counting semaphore | Count of available units | Blocks | Resource pools/buffer slots |
| Reader-writer lock | Reader group or one writer | Blocks/spins by implementation | Read-heavy workloads with sufficiently long sections |
| Condition variable | No stored resource count | Atomically releases lock and waits | Waiting for a predicate/state change |
| Barrier | Group rendezvous | Waits for all participants | Phase-based parallel algorithms |

## Mutex vs semaphore

- Mutex expresses ownership: the locker should unlock it.
- Semaphore represents permits/events; one thread may signal a permit another waits for.
- Binary semaphore can enforce exclusion, but its semantics are not identical to a mutex.
- Mutex implementations may support priority inheritance; semaphore behavior varies.

## Spinlock vs mutex

Use a spinlock only when expected wait is shorter than sleep/wakeup overhead and the running owner can make progress. Spinning is harmful on a single available CPU if the owner cannot run, and dangerous across blocking operations.

## Condition variables

Correct conceptual pattern:

```text
lock(m)
while predicate is false:
    wait(cv, m)   // atomically releases m and sleeps; reacquires before return
modify/use protected state
unlock(m)
```

Why `while`, not `if`:

- Spurious wakeups may occur.
- Another thread may consume/change the condition before this thread reacquires the mutex.
- A notification means “state may have changed,” not “your condition is guaranteed true.”

The state/predicate—not the notification—is the source of truth.

## Semaphore patterns

### Bounded buffer

- `empty` counting semaphore tracks free slots.
- `full` tracks occupied slots.
- Mutex protects buffer structure/index updates.
- Producer waits for `empty`, modifies under mutex, then signals `full`.
- Consumer waits for `full`, modifies under mutex, then signals `empty`.

Acquire resources in a consistent order; waiting on a capacity semaphore while holding an unrelated needed lock can deadlock.

## Reader-writer locks

- Multiple readers may enter together; writer requires exclusivity.
- Helpful only when read sections are long/frequent enough to offset added bookkeeping.
- Reader preference can starve writers; writer preference can delay readers.
- Upgrading a read lock to write lock can deadlock if multiple readers attempt it.

## Lock granularity

| Coarse-grained | Fine-grained |
|---|---|
| Simple invariants | More parallelism |
| Fewer lock-order edges | Higher complexity |
| More contention | More overhead and deadlock opportunities |

Start with a simple correct lock boundary. Split only after measuring contention and defining ownership/order clearly.

## Classic problems

### Producer-consumer

Tests mutual exclusion, capacity accounting, lost wakeups and shutdown behavior.

### Readers-writers

Tests concurrency versus starvation policy.

### Dining philosophers

Tests circular wait. Solutions include global lock ordering, limiting simultaneous contenders, or an arbitrator.

## Common bugs

- Check-then-act outside lock
- Lost update
- Lost wakeup
- Wrong lock order
- Sleeping/blocking while holding an important lock
- Returning early without unlocking
- Publishing partially initialized data
- Destroying object while another thread references it
- Incorrect double-checked initialization
- Assuming atomicity implies ordering for all data

## Lock-free terminology

- **Lock-free:** system as a whole makes progress; an individual thread may starve.
- **Wait-free:** every operation completes in bounded steps.
- **Obstruction-free:** progress if a thread eventually runs without interference.

Lock-free does not mean race-free, simple or automatically faster. Memory reclamation and ABA can be difficult.

## Common traps

- Condition variables have no remembered “permit”; semaphores have counts.
- Signal/broadcast should be associated with a state change protected by the same mutex.
- `volatile` is not a general synchronization mechanism.
- A reader-writer lock may underperform a mutex.
- Fine-grained locking can lower performance through overhead/cache traffic.
- Thread safety of individual operations does not make a sequence of them atomic.

## Interview checks

1. Why is `count++` unsafe across threads?
2. Mutex versus binary semaphore?
3. Why does condition-variable wait release the mutex atomically?
4. Why check the predicate in a loop?
5. Spinlock versus mutex: give safe and unsafe scenarios.
6. Can atomics replace every mutex?
7. How can reader-writer locks cause starvation?
8. How would you make an LRU cache thread-safe?
9. Why is lock-free not necessarily faster?

## Resume connection: concurrent LRU store

Questions to prepare:

- One global lock or separate locks for map and list?
- Can a GET mutate LRU ordering, making it logically a write?
- Can `shared_mutex` help if every GET updates recency?
- What is the lock order between key storage, LRU metadata and TTL index?
- Can the eviction thread remove an entry being read?
- How are object lifetimes protected after releasing a lock?

## 60-second recall

- Race = timing-dependent behavior; data race is a conflicting memory-access category.
- Mutex owns exclusion; semaphore counts permits; CV waits for a predicate.
- CV waits must atomically release/reacquire a mutex and recheck in a loop.
- Spin only for very short waits when the owner can run.
- Correctness first; refine lock granularity after measurement.

## References

- [OSTEP: Locks, condition variables, semaphores and concurrency bugs](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [Linux kernel locking documentation](https://docs.kernel.org/locking/index.html)

