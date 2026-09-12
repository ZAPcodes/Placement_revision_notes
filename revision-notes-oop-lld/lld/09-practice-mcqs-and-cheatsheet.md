# 09 — Practice MCQs + Cheatsheet (read before OA / interview)

## Part A — Practice MCQs

Answers are hidden under each question.

### Principles
**1.** A `Report` class that generates content, formats it as PDF, and emails it violates:
(a) OCP (b) SRP (c) LSP (d) ISP
<details><summary>Answer</summary>(b) — three reasons to change.</details>

**2.** Adding a new shape requires editing an `if/else` chain in `AreaCalculator`. Which principle is violated?
(a) SRP (b) OCP (c) DIP (d) YAGNI
<details><summary>Answer</summary>(b)</details>

**3.** `Penguin extends Bird` overrides `fly()` to throw an exception. This violates:
(a) ISP (b) LSP (c) DRY (d) KISS
<details><summary>Answer</summary>(b)</details>

**4.** A `Machine` interface with `print()`, `scan()`, `fax()` forces a basic printer to stub two methods. Violation:
(a) ISP (b) SRP (c) OCP (d) DIP
<details><summary>Answer</summary>(a)</details>

**5.** "High-level modules should depend on abstractions, not concretions" is:
(a) Dependency Injection (b) Dependency Inversion Principle (c) Inversion of Control (d) Law of Demeter
<details><summary>Answer</summary>(b). DI is a technique to achieve it.</details>

**6.** The preferred form of dependency injection for mandatory dependencies is:
(a) Setter (b) Field (c) Constructor (d) Static
<details><summary>Answer</summary>(c)</details>

**7.** `order.getCustomer().getAddress().getCity()` most directly violates:
(a) Law of Demeter (b) LSP (c) DRY (d) YAGNI
<details><summary>Answer</summary>(a)</details>

**8.** Good design aims for:
(a) High coupling, low cohesion (b) Low coupling, high cohesion (c) High coupling, high cohesion (d) Low coupling, low cohesion
<details><summary>Answer</summary>(b)</details>

### Creational
**9.** Which pattern solves the telescoping constructor problem?
(a) Factory (b) Prototype (c) Builder (d) Singleton
<details><summary>Answer</summary>(c)</details>

**10.** In double-checked locking in Java, `volatile` on the instance field prevents:
(a) Deadlock (b) Seeing a partially constructed object due to reordering (c) Multiple class loaders (d) Serialization issues
<details><summary>Answer</summary>(b)</details>

**11.** Which singleton implementation is safe against reflection and serialization by default?
(a) Eager (b) DCL (c) Bill Pugh holder (d) Enum
<details><summary>Answer</summary>(d)</details>

**12.** Which method prevents deserialization from creating a second singleton instance?
(a) `clone()` (b) `readResolve()` (c) `finalize()` (d) `hashCode()`
<details><summary>Answer</summary>(b)</details>

**13.** Creating families of related objects (Windows button + Windows checkbox) uses:
(a) Factory Method (b) Abstract Factory (c) Builder (d) Prototype
<details><summary>Answer</summary>(b)</details>

**14.** Factory Method relies primarily on:
(a) Composition (b) Inheritance — subclasses decide the concrete class (c) Cloning (d) A static map
<details><summary>Answer</summary>(b)</details>

**15.** When object creation is expensive and you copy a preconfigured instance:
(a) Prototype (b) Flyweight (c) Builder (d) Proxy
<details><summary>Answer</summary>(a)</details>

### Behavioural
**16.** Choosing a payment method (UPI/Card/Wallet) at runtime is best modelled by:
(a) State (b) Strategy (c) Template Method (d) Visitor
<details><summary>Answer</summary>(b)</details>

**17.** A vending machine whose response to `insertCoin()` depends on whether it's idle, has money, or is out of stock:
(a) Strategy (b) State (c) Command (d) Observer
<details><summary>Answer</summary>(b)</details>

**18.** Undo/redo in a text editor using objects representing actions:
(a) Command (b) Mediator (c) Iterator (d) Strategy
<details><summary>Answer</summary>(a) — Memento is the snapshot alternative.</details>

**19.** ATM dispensing ₹2000, then ₹500, then ₹100 notes through linked handlers:
(a) Decorator (b) Chain of Responsibility (c) Composite (d) Observer
<details><summary>Answer</summary>(b)</details>

**20.** Which pattern uses double dispatch?
(a) Visitor (b) Strategy (c) Observer (d) Adapter
<details><summary>Answer</summary>(a)</details>

