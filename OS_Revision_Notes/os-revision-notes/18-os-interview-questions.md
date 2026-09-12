# 18. Operating Systems Interview Questions

> Use this as active recall. Cover the answer, speak for 30–60 seconds, then check precision. Follow-ups marked **Trap** target common mistakes.

## A. Fundamentals and kernel boundary

### 1. What are the main responsibilities of an OS?

Provide abstractions, allocate CPU/memory/storage/devices, isolate processes, enforce protection and expose common services through system calls.

### 2. Kernel mode vs user mode?

Kernel mode can execute privileged instructions and access protected hardware/memory. User mode is restricted; applications request privileged work through controlled kernel entry.

### 3. System call vs function call?

A function call normally changes control flow within the same privilege domain. A system call uses a special instruction/trap to enter the kernel, validate arguments and perform privileged work. A library function may or may not invoke a syscall.

### 4. Interrupt vs exception?

An interrupt is normally asynchronous to the current instruction and comes from hardware such as a timer/device. An exception is synchronous with the current instruction, such as a fault or system-call trap.

### 5. Mode switch vs context switch?

A mode switch changes privilege while the same process may continue. A context switch changes the running thread/process. A syscall causes a mode switch but need not schedule another process.

### 6. Why does the OS need a timer?

It ensures the kernel regains control from user programs, supports preemptive scheduling and prevents one task from monopolizing the CPU.

### 7. Monolithic vs microkernel?

Monolithic kernels keep most services in kernel space for efficient direct calls. Microkernels keep a minimal privileged core and move services to isolated user processes, improving modularity/fault isolation but adding IPC overhead.

### 8. Mechanism vs policy?

Mechanism provides an ability; policy chooses how to use it. Timer interrupts enable preemption; the scheduling algorithm chooses the next task.

**Trap:** “Every interrupt causes a context switch.” False. The handler may return to the same thread.

## B. Processes

### 9. Program vs process?

A program is passive executable content. A process is an active instance with an address space, CPU state, kernel metadata and resources.

### 10. What is stored in a PCB?

PID/state, saved registers/program counter, scheduling/accounting data, address-space reference, open resources, credentials and signal information.

### 11. Ready vs blocked?

Ready means able to execute but waiting for CPU. Blocked means unable to execute until an event such as I/O completion or lock availability.

### 12. What happens during a context switch?

Kernel saves the current thread's execution state, selects another runnable thread, changes address-space state if needed, restores registers and resumes it. Cache/TLB disruption can dominate indirect cost.

### 13. Explain `fork()`.

It creates a child with a logically duplicated address space. Parent receives child PID; child receives zero. Physical pages are normally shared copy-on-write initially.

### 14. Explain `exec()`.

It replaces the current process image with a new program. PID remains, successful execution does not return to old code, and descriptors may survive unless close-on-exec.

### 15. Why use `fork()` then `exec()`?

The child can adjust descriptors, environment, credentials or redirections between creation and program replacement. This makes process setup composable.

### 16. What is a zombie?

An exited child whose status has not been collected. It does not execute and retains only limited kernel bookkeeping such as PID/status.

### 17. Zombie vs orphan?

Zombie is dead but unreaped. Orphan is still running after its original parent exits and is re-parented to an appropriate system process/subreaper.

### 18. What is copy-on-write?

Processes temporarily share physical pages read-only. A write fault allocates a private copy for the writer, avoiding eager copying.

### 19. Are files duplicated after `fork()`?

Descriptor table entries are inherited, but they generally reference the same underlying open-file description, so parent and child may share file offset/status state.

**Trap:** “`exec()` creates a child.” False; `fork()` creates, `exec()` replaces.

## C. Threads and scheduling

### 20. What do threads share?

They share code, heap/mappings, globals and process resources such as open descriptors. Each has its own registers, program counter, stack, scheduling state and thread-local storage.

### 21. Why does each thread need its own stack?

Threads execute independent call chains with distinct locals, return addresses and saved registers.

### 22. Concurrency vs parallelism?

