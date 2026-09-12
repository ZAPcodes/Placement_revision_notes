# 08 — Concurrency Problems (concepts, no code) ★★

For each problem: what's being tested, the approach in words, which primitive fits, and the traps. These appear in LeetCode's concurrency section and as follow-ups in LLD rounds.

---

## 1. Print Zero Even Odd
**Problem:** three threads share one object. Thread A calls `zero()`, B calls `even()`, C calls `odd()`. Output for n = 5 must be `0102030405`.

**Tests:** strict **turn-taking** between threads.

**Approach:** keep the "whose turn is it" information in shared state and make each thread wait until it's their turn.
- **Semaphores (cleanest):** `zeroSem = 1`, `oddSem = 0`, `evenSem = 0`. Zero thread acquires `zeroSem`, prints 0, then releases `oddSem` or `evenSem` depending on the next number. Odd/even threads acquire their semaphore, print, release `zeroSem`.
- **Alternative:** one lock + a `turn` variable + `wait()`/`notifyAll()` in a `while` loop.

> [!WARNING]
> **Traps:** using `if` instead of `while` around `wait()`; using `notify()` and waking the wrong thread → hang; forgetting that zero must run n times, odd ⌈n/2⌉ times, even ⌊n/2⌋ times.

## 2. FizzBuzz Multithreaded
**Problem:** four threads: `fizz` (divisible by 3 only), `buzz` (5 only), `fizzbuzz` (15), `number` (neither). Print 1..n in order with the right thread printing each.

**Tests:** **condition-based** coordination, where the condition depends on a shared counter.

**Approach:** shared `current` counter. Each thread loops: wait until `current > n` (exit) or `current` matches *its* condition; print; increment; wake everyone.
- Lock + `Condition`/`notifyAll`, or a semaphore per thread with the number thread acting as a dispatcher.

> [!WARNING]
> **Traps:** threads must also exit cleanly once `current > n` — otherwise some wait forever. Check the fizzbuzz (15) condition before 3 and 5.

## 3. Design a Bounded Blocking Queue
**Problem:** a queue with capacity `k`: `enqueue` blocks when full, `dequeue` blocks when empty, `size()` works under concurrency.

**Tests:** the **producer–consumer** problem — the single most common concurrency interview question.

**Approach options:**
- **One lock + two conditions** (`notFull`, `notEmpty`): enqueue waits on `notFull`, then signals `notEmpty`; dequeue waits on `notEmpty`, then signals `notFull`. This is how `ArrayBlockingQueue` works.
- **Two semaphores + a mutex:** `emptySlots = k`, `filledSlots = 0`, and a mutex protecting the underlying queue.
- **synchronized + wait/notifyAll** (simplest to write).

> [!WARNING]
> **Traps:** `while` not `if` around waits; with a single `wait()` queue use `notifyAll`, not `notify`; acquire the counting semaphore **before** the mutex (reverse order can deadlock — a producer holding the mutex waits for a slot while consumers can't get the mutex to free one).

## 4. The Dining Philosophers
**Problem:** 5 philosophers around a table, 5 forks, one between each pair. A philosopher needs both adjacent forks to eat.

**Tests:** **deadlock** (everyone picks up their left fork and waits for the right) and **starvation**.

**Solutions — each breaks one Coffman condition:**
| Solution | How | Breaks |
|---|---|---|
| **Resource ordering** | Number forks; always pick the lower-numbered first (equivalently, one philosopher picks right-first) | Circular wait |
| **Limit diners** | Semaphore allowing at most 4 philosophers to try at once | Circular wait (one always has both forks available) |
| **All-or-nothing** | Pick up both forks atomically (under a lock), or `tryLock` both and release if the second fails | Hold and wait |
| **Arbitrator / waiter** | A central waiter grants permission to pick up forks | Hold and wait (centralised) |

> [!WARNING]
> **Traps:** the `tryLock`-and-release approach can **livelock** (everyone repeatedly picks, fails, drops) — add random back-off. Resource ordering prevents deadlock but not necessarily starvation.

## 5. Design a Multithreaded Web Crawler
**Problem:** starting from a URL, crawl all pages under the same hostname using multiple threads; don't visit a URL twice.

**Tests:** **shared visited set**, **work distribution**, and knowing **when to stop**.

**Approach:**
- A **thread-safe visited set** (`ConcurrentHashMap.newKeySet()`); use its atomic `add()` return value to claim a URL — `if (visited.add(url)) submit(url)` — no separate check-then-add race.
- A **thread pool** (executor) processes URLs; each task fetches a page, extracts links, filters by hostname, submits unvisited ones.
- **Termination:** track in-flight tasks (an `AtomicInteger` or a `Phaser`), or use `CompletableFuture`s and wait on all of them; stop when no tasks remain.
- Alternative: BFS with a shared `BlockingQueue` frontier and N worker threads.

> [!WARNING]
> **Traps:** `contains()` then `add()` → two threads crawl the same URL; hostname extraction bugs (`http://a.com` vs `http://a.com.evil`); ending too early when the queue is momentarily empty but tasks are still running.

---

## Primitive selection cheat table

| Need | Use |
|---|---|
| Mutual exclusion on a small critical section | `synchronized` / `ReentrantLock` / `std::mutex` |
| Wait until a condition holds | `wait/notifyAll` in a `while`, `Condition`, `std::condition_variable` |
| Limit concurrent access to N | Counting `Semaphore` |
| Strict turn order between threads | Semaphores (one per thread / turn) |
| Producer–consumer | `BlockingQueue` (or lock + 2 conditions) |
| Wait for N tasks to finish once | `CountDownLatch` / `join` / `allOf` |
| Many reads, few writes | `ReadWriteLock`, `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| Single counter/flag | `AtomicInteger` / `volatile boolean` |
