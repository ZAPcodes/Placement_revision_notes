# 04 — Behavioural Patterns ★★★

Behavioural patterns are about **how objects communicate and split responsibilities** — who does what, and how requests flow.

GoF has 11 behavioural patterns. The CodeWithAryan course covers 10; the 11th, **Interpreter** (define a grammar and an interpreter for a small language, e.g. evaluating expressions / regex engines), is rarely asked.

---

## 1. Strategy ★★★
**Define a family of algorithms, encapsulate each one, and make them interchangeable at runtime.**
- **Participants:** `Context` (holds a reference to a strategy), `Strategy` interface, `ConcreteStrategy` classes.
- **Examples:** payment (`CreditCard`, `UPI`, `Wallet`), navigation (`Car`, `Walk`, `Transit` routes), sorting comparators (`Comparator` in Java is a strategy), pricing/discount rules, parking fee calculation.
- **Why:** removes `if/else` on algorithm type; each algorithm testable alone; add new algorithm without touching context (OCP).
- **Client usually picks** the strategy and injects it into the context.

## 2. Observer ★★★
**Define a one-to-many dependency so that when one object (subject) changes state, all its dependents (observers) are notified automatically.**
- **Participants:** `Subject` (`attach`, `detach`, `notify`), `Observer` interface (`update`), concrete observers.
- **Examples:** YouTube channel → subscribers, stock price → display widgets, event listeners in GUIs, order status → email/SMS/push notifications, MVC (model notifies views).
- **Push vs pull model:** subject sends the changed data in `update(data)` (push), or observers call back `subject.getState()` (pull).

> [!WARNING]
> **Tricky:**
> - **Lapsed listener / memory leak:** observers that are never detached stay referenced by the subject and never get garbage-collected.
> - Notification **order is not guaranteed**; don't let observers depend on each other's order.
> - An observer that modifies the subject during `notify` can cause cascades or infinite loops.
> - Notifying synchronously means one slow observer blocks the subject.
> - **Observer vs Pub-Sub:** see 06 — Pub-Sub adds a broker so publishers and subscribers don't know each other.

## 3. Iterator ★
**Provide a way to access elements of a collection sequentially without exposing its underlying representation.**
- **Participants:** `Iterator` (`hasNext()`, `next()`), `Aggregate`/`Iterable` (`createIterator()`), concrete iterators per collection.
- **Examples:** Java `Iterator`/`Iterable` (enables for-each), C++ STL iterators, iterating a tree in-order/pre-order with different iterators over the same structure.
- **External iterator:** client controls the loop (`hasNext`/`next`). **Internal iterator:** collection drives the loop and applies your function (`forEach(lambda)`).

> [!WARNING]
> **Tricky:** modifying a Java collection while iterating it (other than via `iterator.remove()`) throws `ConcurrentModificationException` — iterators are **fail-fast**. Concurrent collections provide **fail-safe / weakly consistent** iterators (see 07).

## 4. Command ★★
**Encapsulate a request as an object**, allowing you to parameterise clients with requests, queue or log them, and support **undo/redo**.
- **Participants:** `Command` (`execute()`, optionally `undo()`), `ConcreteCommand` (holds a receiver + parameters), `Receiver` (does the real work), `Invoker` (triggers commands, keeps history), `Client` (creates and wires).
- **Examples:** remote control buttons (`LightOnCommand`), text editor undo/redo stack, task queues and job schedulers, transaction logs, macro recording (a composite command). Java's `Runnable` is essentially a command.
- **Undo/redo:** keep two stacks of executed commands.

