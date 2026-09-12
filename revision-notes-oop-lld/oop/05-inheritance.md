# 05 — Inheritance ★★★

**Inheritance:** a derived (child) class acquires the members of a base (parent) class, modelling an **is-a** relationship. Benefits: code reuse, a common interface for polymorphism, hierarchical modelling.

## Types

| Type | Shape | C++ | Java (classes) |
|---|---|---|---|
| Single | A → B | ✅ | ✅ |
| Multilevel | A → B → C | ✅ | ✅ |
| Hierarchical | A → B, A → C | ✅ | ✅ |
| Multiple | A, B → C | ✅ | ❌ (only via interfaces) |
| Hybrid | combination, e.g. diamond | ✅ | ❌ (only via interfaces) |

## Modes of inheritance (C++)
How base members appear in the derived class:

| Base member ↓ / Mode → | `public` | `protected` | `private` |
|---|---|---|---|
| public | public | protected | private |
| protected | protected | protected | private |
| private | inaccessible | inaccessible | inaccessible |

- Default mode: `private` for `class`, `public` for `struct`.
- Only **public** inheritance models is-a; a `Base*` can point to the derived object only with public inheritance (from outside the class).
- Private inheritance ≈ "implemented-in-terms-of" (prefer composition).

> [!WARNING]
> **Tricky:** private members of the base **are inherited** — they're physically inside the derived object (they count in `sizeof`) — they just aren't **accessible** by name in the derived class.

## What is NOT inherited (C++)
Constructors (unless `using Base::Base;`), destructor, copy/move assignment operators (the derived class's implicit ones hide them), and friendships.

## Construction / destruction order
1. Virtual base classes (depth-first, left to right)
2. Non-virtual base classes, in the order **listed in the class head** (not the initializer list)
3. Data members, in **declaration** order
4. Derived constructor body

Destruction: exactly the reverse.

> [!WARNING]
> **Tricky:** `class C : public B, public A` constructs `B` before `A`, even if C's initializer list says `: A(), B()`.

## The diamond problem
```
      A
     / \
    B   C
     \ /
      D
```
If `B` and `C` both inherit `A` normally, `D` contains **two copies** of `A`'s members → accessing `d.x` or calling `d.f()` from `A` is **ambiguous** (compile error), and the data is duplicated.

**C++ fix — virtual inheritance:** `class B : virtual public A`, `class C : virtual public A`. Now `D` has **one shared** `A` subobject.

> [!WARNING]
> **Tricky (virtual inheritance):**
> - The **most-derived class** (`D`) is responsible for constructing the virtual base `A`. Calls to `A`'s constructor from `B`'s and `C`'s initializer lists are **ignored** when constructing a `D`.
> - If `A` has no default constructor, `D` must call `A(...)` explicitly.
> - If `B` and `C` both override a virtual function of `A` and `D` doesn't, the call is still ambiguous → `D` must override it.

**Java:** class multiple inheritance isn't allowed, so the classic diamond can't happen with classes. With **default methods** in interfaces:
1. A method from a **class** always wins over an interface default.
2. Otherwise the **more specific** interface (the sub-interface) wins.
3. Otherwise the class **must override**, and can pick one with `InterfaceName.super.method()`.

## Casting in hierarchies (C++)
- **Upcasting** (derived → base): implicit and always safe.
- **Downcasting** (base → derived): use `dynamic_cast` (needs at least one virtual function in the base). Failure returns `nullptr` for pointers, throws `std::bad_cast` for references. `static_cast` downcast is unchecked — wrong type gives undefined behaviour.
- **Java:** downcast with `(Derived) obj`; wrong type throws `ClassCastException`; check with `instanceof`.

## Java specifics
- `extends` (one class), `implements` (many interfaces). Every class implicitly extends `Object`.
- `super.method()` calls the parent's version; `super(...)` calls the parent constructor.
- `final class` cannot be extended (`String`, wrapper classes). `final` method cannot be overridden.
- Private and static methods aren't overridden.

## When not to inherit
- Inheritance is tight coupling: base-class changes ripple into derived classes (fragile base class problem).
- If "B is-a A" fails under substitution (see **Liskov Substitution** in LLD 02), don't inherit. Classic: `Square extends Rectangle`.
- Prefer composition when you only want to reuse code, not the interface.

## Quick interview answers
- *Why doesn't Java support multiple inheritance of classes?* To avoid diamond ambiguity and complexity; interfaces give multiple type inheritance without state conflicts.
- *Does a derived class inherit the constructor?* No (C++ can opt in with `using Base::Base`).
- *Can you inherit from a class with a private constructor?* Not if the derived class has to call it (unless it's a friend / nested class).