**21.** With Visitor, which change is **hard**?
(a) Adding a new operation (b) Adding a new element class (c) Adding a new visitor (d) Traversing the structure
<details><summary>Answer</summary>(b)</details>

**22.** A base class defines `final process()` calling `read()`, `transform()`, `write()`, with subclasses overriding `transform()`:
(a) Strategy (b) Template Method (c) Builder (d) Facade
<details><summary>Answer</summary>(b)</details>

**23.** Chat room where users send messages to a central object that forwards them:
(a) Observer (b) Mediator (c) Facade (d) Proxy
<details><summary>Answer</summary>(b)</details>

**24.** The key difference between Strategy and State is:
(a) State uses interfaces (b) In State, the states themselves trigger transitions; in Strategy the client chooses (c) Strategy can't be changed at runtime (d) No difference
<details><summary>Answer</summary>(b)</details>

### Structural
**25.** Java's `new BufferedReader(new FileReader(f))` is an example of:
(a) Adapter (b) Decorator (c) Proxy (d) Bridge
<details><summary>Answer</summary>(b)</details>

**26.** Making a third-party SDK conform to your existing interface:
(a) Facade (b) Adapter (c) Bridge (d) Composite
<details><summary>Answer</summary>(b)</details>

**27.** Files and folders both supporting `getSize()`:
(a) Composite (b) Decorator (c) Flyweight (d) Visitor
<details><summary>Answer</summary>(a)</details>

**28.** Loading a large image only when it's first displayed:
(a) Virtual Proxy (b) Flyweight (c) Facade (d) Builder
<details><summary>Answer</summary>(a)</details>

**29.** Avoiding `m × n` subclasses for Shape × Color:
(a) Bridge (b) Adapter (c) Decorator (d) Composite
<details><summary>Answer</summary>(a)</details>

**30.** The Java String pool is an example of:
(a) Singleton (b) Flyweight (c) Prototype (d) Proxy
<details><summary>Answer</summary>(b)</details>

**31.** Which pattern changes the interface of the wrapped object?
(a) Decorator (b) Proxy (c) Adapter (d) Composite
<details><summary>Answer</summary>(c)</details>

### Concurrency
**32.** Calling `t.run()` directly instead of `t.start()`:
(a) Starts a new thread (b) Runs on the current thread (c) Throws exception (d) Deadlocks
<details><summary>Answer</summary>(b)</details>

**33.** Which releases the monitor lock?
(a) `sleep()` (b) `wait()` (c) `yield()` (d) `join()` on another thread while holding a lock
<details><summary>Answer</summary>(b)</details>

**34.** Calling `wait()` outside a synchronized block throws:
(a) `InterruptedException` (b) `IllegalMonitorStateException` (c) `IllegalThreadStateException` (d) Nothing
<details><summary>Answer</summary>(b)</details>

**35.** `volatile int count; count++` from multiple threads is:
(a) Thread-safe (b) Not thread-safe — volatile gives visibility, not atomicity (c) Compile error (d) Deadlocks
<details><summary>Answer</summary>(b)</details>

**36.** `ConcurrentHashMap` allows:
(a) Null keys only (b) Null values only (c) Both (d) Neither
<details><summary>Answer</summary>(d)</details>

**37.** A `ThreadPoolExecutor` with core = 2, max = 10, and an unbounded `LinkedBlockingQueue` will run at most how many threads?
(a) 2 (b) 10 (c) Unlimited (d) 12
<details><summary>Answer</summary>(a) — the queue never fills, so the pool never grows past core.</details>

**38.** Which is reusable after all threads reach it?
(a) `CountDownLatch` (b) `CyclicBarrier` (c) `Semaphore(0)` (d) `Future`
<details><summary>Answer</summary>(b)</details>

**39.** Key difference between a mutex and a binary semaphore:
(a) Speed (b) Ownership — only the locking thread can unlock a mutex (c) Semaphores can't block (d) None
<details><summary>Answer</summary>(b)</details>

**40.** Dining philosophers deadlock is prevented by making one philosopher pick up forks in the opposite order. This breaks:
(a) Mutual exclusion (b) Hold and wait (c) No preemption (d) Circular wait
<details><summary>Answer</summary>(d)</details>

**41.** In `CompletableFuture`, chaining a function that itself returns a `CompletableFuture` without nesting uses:
(a) `thenApply` (b) `thenCompose` (c) `thenAccept` (d) `thenRun`
<details><summary>Answer</summary>(b)</details>