## 5. Mediator ★
**Define an object that encapsulates how a set of objects interact**, so they don't refer to each other directly. Turns many-to-many connections into many-to-one.
- **Examples:** chat room (users send to the room, room forwards), air traffic control tower (planes don't talk to each other), a dialog box coordinating its widgets, auction system.
- **Pros:** colleagues are decoupled and reusable. **Cons:** the mediator can grow into a **god object**.

## 6. State ★★
**Allow an object to change its behaviour when its internal state changes.** The object appears to change its class.
- **Participants:** `Context` (holds current state, delegates calls to it), `State` interface (one method per action), concrete states that perform the action and **decide the next state**.
- **Examples:** vending machine (`Idle` → `HasMoney` → `Dispensing`), traffic light, order lifecycle (`Placed` → `Shipped` → `Delivered`), elevator (`Idle`, `MovingUp`, `MovingDown`), document workflow (`Draft` → `Review` → `Published`), TCP connection.
- **Why:** replaces giant `switch (state)` blocks inside every method; each state's rules live in one class; invalid transitions become impossible or explicit.

## 7. Template Method ★★
**Define the skeleton of an algorithm in a base-class method, deferring some steps to subclasses.** Subclasses change specific steps without changing the overall structure.
- **Example:** `DataParser.parse()` = `open()` → `extract()` → `transform()` → `close()`; `CSVParser` and `JSONParser` override only `extract()`. Beverage: `boilWater → brew → pour → addCondiments` for Tea vs Coffee.
- The template method is usually `final` so subclasses can't reorder steps.
- **Hooks:** optional steps with a default (often empty) implementation that subclasses *may* override.
- **Hollywood Principle:** "Don't call us, we'll call you" — the base class calls the subclass steps.
- **Real-world:** Java `AbstractList`, `HttpServlet.service()` calling `doGet`/`doPost`, JUnit's setup → test → teardown.

## 8. Chain of Responsibility ★★
**Pass a request along a chain of handlers.** Each handler either processes it or forwards it to the next. The sender doesn't know which handler will handle it.
- **Examples:** logger with levels (`Info` → `Debug` → `Error`), **ATM cash dispensing** (₹2000 handler → ₹500 → ₹100), expense approval (Manager → Director → CEO by amount), servlet filters, Express/Spring middleware, customer-support escalation.
- **Pros:** decouples sender from receivers; add/reorder handlers easily.

> [!WARNING]
> **Tricky:** a request may reach the end **unhandled** — always define a default/terminal behaviour. Also, variants exist where *every* handler processes the request (filters/middleware), not just the first that can.

## 9. Visitor ★
**Add new operations to a class hierarchy without modifying the classes.** The operation lives in a visitor object.
- **Mechanism — double dispatch:** each element has `accept(Visitor v) { v.visit(this); }`. The call is resolved on **both** the element's type (via `accept`) and the visitor's type (via `visit` overload).
- **Examples:** compiler AST operations (type-check, optimise, print), shopping cart items with tax/discount visitors, file-system nodes with size-calculator and search visitors, exporting shapes to XML/JSON.

> [!WARNING]
> **Tricky:** the trade-off is the reverse of normal OOP. **Adding a new operation is easy** (new visitor), but **adding a new element type is hard** (every visitor needs a new `visit` method). Use Visitor when the class hierarchy is stable but operations change often.

## 10. Memento ★
**Capture and externalise an object's internal state so it can be restored later, without violating encapsulation.**
- **Participants:** `Originator` (creates a memento of itself and restores from one), `Memento` (the snapshot; opaque to others), `Caretaker` (stores mementos, e.g. undo stack, but never inspects them).
- **Examples:** text editor undo, game save points, form drafts, database transaction rollback, "restore to checkpoint".
- **Cons:** memory cost if snapshots are large or frequent.

> [!WARNING]
> **Tricky — Command vs Memento for undo:** Command undoes by running an **inverse operation** (small, but each command must know how to reverse itself). Memento undoes by restoring a **full snapshot** (simple, but memory-heavy). They're often combined.

---

## Summary

| Pattern | One-liner | Tell-tale sign |
|---|---|---|
| Strategy | Swap algorithms at runtime | "Multiple ways to pay/sort/price/route" |
| Observer | Notify dependents on change | "Subscribe", "notify", "alerts" |
| Iterator | Traverse without exposing internals | "Iterate over custom collection" |
| Command | Request as object | "Undo/redo", "queue", "log actions" |
| Mediator | Central hub for interactions | "Chat room", "control tower" |
| State | Behaviour depends on state | "Status transitions", vending machine, elevator |
| Template Method | Fixed skeleton, variable steps | "Same steps, one step differs" |
| Chain of Responsibility | Pass along handlers | "Escalation", "approval levels", "filters" |
| Visitor | New operations without changing classes | "Many operations on a stable hierarchy" |
| Memento | Snapshot and restore | "Undo", "save/restore state" |
