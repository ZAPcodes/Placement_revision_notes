# 03. Threads and Multicore Processing

> A thread is an execution stream. Threads in one process share the address space and most process resources.

## What is shared?

| Shared by threads of one process | Private to each thread |
|---|---|
| Code and global data | Program counter |
| Heap and mapped memory | CPU registers |
| Open-file descriptor table | Stack |
| Process credentials | Scheduling/thread state |
| Signal dispositions | Thread-local storage |

Every thread requires its own stack because each can be in a different call chain with different local variables and return addresses.

## Process vs thread

| Process | Thread |
|---|---|
| Separate virtual address space | Shares process address space |
| Stronger isolation | Easier data sharing, weaker isolation |
| Communication generally uses IPC | Communication can use ordinary shared variables |
| Creation/switch often costlier | Usually cheaper, but still not free |
| One process failure is more contained | A bad thread can corrupt the entire process |

## Why use threads?

- Responsiveness: one request can progress while another blocks.
- Parallelism: threads may run on different cores.
- Resource sharing: no serialization/IPC needed for every shared structure.
- Throughput: overlap CPU work and waiting.
- Natural structure: server connection handling, background maintenance and pipelines.

Costs: races, deadlocks, stack memory, scheduling overhead, cache contention and more difficult testing.

## User-level vs kernel-level threads

| User-level threads | Kernel-level threads |
|---|---|
| Scheduled by a runtime/library | Scheduled by OS kernel |
| Very cheap user-space operations possible | Kernel can schedule each independently |
| Kernel may not see each logical thread | Each thread is visible to kernel |
| A blocking syscall may block the whole process in many-to-one model | One blocked thread need not block others |
| Cannot get true multicore parallelism in many-to-one mapping | Can run simultaneously on multiple cores |

### Mapping models

- **Many-to-one:** many user threads mapped to one kernel thread.
- **One-to-one:** each user thread maps to a kernel thread.
- **Many-to-many:** runtime schedules many user threads over several kernel threads.

Modern runtimes may use fibers/coroutines or language tasks above kernel threads; do not assume a language task always equals an OS thread.

## Thread lifecycle

- Creation allocates thread state and stack.
- Runnable thread competes for CPU.
- Blocking operation moves it off the runnable queue.
- Join waits for completion and usually retrieves status.
- Detached thread releases its resources without a future join.
- Cancellation is dangerous if the thread holds locks or owns partially updated state.

## Thread pools

A fixed/bounded pool separates tasks from worker creation:

- Avoids unbounded thread creation.
- Amortizes setup cost.
- Controls concurrency and memory usage.
- Allows queues, backpressure and rejection policies.

Important design questions:

- Pool size for CPU-bound vs I/O-bound work
- Bounded vs unbounded queue
- Task cancellation and shutdown
- Exception handling
- Long-running tasks starving short tasks
- Per-connection thread vs event-driven design

## Concurrency vs parallelism

- **Concurrency:** multiple tasks make progress over overlapping time.
- **Parallelism:** multiple tasks execute simultaneously.
- A single core can run concurrent threads through interleaving.
- Multiple cores allow parallel execution, but only if the workload and synchronization permit it.

## Multicore concerns

### CPU affinity

Keeping a thread on one CPU can improve cache locality. Strict affinity can also harm load balancing.

### Load balancing

Schedulers move runnable work among CPUs. Migration balances utilization but may lose warm-cache state.

### False sharing

Independent variables placed on the same cache line can cause cores to invalidate each other's cache lines. No logical data race is required for severe performance loss.

### Oversubscription

Too many runnable threads cause context switches and cache disruption. More threads do not imply more throughput.

## Thread safety and re-entrancy

- **Thread-safe:** safe when invoked concurrently, possibly using locks.
- **Re-entrant:** can be interrupted and safely invoked again before a prior call finishes; should not depend on unsafe shared mutable state.
- Re-entrant code is generally thread-safe, but thread-safe code using internal locks/global state need not be re-entrant.

## Common traps

- Threads share heap data but do not share their stacks.
- Shared address space makes communication easy, not automatically safe.
- Same-process thread switches can still enter the kernel and disturb caches.
- User-level thread switching is cheap, but a many-to-one runtime cannot use multiple cores simultaneously.
- Thread pools require bounded-resource thinking; an unbounded task queue only moves the overload problem.
- `volatile` does not create mutual exclusion or general cross-thread ordering.

## Interview checks

1. What resources are shared by two threads?
2. Why is a thread switch usually cheaper than a process switch?
3. Can threads run in parallel on a single-core machine?
4. When would processes be safer than threads?
5. How would you size a thread pool?
6. Why can adding threads reduce throughput?
7. Explain user-level and kernel-level thread trade-offs.
8. What is false sharing, and how can it occur without a race?
9. Thread-safe versus re-entrant?

## Resume connection: C++ key-value store

Be ready to explain:

- Why a fixed thread pool was selected over one thread per client.
- What happens when the task queue fills.
- Which state is shared between worker threads.
- Whether long requests can starve short requests.
- How shutdown handles queued work and active sockets.
- How the background TTL thread coordinates with GET/SET/DEL.

## 60-second recall

- Threads share process resources but own execution state and stacks.
- Concurrency is overlap; parallelism is simultaneous execution.
- Kernel threads enable independent blocking and multicore execution.
- Pools control creation cost and concurrency but require queue/backpressure design.
- Multicore performance is limited by contention, migration and false sharing.

## References

- [OSTEP: Concurrency and Threads](https://pages.cs.wisc.edu/~remzi/OSTEP/)
- [MIT 6.1810: thread switching and multicore scalability](https://pdos.csail.mit.edu/6.S081/2026/schedule.html)

