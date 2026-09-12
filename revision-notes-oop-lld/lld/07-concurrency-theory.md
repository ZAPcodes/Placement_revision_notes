# 07 — Concurrency Theory ★★

The course teaches concurrency in **Java**, so this file is Java-first, with **C++ equivalents** noted. Core OS concepts (race condition, critical section, semaphores, deadlock conditions) are in the OS notes; this file covers the language-level tools.

## Basics recap
- **Process:** independent program with its own address space. **Thread:** unit of execution inside a process; threads share heap and code, each has its own stack and registers.
- **Concurrency:** multiple tasks making progress in overlapping time (can be on one core). **Parallelism:** tasks literally running at the same instant on multiple cores.
- **Race condition:** result depends on the timing of threads accessing shared mutable data. `count++` is **not atomic** (read → add → write).

## 1. Creating threads
| Way | Notes |
|---|---|
| `extends Thread`, override `run()` | Uses up your single inheritance |
| `implements Runnable`, pass to `new Thread(r)` | Preferred; separates task from thread |
| `implements Callable<V>` | Like Runnable but **returns a value** and can **throw checked exceptions**; used with executors → `Future<V>` |
| Lambda | `new Thread(() -> work()).start();` |

> [!WARNING]
> **Tricky:**
> - `t.run()` just calls the method **on the current thread** — no new thread. `t.start()` creates the thread, which then calls `run()`.
> - Calling `start()` twice on the same thread → `IllegalThreadStateException`.
> - **Daemon threads** (`setDaemon(true)` before start) don't keep the JVM alive; the JVM exits when only daemon threads remain (e.g. GC).

**C++:** `std::thread t(func, args...); t.join();` — a `std::thread` that is still joinable when destroyed calls `std::terminate`. C++20 `std::jthread` joins automatically.

## 2. Thread lifecycle (Java `Thread.State`)

| State | When |
|---|---|
| `NEW` | Created, `start()` not called |
| `RUNNABLE` | Running or ready to run (Java merges both) |
| `BLOCKED` | Waiting to acquire a **monitor lock** (`synchronized`) |
| `WAITING` | Waiting indefinitely: `wait()`, `join()`, `LockSupport.park()` |
| `TIMED_WAITING` | `sleep(ms)`, `wait(ms)`, `join(ms)` |
| `TERMINATED` | `run()` finished or threw |

### Useful methods
- `join()` — wait for another thread to finish.
- `sleep(ms)` — pause; **keeps any locks held**.
- `yield()` — hint to the scheduler to let others run; no guarantee.
- `interrupt()` — sets a flag; blocking methods (`sleep`, `wait`, `join`) throw `InterruptedException`.

> [!WARNING]
> **Tricky — `sleep()` vs `wait()` (asked constantly):**
>
> | | `sleep()` | `wait()` |
> |---|---|---|
> | Class | `Thread` (static) | `Object` |
> | Releases lock? | **No** | **Yes** |
> | Needs `synchronized`? | No | **Yes**, else `IllegalMonitorStateException` |
> | Woken by | Time elapsing / interrupt | `notify()`/`notifyAll()` / timeout / interrupt |

## 3. Thread pools & Executors
Creating a thread per task is expensive (memory for stack, OS scheduling). A **thread pool** reuses a fixed set of worker threads that pull tasks from a queue.

| Factory (`Executors.`) | Behaviour |
|---|---|
| `newFixedThreadPool(n)` | n threads, **unbounded** queue |
| `newCachedThreadPool()` | Creates threads on demand, reuses idle ones (60 s keep-alive), no queue capacity (`SynchronousQueue`) — can explode thread count |
| `newSingleThreadExecutor()` | One thread → tasks run sequentially in order |
| `newScheduledThreadPool(n)` | Delayed / periodic tasks |
| `newVirtualThreadPerTaskExecutor()` (Java 21) | A lightweight virtual thread per task |

- `execute(Runnable)` — fire and forget. `submit(Runnable/Callable)` — returns a `Future`.
- `shutdown()` — stop accepting, finish queued tasks. `shutdownNow()` — interrupt running tasks, return queued ones. `awaitTermination()` — wait.

### `ThreadPoolExecutor` parameters
`corePoolSize`, `maximumPoolSize`, `keepAliveTime`, `workQueue`, `threadFactory`, `RejectedExecutionHandler`.

