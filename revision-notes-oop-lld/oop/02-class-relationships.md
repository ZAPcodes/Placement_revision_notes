# 02 — Class Relationships ★★★

## The relationships

| Relationship | Meaning | Lifetime | Example | UML symbol |
|---|---|---|---|---|
| **Dependency** | A *uses* B temporarily (parameter, local variable, return type) | Unrelated | `Printer::print(Document d)` | Dashed arrow `- - ->` |
| **Association** | A *knows* B and holds a long-lived link | Independent | Teacher — Student | Solid line (arrow for direction) |
| **Aggregation** | Weak **has-a**: a whole *contains* parts that can exist on their own | Part outlives whole | Department — Professor | Hollow diamond on the whole ◇— |
| **Composition** | Strong **has-a**: whole *owns* parts; parts die with it | Part's lifetime bound to whole | House — Room | Filled diamond on the whole ◆— |
| **Inheritance (Generalisation)** | **is-a** | — | Dog is-a Animal | Solid line, hollow triangle ─▷ |
| **Realisation** | Class implements an interface | — | `ArrayList` implements `List` | Dashed line, hollow triangle - -▷ |

**Nesting:** Composition ⊂ Aggregation ⊂ Association. Every composition is an aggregation, and every aggregation is an association.

## Deciding in an interview
Ask: **"If the whole is destroyed, must the part be destroyed too?"**
- Yes → composition.
- No, the part is shared or reused elsewhere → aggregation.
- It's not a whole–part relationship at all, just a link → association.
- Used only inside one method call → dependency.

## How each looks in code (C++)
- **Composition:** member held by value or by `std::unique_ptr`, created inside the owner.
  `class House { Room kitchen; };`
- **Aggregation:** raw pointer / reference / `shared_ptr` passed in from outside; the owner doesn't delete it.
  `class Department { std::vector<Professor*> profs; };`
- **Association:** a reference/pointer to a peer object, often bidirectional.
- **Dependency:** only appears as a parameter or local.

**Java:** everything is a reference, so the difference is about **who creates and who can hold** the object. Composition = created inside the owner and never exposed; aggregation = passed in via constructor/setter.

## Multiplicity
One-to-one, one-to-many, many-to-many; written in UML as `1`, `0..1`, `*`, `1..*`.

> [!WARNING]
> **Tricky:**
> - "Car has an Engine" — composition *or* aggregation depending on requirements. If engines are swapped between cars in the domain (a garage system), it's aggregation. Always justify with the lifetime question.
> - Inheritance is **is-a**; everything else here is **has-a** or **uses-a**. "Stack is-a Vector" (Java's `Stack extends Vector`) is a famous design mistake — a stack should *have* a list (composition), not be one.
> - **Composition over inheritance:** prefer building behaviour by holding objects rather than extending classes. It gives runtime flexibility, avoids fragile base classes, and avoids deep hierarchies. Strategy, Decorator, and Bridge are all built on this idea.

## Quick interview answers
- *Association vs aggregation?* Aggregation is an association with a whole–part meaning; plain association has no ownership.
- *Aggregation vs composition?* Both are whole–part; composition adds exclusive ownership and a shared lifetime.