Concurrency is overlapping progress; parallelism is simultaneous execution. One CPU can provide concurrency through interleaving but not simultaneous CPU execution.

### 23. User-level vs kernel-level threads?

User threads can switch cheaply under a runtime, but in many-to-one mapping one blocking call blocks all and multicore parallelism is unavailable. Kernel threads are independently schedulable and can run on different cores.

### 24. Why use a thread pool?

It amortizes creation, bounds concurrent resource use and enables queue/backpressure policies. An unbounded queue or pool can still overload the service.

### 25. FCFS weakness?

Convoy effect: short/I/O-bound jobs wait behind a long CPU-bound job, harming response and device/CPU overlap.

### 26. Why is SJF optimal for average waiting time?

With known fixed bursts, placing shorter work first prevents many short jobs from waiting behind a long one. Real systems must predict bursts and address starvation.

### 27. Round Robin quantum trade-off?

Too large approaches FCFS; too small wastes time on context switching. Quantum should exceed switching cost while meeting responsiveness goals.

### 28. Starvation vs priority inversion?

Starvation is indefinite denial while others progress. Priority inversion specifically occurs when high-priority work waits on lower-priority work holding a needed resource.

### 29. What is CPU affinity?

A preference/restriction keeping a task on a CPU to preserve cache locality. Too-strict affinity can reduce load balance.

### 30. Why can more threads reduce performance?

More scheduling, lock contention, cache invalidation, memory/stack use and queueing; the serial fraction also limits scaling.

## D. Synchronization

### 31. Race condition vs data race?

Race condition is broad timing-dependent behavior. Data race is unsynchronized conflicting memory access, at least one write, under a language/hardware memory model.

### 32. Why is `x++` not necessarily atomic?

It is read-modify-write. Two threads may read the same value and overwrite one update.

### 33. Mutex vs semaphore?

Mutex protects ownership of a critical section and should be unlocked by its owner. Semaphore counts permits/events and can be posted by a different thread.

### 34. Binary semaphore vs mutex?

Both can enforce exclusion, but a mutex carries ownership semantics and may provide priority inheritance/debugging. A binary semaphore models a transferable permit/signal.

### 35. Spinlock vs mutex?

Spinlocks busy-wait and suit very short sections when the owner can run and sleeping is inappropriate. Mutexes block/sleep and suit longer waits. Never block or perform slow I/O while holding a spinlock.

### 36. Semaphore vs condition variable?

Semaphore stores a count of permits. Condition variable stores no resource count; it lets threads sleep until shared state may satisfy a predicate, checked under a mutex.

### 37. Why does condition-variable wait release the mutex atomically?

Otherwise a notifier could change state and signal after the waiter checks the predicate but before it sleeps, causing a lost wakeup.

### 38. Why use `while`, not `if`, around a condition-variable wait?

Wakeups may be spurious, and another thread may consume the condition before the waiter reacquires the mutex.

### 39. Can atomics replace locks?

For simple independent state, sometimes. Multi-variable invariants, blocking coordination and complex lifetime changes often need locks or sophisticated lock-free protocols.

### 40. What is lock granularity?

How much state one lock protects. Coarse locks simplify correctness but contend; fine locks increase parallelism but create overhead and lock-order complexity.

### 41. What is false sharing?

Different threads modify independent variables on the same cache line, causing unnecessary coherence invalidations and poor scaling without a logical data race.

### 42. Is `volatile` a thread-safety tool?

Not generally. It does not supply mutual exclusion or the full atomicity/ordering needed for shared-memory synchronization.

### 43. Thread-safe vs re-entrant?

Thread-safe code works under concurrent calls, potentially through locks. Re-entrant code can be interrupted and safely called again before a previous invocation completes; locked/global-state code may be thread-safe but not re-entrant.

### 44. Lock-free vs wait-free?

Lock-free guarantees system-wide progress; some thread may starve. Wait-free guarantees each operation completes within bounded steps.

**Trap:** “A thread-safe map makes `if (!contains) insert` atomic.” False; composition of safe operations is not automatically atomic.