> [!WARNING]
> **Tricky — order in which the pool grows:**
> 1. Fewer than **core** threads running → create a new thread.
> 2. Else → put the task in the **queue**.
> 3. Queue full → create threads up to **max**.
> 4. Still can't → **reject**.
>
> So with an **unbounded queue**, the pool **never grows beyond core** — `maximumPoolSize` is ignored.
>
> **Rejection policies:** `AbortPolicy` (default, throws `RejectedExecutionException`), `CallerRunsPolicy` (the submitting thread runs it — natural back-pressure), `DiscardPolicy`, `DiscardOldestPolicy`.

## 4. Synchronization
### `synchronized`
- On an **instance method** → locks `this`. On a **static method** → locks the `Class` object. On a **block** → locks the given object.
- Guarantees **mutual exclusion** and **visibility** (changes made inside are visible to the next thread acquiring the same lock).
- Java intrinsic locks are **reentrant**: a thread holding a lock can acquire it again (e.g. a synchronized method calling another synchronized method on the same object).

> [!WARNING]
> **Tricky:** a static synchronized method and an instance synchronized method of the same class **do not block each other** — they use different locks (Class object vs `this`).

### `volatile`
- Guarantees **visibility** (every read sees the latest write, no CPU-cache staleness) and prevents certain **reorderings**.
- Does **not** guarantee **atomicity**: `volatile int count; count++` is still a race.
- Use for flags (`volatile boolean running`) and in double-checked locking.

### Atomic classes
`AtomicInteger`, `AtomicLong`, `AtomicReference`, `LongAdder` — lock-free atomic operations using **CAS (compare-and-swap)**: "set to new value only if it's still the expected old value, else retry".
- **ABA problem:** value changes A → B → A, and CAS wrongly thinks nothing changed. Fix: `AtomicStampedReference` (version stamp).

**C++:** `std::mutex` + `std::lock_guard` / `std::unique_lock` (RAII), `std::atomic<T>`. C++ `volatile` is **not** for threading.

## 5. Thread communication (wait / notify)
- `wait()` releases the lock and suspends; `notify()` wakes **one** waiting thread; `notifyAll()` wakes **all**.
- All three must be called while holding the object's monitor (inside `synchronized` on that object).
- Classic use: producer–consumer on a shared buffer.

> [!WARNING]
> **Tricky:**
> - **Always call `wait()` inside a `while` loop**, not `if`: `while (queue.isEmpty()) wait();` — because of **spurious wakeups** and because another thread may consume the item before you re-acquire the lock.
> - `notify()` can wake the "wrong" kind of waiter (a producer instead of a consumer) → possible stall. `notifyAll()` is safer; `Condition` objects (below) allow separate wait queues.
> - A woken thread doesn't run immediately — it must first **re-acquire the lock**.

**C++:** `std::condition_variable` with `wait(lock, predicate)` — the predicate overload handles spurious wakeups for you.

## 6. Locks (`java.util.concurrent.locks`)

| Lock | Key features |
|---|---|
| `ReentrantLock` | Explicit `lock()`/`unlock()`; `tryLock()` (non-blocking / with timeout — helps avoid deadlock); `lockInterruptibly()`; optional **fairness** (longest-waiting thread gets it); multiple `Condition`s |
| `ReentrantReadWriteLock` | Many readers **or** one writer. Good for read-heavy data (caches, configs) |
| `StampedLock` (Java 8) | Read/write lock + **optimistic reads** (read without locking, then validate the stamp); **not reentrant** |
| `Condition` | `await()` / `signal()` / `signalAll()` — like wait/notify but you can have several per lock (e.g. `notFull`, `notEmpty`) |

> [!WARNING]
> **Tricky:**
> - Always `unlock()` in a **`finally`** block, otherwise an exception leaves the lock held forever.
> - `synchronized` vs `ReentrantLock`: synchronized is simpler and auto-releases; ReentrantLock adds tryLock, timeouts, interruptibility, fairness, multiple conditions.
> - Fair locks reduce starvation but lower throughput.
> - **Optimistic** locking (versions/CAS — assume no conflict, detect it) vs **pessimistic** locking (lock first). Optimistic suits low contention.

**C++:** `std::mutex`, `std::recursive_mutex`, `std::timed_mutex`, `std::shared_mutex` (read-write, C++17) with `std::shared_lock`.

## 7. Semaphore
`Semaphore(permits)` — `acquire()` takes a permit (blocks if none), `release()` returns one.
- **Binary semaphore** (1 permit) ≈ a lock; **counting semaphore** limits concurrent access to N (e.g. max 10 DB connections, parking spots).

