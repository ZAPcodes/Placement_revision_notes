# 03 — Constructors & Destructors ★★★

## Constructor basics
- Special member function with the **same name as the class** and **no return type** (not even `void`).
- Called automatically when an object is created; initialises the object into a valid state.
- Can be overloaded. Cannot be `virtual`, `static`, or `const` (C++).
- Can be `private` (used for Singleton, factory methods, or preventing instantiation).

## Types (C++)

| Type | Signature | Notes |
|---|---|---|
| Default | `A()` | Compiler generates one **only if you declare no constructor at all** |
| Parameterized | `A(int x)` | |
| Copy | `A(const A& other)` | Creates a new object as a copy of an existing one |
| Move (C++11) | `A(A&& other)` | Steals resources from a temporary |
| Delegating (C++11) | `A() : A(0) {}` | One constructor calls another |
| Converting | Any single-argument constructor | Enables implicit conversion `A a = 5;` unless marked `explicit` |

> [!WARNING]
> **Tricky — the missing default constructor:** if you write `A(int x)` only, then `A a;` fails to compile. The compiler stops generating the default constructor once any constructor is declared. Fix with `A() = default;`.

## Copy constructor
**Called when:** an object is initialised from another object (`A b = a;` or `A b(a);`), passed **by value**, or returned **by value** (though the compiler usually elides this — RVO).
**Not called when:** assigning to an already existing object (`b = a;` calls the **copy assignment operator**).

> [!WARNING]
> **Tricky:**
> - `A b = a;` is **copy construction**, not assignment, despite the `=`.
> - Why must the parameter be a **reference**? Passing by value would itself need a copy, which calls the copy constructor again → infinite recursion. The compiler rejects `A(A other)`.
> - Why `const`? So you can copy from `const` objects and temporaries.

## Shallow vs deep copy
- **Shallow copy** (what the compiler-generated copy does): copies every member bit by bit. For pointer members, both objects now point to the **same** heap memory.
  - Problems: changing one affects the other; both destructors `delete` the same memory → **double free** / crash; dangling pointers.
- **Deep copy:** allocate new memory and copy the pointed-to contents. Needs a user-written copy constructor and copy assignment operator.
- **Java:** assignment copies references. `clone()` is shallow by default; a deep copy needs overriding `clone()` or a copy constructor written by hand.

## Rule of 3 / 5 / 0
- **Rule of 3:** if a class needs a custom destructor, copy constructor, **or** copy assignment operator, it almost certainly needs all three (it manages a resource).
- **Rule of 5 (C++11):** add move constructor and move assignment operator.
- **Rule of 0:** best practice — use RAII members (`std::string`, `std::vector`, smart pointers) so you need to write none of them.

## Member initializer list
`A(int v) : x(v), ref(r), c(10) {}`

**Required for:** `const` members, reference members, members without a default constructor, base classes without a default constructor.
Also more efficient than assigning in the body (avoids default-construct-then-assign).

> [!WARNING]
> **Tricky — initialization order:** members are initialised in the order they are **declared in the class**, **not** the order written in the initializer list.
> ```cpp
> class A { int x; int y;
>   A(int v) : y(v), x(y + 1) {}   // x is initialised FIRST, using garbage y
> };
> ```

## Virtual calls inside constructors

> [!WARNING]
> **Tricky (very common):**
> - **C++:** during the base-class constructor, the object's dynamic type *is* the base class (vptr points to the base vtable). A virtual call inside the base constructor runs the **base** version. Same inside destructors.
> - **Java:** the opposite — dispatch goes to the **derived** override, even though the derived fields haven't been initialised yet (they hold default values like `0`/`null`). Classic output question: prints `0`.

## Destructors
- `~A()`: no parameters, no return type, **cannot be overloaded** (exactly one per class).
- Called automatically when a stack object goes out of scope, or on `delete` for heap objects.
- Order: **reverse of construction** — derived destructor body, then members (reverse declaration order), then base.
- Should not throw exceptions (destructors are implicitly `noexcept` since C++11; throwing during stack unwinding calls `std::terminate`).
- **Virtual destructor:** required when deleting a derived object via a base pointer. Without it, only the base destructor runs → resource leak (formally undefined behaviour).
- A destructor *can* be virtual; a constructor *cannot*.

**Java:** no destructors. Garbage collector reclaims memory; `finalize()` is deprecated. Use `try-with-resources` / `AutoCloseable` for cleanup.

## Java constructor specifics
- `this(...)` or `super(...)` must be the **first statement**; you can't have both.
- If you don't call `super(...)`, the compiler inserts `super()` — compile error if the parent has no no-arg constructor.
- Constructors are not inherited and cannot be `final`, `static`, or `abstract`.

## Quick interview answers
- *Why can't a constructor be virtual?* Virtual dispatch needs the vptr, which is set up **by** the constructor; and when creating an object you always know its exact type.
- *Can a constructor throw?* Yes. The object is considered never constructed, so its destructor does **not** run, but already-constructed members and bases **are** destroyed.
- *Private destructor?* Forces heap-only allocation (object must be destroyed through a member function).
- *Can we call a constructor explicitly?* `A()` creates a temporary; placement new constructs at a given address.