## E. Deadlocks

### 45. Four necessary deadlock conditions?

Mutual exclusion, hold-and-wait, no preemption and circular wait.

### 46. How does lock ordering prevent deadlock?

If every thread acquires locks in one strict global order, a circular dependency cannot form.

### 47. Prevention vs avoidance?

Prevention structurally breaks a necessary condition. Avoidance examines each request and grants it only if the resulting state remains safe.

### 48. Unsafe vs deadlocked state?

Unsafe means no guaranteed safe completion sequence; it can still be running. Deadlocked means participants are already permanently waiting.

### 49. Deadlock vs livelock?

Deadlocked participants are blocked. Livelocked participants actively change/retry but continually interfere and make no useful progress.

### 50. Can timeouts solve deadlock?

They can bound waiting/detect symptoms, but correctness also requires safe cancellation, rollback and consistent resource cleanup.

## F. IPC and signals

### 51. Shared memory vs pipe?

Shared memory is efficient for bulk data but requires explicit synchronization/protocol. Pipe is a kernel-managed byte stream with simpler read/write blocking semantics.

### 52. Why might a pipe read never return EOF?

At least one process still holds an open write endpoint, possibly an accidentally inherited duplicate.

### 53. Pipe vs message queue?

Pipe is generally a byte stream. Message queue preserves discrete message units and may support priorities, at the cost of limits/copying.

### 54. Pipe vs socket?

Pipes are commonly local and simple, especially parent-child. Sockets are bidirectional and can support unrelated local processes or networks.

### 55. Blocking vs synchronous?

Blocking describes whether the calling thread waits. Synchronous describes how completion relates to the call/control flow. They are separate dimensions.

### 56. Why are signal handlers restricted?

They can interrupt code while libraries hold internal state/locks. Calling non-async-signal-safe functions can corrupt state or deadlock.

## G. Memory, paging and virtual memory

### 57. Virtual vs physical address?

Virtual address belongs to a process address space. MMU/page tables translate it to a physical memory location and enforce permissions.

### 58. Internal vs external fragmentation?

Internal is unused space inside an allocated unit. External is free space split into holes between variable allocations.

### 59. Paging vs segmentation?

Paging uses fixed-size virtual pages/physical frames and is largely invisible to program structure. Segmentation uses variable-size logical regions with independent base/limit/protection.

### 60. Page table vs TLB?

Page table is the authoritative memory mapping structure. TLB is a small hardware cache of recent translations.

### 61. TLB miss vs page fault?

TLB miss requires page-table lookup. If mapping is present/permitted, access continues after filling TLB. Page fault traps to kernel because mapping/residency/protection requires handling.

### 62. Why multilevel page tables?

They allocate lower-level metadata only for populated address regions, saving memory for sparse address spaces.

### 63. Page-size trade-offs?

Large pages reduce page-table size and increase TLB reach but increase allocation granularity/internal waste. Small pages offer finer mapping/protection but create more metadata/TLB pressure.

### 64. Page fault vs segmentation fault?

Page fault is the mechanism for exceptional translation/access; many are legal and transparently resolved. Segmentation/access violation is typically delivered when the access is invalid or prohibited.

### 65. Explain demand paging.

Pages are loaded/allocated on first access. It saves startup and RAM but introduces page-fault latency and requires replacement/backing policies.

### 66. Explain COW after `fork()`.

Parent/child share read-only physical pages. First write faults, copies the page and makes the writer's mapping private.

### 67. What is thrashing?

Active working sets exceed available memory, so the system spends most time faulting/swapping rather than executing useful work.

### 68. Why can FIFO worsen with more frames?

FIFO lacks the stack property and can exhibit Belady's anomaly; its eviction order can become less favorable after capacity changes.

### 69. Why is exact LRU expensive?

It requires precise ordering/recency updates on many accesses. Hardware reference bits plus Clock/aging approximate it more cheaply.

### 70. Why may memory usage stay high after `free()`?

The allocator may retain freed blocks/arenas for reuse, fragmentation may prevent returning regions, or resident pages may remain until pressure.