> [!WARNING]
> **Tricky:** a semaphore has **no ownership** — any thread can `release()`, even one that never acquired. A mutex/lock must be unlocked by the thread that locked it. That's the core **mutex vs binary semaphore** difference.

Other synchronizers: `CountDownLatch` (wait until N events happen; **one-shot**), `CyclicBarrier` (N threads wait for each other; **reusable**), `Phaser`, `Exchanger`.

## 8. Concurrent collections

| Collection | Notes |
|---|---|
| `ConcurrentHashMap` | Thread-safe map. Java 8+: **CAS** for empty buckets, **synchronized on the bucket's first node** otherwise (older Java used segments). Reads mostly lock-free. **No `null` keys or values.** Atomic ops: `putIfAbsent`, `computeIfAbsent`, `merge` |
| `CopyOnWriteArrayList` | Every write copies the whole array. Great for read-heavy, rarely-modified lists (listener lists). Iterators see a snapshot |
| `BlockingQueue` | `put()` blocks when full, `take()` blocks when empty. Impl: `ArrayBlockingQueue` (bounded, array), `LinkedBlockingQueue` (optionally bounded), `PriorityBlockingQueue`, `DelayQueue`, `SynchronousQueue` (zero capacity; hand-off) |
| `ConcurrentLinkedQueue` | Lock-free non-blocking queue |
| `ConcurrentSkipListMap` | Sorted concurrent map |

> [!WARNING]
> **Tricky:**
> - **`Hashtable` / `Collections.synchronizedMap` vs `ConcurrentHashMap`:** the first two lock the **whole map** for every operation; CHM locks per bucket, so it scales far better.
> - **Why no nulls in CHM?** `get(key) == null` would be ambiguous (absent vs mapped to null), and you can't check-then-act safely in concurrent code.
> - **Fail-fast vs fail-safe iterators:** `ArrayList`/`HashMap` iterators throw `ConcurrentModificationException` on concurrent modification; concurrent collections' iterators are **weakly consistent** (never throw, may or may not reflect recent changes).
> - Compound actions are still unsafe even on thread-safe collections: `if (!map.containsKey(k)) map.put(k, v);` is a race → use `putIfAbsent` / `computeIfAbsent`.

## 9. Future & CompletableFuture
### `Future<V>`
Returned by `executor.submit()`. `get()` **blocks** until the result is ready; `get(timeout)`; `isDone()`; `cancel()`.
Limitations: blocking, can't chain steps, can't combine multiple futures, no callback on completion, awkward exception handling.

### `CompletableFuture<V>` (Java 8)
Non-blocking, composable async pipeline.

| Method | Use |
|---|---|
| `supplyAsync(supplier)` / `runAsync(runnable)` | Start async task (default: `ForkJoinPool.commonPool()`, or pass an executor) |
| `thenApply(fn)` | Transform result (like `map`) |
| `thenCompose(fn)` | Chain another async call returning a CompletableFuture (like `flatMap`) — avoids nested futures |
| `thenAccept(consumer)` / `thenRun` | Consume result / run after, no return |
| `thenCombine(other, fn)` | Combine two independent futures' results |
| `allOf(...)` / `anyOf(...)` | Wait for all / first of many |
| `exceptionally(fn)` / `handle(fn)` | Recover from errors / handle result-or-error |
| `join()` vs `get()` | Both block; `join()` throws unchecked `CompletionException`, `get()` throws checked exceptions |

> [!WARNING]
> **Tricky:** `thenApply` vs `thenCompose` — if your function itself returns a `CompletableFuture`, `thenApply` gives you `CompletableFuture<CompletableFuture<T>>`; `thenCompose` flattens it. The `...Async` variants (`thenApplyAsync`) may run the step on another thread.

**C++:** `std::async` returns `std::future<T>`; `std::promise` sets a value for a future. No built-in chaining like CompletableFuture.

## 10. Deadlock, livelock, starvation (language view)
- **Deadlock:** two threads each hold a lock the other needs. Prevent with a **global lock ordering**, `tryLock()` with timeout, or holding fewer locks.
- **Livelock:** threads keep reacting to each other and retrying, so they're active but make no progress (both back off, retry, collide again). Fix with randomised back-off.
- **Starvation:** a thread never gets the CPU/lock (unfair locks, low priority). Fix with fairness.