---

## Part B — Cheatsheet

### SOLID in one line each
| | Principle | Violation smell |
|---|---|---|
| **S** | One reason to change | God class doing logic + I/O + persistence |
| **O** | Extend without modifying | `if/else`/`switch` on type that grows |
| **L** | Subtypes substitutable | Override throws / does nothing (Square–Rectangle, Penguin–Bird) |
| **I** | Small focused interfaces | Stub methods implementing a fat interface |
| **D** | Depend on abstractions | `new ConcreteX()` inside business logic |

DRY = one source of knowledge · KISS = simplest thing that works · YAGNI = don't build it until needed.

### All 22 course patterns (+ Interpreter)
| Creational | Structural | Behavioural |
|---|---|---|
| Factory (Method) — subclass picks class | Adapter — convert interface | Strategy — swap algorithm |
| Abstract Factory — families | Composite — tree, uniform | Observer — notify dependents |
| Builder — step-by-step | Facade — simple front | Iterator — traverse hidden structure |
| Singleton — one instance | Decorator — wrap to add | Command — request as object, undo |
| Prototype — clone | Bridge — split two dimensions | Mediator — central hub |
| | Proxy — controlled stand-in | State — behaviour by state |
| | Flyweight — share intrinsic state | Template Method — skeleton + steps |
| | | Chain of Responsibility — pass along |
| | | Visitor — new ops, double dispatch |
| | | Memento — snapshot/restore |
| | | *(Interpreter — grammar evaluation)* |

### Keyword → pattern
| If the question says... | Think |
|---|---|
| "multiple ways to pay / price / route / sort" | Strategy |
| "notify / subscribe / alert" | Observer (Pub-Sub if broker) |
| "create object by type" | Factory |
| "only one instance" | Singleton |
| "many optional fields", "immutable" | Builder |
| "status changes", "vending machine", "elevator", "traffic light" | State |
| "undo / redo", "queue of actions" | Command (+ Memento) |
| "add-ons / toppings / layers" | Decorator |
| "legacy / third-party API" | Adapter |
| "folder/file, hierarchy, org chart" | Composite |
| "one call, many subsystems" | Facade |
| "approval levels / escalation / filters" | Chain of Responsibility |
| "lazy load / access control / caching" | Proxy |
| "millions of similar objects" | Flyweight |
| "same steps, one differs" | Template Method |

### Must-know comparisons (one sentence each)
- **Strategy vs State:** client picks the strategy; states pick the next state.
- **Factory vs Strategy:** factory *creates* objects; strategy *executes* behaviour.
- **Factory Method vs Abstract Factory:** one product via inheritance vs families via composition.
- **Adapter vs Decorator vs Proxy:** changes interface / adds behaviour / controls access.
- **Facade vs Mediator:** one-way simplification vs two-way coordination.
- **Observer vs Pub-Sub:** direct references vs broker in between.
- **Template Method vs Strategy:** inheritance vs composition.
- **Command vs Memento undo:** inverse operation vs snapshot.

### Singleton checklist
Private constructor · static instance · `getInstance()` · thread safety (DCL + `volatile`, holder class, enum; Meyers singleton in C++11) · break-proofing (reflection guard, `readResolve`, `clone` override) · criticisms (global state, testing, SRP).

### Concurrency quick facts
- `start()` new thread; `run()` same thread; `start()` twice → `IllegalThreadStateException`.
- `sleep` keeps lock; `wait` releases it and needs `synchronized`; always `wait` in a `while`.
- `volatile` = visibility, not atomicity. `Atomic*` = CAS.
- `synchronized` static → Class lock; instance → `this`. Java locks are reentrant.
- Unlock in `finally`. `tryLock` helps avoid deadlock.
- Mutex has ownership; semaphore doesn't.
- CountDownLatch one-shot; CyclicBarrier reusable.
- CHM: bucket-level locking + CAS, no nulls, use `computeIfAbsent`/`putIfAbsent` for check-then-act.
- Pool growth: core → queue → max → reject. Unbounded queue ⇒ never exceeds core.
- `thenApply` = map, `thenCompose` = flatMap, `thenCombine` = merge two.
- Deadlock fixes: lock ordering, timeouts, fewer locks. Livelock fix: random back-off.

### LLD interview flow
Requirements → entities → relationships → interfaces → patterns (only where needed) → walk a use case → concurrency → extensibility.