## H. File systems

### 71. File descriptor vs inode?

Descriptor is a per-process integer referring to an open-file entry. Inode stores filesystem metadata and block mapping. The open-file entry holds offset/status flags and references the inode.

### 72. Hard link vs symbolic link?

Hard link is another name for the same file identity/inode. Symbolic link is a separate file containing a path and can dangle/cross filesystems.

### 73. What happens when an open file is deleted?

Its directory name/link is removed, but existing open descriptors can continue accessing it. Storage is reclaimed after links and open references disappear.

### 74. Does `write()` guarantee disk durability?

Usually no. It can return after copying into kernel/page cache. Durability needs appropriate synchronization plus filesystem/device guarantees.

### 75. Why journaling?

Multi-block metadata changes can be interrupted by a crash. A write-ahead journal records a committed operation so recovery can replay or discard it consistently.

### 76. Journaling vs log-structured filesystem?

Journaling adds a recovery log around updates to regular home locations. A log-structured filesystem treats the log as the main storage layout and later cleans segments.

### 77. Why can parent/child share a file offset?

Inherited descriptors after `fork()` reference the same underlying open-file description containing the offset.

### 78. What is VFS?

A common kernel abstraction that routes generic file operations to different concrete filesystem implementations.

## I. I/O, storage and RAID

### 79. Polling vs interrupts?

Polling repeatedly checks and can be efficient for very short/high-rate events but wastes CPU. Interrupts free CPU until notification but incur handler overhead and can storm under load.

### 80. What does DMA do?

A controller transfers bulk data between device and memory after CPU setup, then reports completion. CPU still manages descriptors, errors, buffers and coherence.

### 81. Buffering vs caching vs spooling?

Buffering smooths producer/consumer speed or transfer units. Caching stores reusable copies. Spooling queues whole jobs for a serial shared device.

### 82. Why can SSTF starve?

A distant request may wait indefinitely if nearer requests keep arriving.

### 83. Why do SSDs not need classic seek optimization?

They have no moving head, so physical distance is not the dominant latency. Queueing, internal parallelism, garbage collection and writes still matter.

### 84. Why is RAID not backup?

RAID addresses certain disk failures/availability. Deletion, corruption, software bugs, ransomware and site loss can affect every member simultaneously.

## J. Resume-driven scenario questions

### 85. Can a `shared_mutex` make every GET concurrent in an LRU cache?

Not if GET updates LRU order; that metadata mutation needs exclusive coordination or a redesigned/approximate recency mechanism.

### 86. How can TTL eviction race with GET?

Eviction may locate/erase an entry while GET references it. Protect lookup, lifetime and removal through a consistent lock/reference strategy; define expiration at read time.

### 87. How can an append-only log end with a partial record?

Crash/power loss may interrupt a write. Use framed records with length/checksum/version; replay only complete valid records and truncate/ignore the invalid suffix.

### 88. Does replaying a log always give the latest state?

Only if record order, operation semantics, corruption policy and acknowledged-write durability are defined. Expiration timestamps and non-idempotent operations need care.

### 89. Why a fixed thread pool for a server?

It bounds concurrent stacks/scheduling and amortizes creation. It still needs a bounded queue, overload policy, shutdown behavior and protection from long tasks.

### 90. What should “high throughput” mean on a résumé?

A reproducible benchmark: workload mix, payload size, client concurrency, hardware, duration, throughput, p50/p95/p99 latency, errors and comparison baseline.

## Final trap list

- Mode switch != context switch.
- TLB miss != page fault.
- Page fault != segmentation fault.
- `fork` creates; `exec` replaces.
- Zombie is already terminated.
- Threads share heap, not stack.
- `volatile` != synchronization.
- Binary semaphore != mutex semantics.
- Unsafe state != current deadlock.
- Non-blocking != asynchronous.
- File descriptor != inode.
- `write` completion != durable storage.
- Journaling != backup.
- RAID != backup.
- More threads/frames/locks do not monotonically improve performance.

