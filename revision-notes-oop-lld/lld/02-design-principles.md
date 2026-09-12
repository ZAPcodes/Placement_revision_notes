# 02 — Design Principles ★★★

## SOLID

### S — Single Responsibility Principle (SRP)
**A class should have only one reason to change** (one responsibility / one actor it serves).
- **Smell:** an `Invoice` class that calculates totals, prints itself, and saves to the database. A change to printing format, tax rules, or DB schema all touch the same class.
- **Fix:** `Invoice` (data + totals), `InvoicePrinter`, `InvoiceRepository`.

> [!WARNING]
> **Tricky:** SRP doesn't mean "one method per class". It's about one **reason to change**. Over-applying it creates dozens of tiny classes (a real criticism in interviews). **Singleton** is often said to violate SRP because the class manages both its own lifecycle and its business logic.

### O — Open/Closed Principle (OCP)
**Open for extension, closed for modification.** Add new behaviour by adding new code, not editing tested code.
- **Smell:** `calculateArea(shape)` with `if (shape is Circle) ... else if (shape is Square) ...` — every new shape edits this function.
- **Fix:** `Shape` interface with `area()`; each shape implements it. New shape = new class.
- **Patterns that implement OCP:** Strategy, Decorator, Observer, Template Method, Factory Method.

### L — Liskov Substitution Principle (LSP)
**Objects of a subclass must be usable wherever the base class is expected, without breaking correctness.**
- **Classic violation 1 — Square/Rectangle:** `Square extends Rectangle`. Code that does `r.setWidth(5); r.setHeight(4); assert r.area() == 20` fails for a Square.
- **Classic violation 2 — Bird/Penguin:** `Bird` has `fly()`; `Penguin extends Bird` throws in `fly()`. Fix: split into `Bird` and `FlyingBird`.
- **Rules for a valid subtype:**
  - Preconditions **cannot be strengthened** (can't demand more of inputs).
  - Postconditions **cannot be weakened** (can't promise less).
  - Invariants of the base must be preserved.
  - No new exceptions the base didn't declare.

> [!WARNING]
> **Tricky:** an overridden method that throws `UnsupportedOperationException` or does nothing is an LSP red flag. Java's own `Collections.unmodifiableList()` is a known example — it implements `List` but throws on `add()`.

### I — Interface Segregation Principle (ISP)
**Clients should not be forced to depend on methods they don't use.** Prefer many small, role-specific interfaces over one fat interface.
- **Smell:** `Worker` interface with `work()` and `eat()`; `RobotWorker` has to implement `eat()` as a no-op.
- **Fix:** `Workable`, `Eatable` separately.
- **Smell 2:** a `Printer` interface with `print()`, `scan()`, `fax()` — a basic printer is forced to stub `scan` and `fax`.

### D — Dependency Inversion Principle (DIP)
1. **High-level modules shouldn't depend on low-level modules; both should depend on abstractions.**
2. **Abstractions shouldn't depend on details; details depend on abstractions.**
- **Smell:** `OrderService` does `new MySQLDatabase()` inside itself. Can't switch to MongoDB or mock it in tests.
- **Fix:** `OrderService` depends on a `Database` interface; the concrete DB is passed in.

> [!WARNING]
> **Tricky — DIP vs DI vs IoC (often confused):**
> - **DIP** is a *principle* (depend on abstractions).
> - **Dependency Injection** is a *technique* to satisfy it: dependencies are supplied from outside — via **constructor** (preferred, makes dependencies explicit and immutable), **setter**, or **interface/method** injection.
> - **Inversion of Control** is the broader idea that a framework/container controls object creation and flow (Spring's IoC container does DI for you).

## DRY — Don't Repeat Yourself
Every piece of **knowledge** should have one authoritative representation.
- Duplicated tax logic in three places → a fix in one place leaves two bugs.
- Fix with functions, shared classes, constants, config.

> [!WARNING]
> **Tricky:** DRY is about duplicated **knowledge**, not duplicated-looking code. Two functions that happen to look alike but change for different reasons should **stay separate** — merging them creates the "wrong abstraction". Some say *duplication is cheaper than the wrong abstraction.*

## KISS — Keep It Simple, Stupid
Prefer the simplest solution that works. Avoid clever one-liners, unnecessary abstraction layers, and premature optimisation. Simple code is easier to read, test, and change.

## YAGNI — You Aren't Gonna Need It
Don't build features or extension points until they are actually needed. Speculative generality adds code to maintain and often guesses the future wrong.

> [!WARNING]
> **Tricky:** YAGNI vs OCP seems contradictory. Resolution: design so that code *can* be extended cheaply (clean interfaces, SRP), but don't *implement* the extensions until required.

## Other principles worth knowing

| Principle | Meaning |
|---|---|
| **High cohesion** | Members of a class/module are closely related and focused on one purpose. Good. |
| **Low coupling** | Modules depend on each other as little as possible, via abstractions. Good. |
| **Composition over inheritance** | Build behaviour by holding objects; inherit only for true is-a with substitutability. |
| **Law of Demeter** (principle of least knowledge) | Talk only to your immediate friends. Avoid `a.getB().getC().doX()` chains ("train wrecks"). |
| **Program to an interface, not an implementation** | Declare `List<X> l = new ArrayList<>()`, not `ArrayList<X>`. |
| **Separation of concerns** | Split a program into distinct sections each handling one concern (e.g. MVC). |

**Aim: high cohesion, low coupling.**

## Quick interview answers
- *Which SOLID principle does the Strategy pattern support?* OCP (new strategies without changing context), and DIP (context depends on strategy interface).
- *Which principle does a fat interface violate?* ISP.
- *A subclass overrides a method to throw an exception — which principle?* LSP.
- *A class that both computes and persists data?* SRP.
- *`new ConcreteClass()` scattered in business logic?* DIP (fix with injection or a factory).
