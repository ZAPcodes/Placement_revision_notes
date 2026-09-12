# 06 — Encapsulation ★★

**Encapsulation:** bundling data and the methods that operate on it into one unit (the class), and **restricting direct access** to the data so it can only change through controlled methods.

**Data hiding** is the mechanism (private members); encapsulation is the broader design idea.

## Why it matters
- **Protects invariants:** a `BankAccount` can reject negative deposits in `deposit()`; with a public `balance` field anyone can set it to anything.
- **Change internals freely:** swap the storage (e.g. store cents instead of rupees) without touching callers.
- **Easier debugging:** all changes to the state go through a few methods.
- **Read-only / write-only access:** provide just a getter, or just a setter.

## Achieving it
- Make data members `private`.
- Expose `public` methods that validate inputs (setters) and present data (getters).
- A class whose data members are **all** private is called **tightly encapsulated**.

## Encapsulation vs Abstraction (asked constantly)

| | Encapsulation | Abstraction |
|---|---|---|
| Focus | **How** data is protected | **What** the object does |
| Hides | Internal **state** | Implementation **complexity** |
| Level | Implementation level | Design level |
| Achieved by | Access modifiers, getters/setters | Abstract classes, interfaces |
| Analogy | Capsule wrapping the medicine | Car pedal: you press, you don't see the engine |

One line: *abstraction hides complexity from the user; encapsulation hides data from misuse.*

> [!WARNING]
> **Tricky:**
> - **Getters can break encapsulation.** Returning a non-const reference/pointer to a private member (`int& getX()`), or returning an internal mutable `List` in Java, lets callers modify private state. Return by value, `const&`, or an unmodifiable copy (`Collections.unmodifiableList`, `List.copyOf`).
> - A class with a public getter and setter for **every** field, and no validation, is encapsulated only on paper — it's effectively a struct.
> - **Access is per class, not per object:** a member function can read the private members of *another* object of the same class (that's how copy constructors work).
> - `friend` (C++) deliberately grants access to private members — used sparingly, e.g. for `operator<<`. It weakens encapsulation.
> - **Java:** `final` fields + no setters + defensive copies = an **immutable** class (like `String`), the strongest form of encapsulation.
