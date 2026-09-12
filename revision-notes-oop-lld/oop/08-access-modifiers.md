# 08 — Access Modifiers & Friend ★★

## C++

| Modifier | Same class | Derived class | Outside |
|---|---|---|---|
| `private` | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ |

- Default: `private` in `class`, `public` in `struct`.
- For how access changes under public/protected/private **inheritance**, see the table in 05.

## Java

| Modifier | Same class | Same package | Subclass (other package) | Everywhere |
|---|---|---|---|---|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *default* (no keyword, package-private) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

> [!WARNING]
> **Tricky:**
> - **Java `protected` ≠ C++ `protected`:** Java's also grants access to **every class in the same package**, subclass or not.
> - Java default access is **more restrictive** than `protected` — a subclass in another package can't see default members.
> - A Java **top-level class** can only be `public` or default (not `private`/`protected`). Nested classes can use all four.
> - Only **one public top-level class** per `.java` file, and the file must be named after it.
> - **Java overriding cannot reduce visibility:** overriding a `public` method as `protected` is a compile error. **C++ allows it**, and access is checked on the **static type** of the pointer/reference.
> - Access control is **per class, not per object** in both languages: `other.privateField` is fine inside a member function of the same class.
> - A `private` constructor stops instantiation from outside → Singleton, static utility classes, factory methods.

## `friend` (C++)
A `friend` function or class declared inside a class can access that class's private and protected members.

```cpp
class Box {
    double w;
    friend void print(const Box& b);   // non-member, but can read b.w
    friend class Inspector;            // every member of Inspector can access Box's privates
};
```

**Properties (all common MCQ traps):**
- A friend function is **not a member** — no `this`, not called with `obj.`, and access specifiers don't affect where it is declared.
- Friendship is **not inherited** — a friend of `Base` is not a friend of `Derived`.
- Friendship is **not transitive** — a friend of your friend isn't your friend.
- Friendship is **not mutual** — if `A` declares `B` a friend, `A` still can't access `B`'s privates.
- Typical use: overloading `operator<<`/`operator>>` and symmetric binary operators.

**Java:** no `friend`. The closest is package-private access or nested classes.

## Quick interview answers
- *Does `friend` violate encapsulation?* It weakens it, but in a controlled way — the class itself decides who its friends are, so it's often described as extending the class's interface.
- *What access does a nested class have?* C++ nested classes can access the enclosing class's private members (C++11). Java inner classes can access the outer class's private members too.
