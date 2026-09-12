# 01 — Classes & Objects ★★★

## Core idea
- **Class:** a user-defined type; a blueprint that declares data (attributes) and behaviour (methods). A class definition by itself occupies no memory for instance data.
- **Object:** a concrete instance of a class, with its own copy of the non-static data members.
- **State** = values of data members · **Behaviour** = member functions · **Identity** = the object's address (C++) or reference (Java).

## Creating objects

| | C++ | Java |
|---|---|---|
| Stack object | `Car c;` | Not possible — objects always live on the heap |
| Heap object | `Car* p = new Car();` then `delete p;` | `Car c = new Car();` (GC frees it) |
| `Car c;` means | A fully constructed object | Only a reference, initially `null` (or uninitialised local) |

> [!WARNING]
> **Tricky — most vexing parse (C++):** `Car c();` does **not** create an object. It declares a *function* named `c` that returns a `Car`. Use `Car c;` or `Car c{};`.

## `struct` vs `class` (C++)
The **only** difference: default member access and default inheritance mode are `public` for `struct` and `private` for `class`. Both can have constructors, virtual functions, inheritance, etc.

## Size of an object (C++)
- Size = non-static data members + **padding** for alignment + a hidden **vptr** if the class has any virtual function.
- Member functions (static or not) do **not** add to object size — code is stored once.
- Static data members do **not** count — they live outside objects.

> [!WARNING]
> **Tricky sizes (typical 64-bit):**
> - `class Empty {};` → `sizeof == 1`, not 0, so that two distinct objects get distinct addresses.
> - `struct S { char c; int i; };` → `8` (3 bytes of padding after `c`).
> - `struct S { char a; int i; char b; };` → `12`; reorder to `{ int i; char a; char b; }` → `8`. Member order affects size.
> - `class V { virtual void f(); };` → `8` (just the vptr).
> - Empty base class inside a derived class usually costs 0 bytes (Empty Base Optimisation).

## `static` members
- **Static data member:** one copy shared by all objects of the class; exists even if no object exists.
  - C++: declared in the class, must be **defined once outside** (`int A::count = 0;`), unless declared `inline static` (C++17) or it's a `const` integral initialised in-class.
- **Static member function:** belongs to the class, called as `A::f()`.
  - Has **no `this` pointer**, so it can access only static members directly.
  - Cannot be `virtual`, and cannot be `const` (const qualifies `this`, which doesn't exist).
- **Java:** same idea. Static methods are **not overridden**, only **hidden** (see 04).

> [!WARNING]
> **Tricky:** `A a, b[3], *p;` with a constructor doing `count++` → count is **4**. `p` is only a pointer; no object is constructed.

## The `this` pointer
- A hidden pointer passed to every non-static member function, pointing at the object the function was called on.
- Type inside a normal member function of `X`: `X* const` (you can't reseat `this`). Inside a `const` member function: `const X* const`.
- Uses: disambiguating members from parameters (`this->x = x;`), method chaining (`return *this;`), passing the current object to another function.
- Not available in static member functions.
- `delete this;` is legal only if the object was allocated with `new` and is never touched afterwards — very risky.
- **Java:** `this` is a reference; also used for constructor chaining `this(...)`.

## `const` member functions and `mutable` (C++)
- `void show() const;` promises not to modify the object. A `const` object can call **only** `const` member functions.
- A `mutable` data member can be modified even inside a `const` function (e.g. a cache or access counter).

> [!WARNING]
> **Tricky:** you can overload on `const`-ness: `int& get();` and `const int& get() const;` are two different functions. Which one is called depends on whether the object is `const`.

## Quick interview answers
- *Does a class occupy memory?* The definition doesn't store instance data; objects do. Static members and code do take memory once.
- *Can an object be created without `new` in Java?* Yes via cloning, deserialisation, reflection (`newInstance`) — but not as a stack object.
