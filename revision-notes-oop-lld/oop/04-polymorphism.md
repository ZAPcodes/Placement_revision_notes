# 04 — Polymorphism ★★★

**Polymorphism:** one interface, many forms. The same call behaves differently depending on the type involved.

| | Compile-time (static) | Run-time (dynamic) |
|---|---|---|
| Mechanisms | Function overloading, operator overloading, templates | Virtual functions + overriding |
| Resolved | By compiler, from **static type** | At runtime, from **dynamic type** (actual object) |
| Binding | Early binding | Late binding |
| Cost | None | One extra indirection (vptr → vtable → function); blocks inlining |

## Function overloading
- Same name, same scope, different **parameter list** (number, type, or order).
- **Return type alone cannot distinguish** overloads → compile error.
- Also can overload on `const` member functions, and on `&` / `&&` qualifiers.

> [!WARNING]
> **Tricky:**
> - `void f(int); void f(int = 5);` — not valid overloads; default args don't change the signature.
> - `f(int)` vs `f(long)` called with `f('a')` → `int` (char → int promotion beats conversion).
> - `f(float)` and `f(double)` called with `f(3.5)` → `double` (literals are `double`). `f(int)` and `f(double)` with `f(3.5f)` → `double` (float→double promotion).
> - **Java:** overloading uses the **static (declared) type** of the argument. `Object o = "hi"; f(o);` calls `f(Object)`, not `f(String)`.
> - **Java:** `f(null)` with `f(Object)` and `f(String)` → `f(String)` (most specific). With `f(String)` and `f(Integer)` → **ambiguous, compile error**.

## Operator overloading (C++)
- Gives user types natural syntax: `a + b`, `cout << a`, `a == b`.
- **Cannot overload:** `::`, `.`, `.*`, `?:`, `sizeof`, `typeid`.
- Cannot create new operators, change precedence/associativity, or change arity.
- `<<` / `>>` for streams must be **non-member** (often `friend`) because the left operand is the stream.
- `=`, `[]`, `()`, `->` must be **member** functions.
- Prefix `++a` → `operator++()`; postfix `a++` → `operator++(int)` (dummy int).
- **Java:** no user-defined operator overloading (only built-in `+` for String).

## Runtime polymorphism — virtual functions
- Mark base function `virtual`; override in derived; call through a **base pointer or reference**.
- Use `override` in the derived class so the compiler checks the signature matches.
- `final` on a function stops further overriding; `final` on a class stops inheritance.

### How it works: vtable & vptr
- Each class with virtual functions has one **vtable** (array of function pointers), built by the compiler.
- Each object of such a class has a hidden **vptr** pointing to its class's vtable, set by the constructor.
- `p->f()` compiles to: follow `p`'s vptr → index into vtable → call.
- One vtable **per class**, one vptr **per object** (per base subobject in multiple inheritance).

> [!WARNING]
> **Tricky — things that are statically bound even in a virtual call:**
> - **Default arguments** come from the **static type**. `B* p = new D; p->f();` with `B::f(int x=1)` and `D::f(int x=2)` runs **D's body with x = 1**.
> - Calls inside constructors/destructors run the version of the class currently being built (see 03).
> - Calls on an object (not a pointer/reference): `D d; d.f();` resolved at compile time.

### Overriding rules
- Same name, same parameters, same cv-qualifiers.
- Return type must be the same, or **covariant** (base returns `B*`, derived may return `D*`).
- Access level may differ in C++ — access is checked against the **static type** (a public base virtual overridden as private in derived can still be called via a base pointer).
- **Java:** override can't **reduce** visibility, can't throw broader checked exceptions.

## Name hiding

> [!WARNING]
> **Tricky:**
> - If a derived class declares a function with the **same name** as a base function (any signature), it **hides all base overloads** of that name.
> ```cpp
> struct B { void f(int); void f(string); };
> struct D : B { void f(double); };
> D d; d.f("hi");   // ERROR: B::f(string) is hidden
> ```
> Fix: `using B::f;` inside `D`.
> - A **non-virtual** function redefined in derived with the same signature is hiding, not overriding: `B* p = new D; p->g();` calls `B::g`.
> - **Java:** static methods are hidden, not overridden → chosen by reference type. **Fields** are never polymorphic → `p.field` uses the reference type.

## Object slicing
Assigning or passing a derived object **by value** to a base type copies only the base part. The derived data is "sliced off" and the vptr is the base's, so virtual calls go to the base version.
Avoid by using pointers or references. (Not an issue in Java — only references exist.)

## Pure virtual functions
`virtual void draw() = 0;` → makes the class **abstract** (see 07).

## Quick interview answers
- *Overloading vs overriding?* Overloading: same scope, different params, compile time. Overriding: base/derived, same signature, runtime.
- *Can static functions be virtual?* No — virtual needs an object (`this`/vptr).
- *Can a virtual function be inline?* Yes; it's actually inlined only when the call is resolved statically.
- *Can a private virtual function be overridden?* Yes (C++); this is the basis of the Non-Virtual Interface (NVI) idiom. In Java, private methods are not overridden.
- *Why is a virtual destructor needed?* So `delete basePtr` runs the derived destructor first.
- *Is the vtable per object?* No — per class. The vptr is per object.
