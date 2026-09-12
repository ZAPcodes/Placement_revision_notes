# 11 — Practice MCQs + Cheatsheet (read before OA / interview)

## Part A — Practice MCQs

Answers are hidden under each question.

**1.** Which of these cannot be overloaded in C++?
(a) `[]` (b) `()` (c) `::` (d) `->`
<details><summary>Answer</summary>(c) — also `.`, `.*`, `?:`, `sizeof`, `typeid`.</details>

**2.** `sizeof` of an empty class in C++ is:
(a) 0 (b) 1 (c) 4 (d) Compiler error
<details><summary>Answer</summary>(b) — so distinct objects have distinct addresses.</details>

**3.** Which function **cannot** be virtual in C++?
(a) Destructor (b) Constructor (c) Private member function (d) Inline function
<details><summary>Answer</summary>(b). Destructors, private, and inline functions can all be virtual.</details>

**4.** A class containing at least one pure virtual function is:
(a) Virtual class (b) Abstract class (c) Final class (d) Friend class
<details><summary>Answer</summary>(b)</details>

**5.** Runtime polymorphism in C++ is achieved through:
(a) Function overloading (b) Operator overloading (c) Virtual functions (d) Templates
<details><summary>Answer</summary>(c). The others are compile-time.</details>

**6.** Which is true about `friend` functions?
(a) They are inherited (b) Friendship is mutual (c) They have a `this` pointer (d) They can access private members of the class that declares them
<details><summary>Answer</summary>(d). Not inherited, not mutual, not transitive, no `this`.</details>

**7.** Why is a copy constructor's parameter passed by reference?
(a) Efficiency only (b) To avoid infinite recursion (c) To allow modification (d) Language convention only
<details><summary>Answer</summary>(b). By-value would need a copy, calling the copy constructor again.</details>

**8.** In `class D : public B1, public B2`, base constructors run in which order?
(a) B2, B1 (b) B1, B2 (c) Order of the initializer list (d) Unspecified
<details><summary>Answer</summary>(b). Order in the class head.</details>

**9.** Deleting a derived object through a base pointer without a virtual destructor:
(a) Calls both destructors (b) Calls only the derived destructor (c) Undefined behaviour, typically only base destructor runs (d) Compile error
<details><summary>Answer</summary>(c)</details>

**10.** The diamond problem in C++ is solved using:
(a) Virtual functions (b) Virtual inheritance (c) Abstract classes (d) Friend classes
<details><summary>Answer</summary>(b)</details>

**11.** Which relationship has the part's lifetime tied to the whole?
(a) Association (b) Aggregation (c) Composition (d) Dependency
<details><summary>Answer</summary>(c)</details>

**12.** Which is true about static member functions in C++?
(a) Can be virtual (b) Have a `this` pointer (c) Can access only static members directly (d) Can be `const`
<details><summary>Answer</summary>(c)</details>

**13.** In Java, which is **not** allowed in an interface (Java 9+)?
(a) Default methods (b) Static methods (c) Private methods (d) Instance fields that aren't constants
<details><summary>Answer</summary>(d). Interface fields are always `public static final`.</details>

**14.** Java `protected` members are accessible to:
(a) Subclasses only (b) Same package only (c) Same package and subclasses in any package (d) Everyone
<details><summary>Answer</summary>(c)</details>

**15.** Which is **not** a valid basis for overloading?
(a) Number of parameters (b) Types of parameters (c) Return type only (d) Order of parameter types
<details><summary>Answer</summary>(c)</details>

**16.** Which statement about abstract classes in Java is **false**?
(a) Can have constructors (b) Can have zero abstract methods (c) Can be instantiated with `new` (d) Can have a `main` method
<details><summary>Answer</summary>(c)</details>

**17.** An abstract method in Java can be:
(a) `private` (b) `static` (c) `final` (d) `protected`
<details><summary>Answer</summary>(d). The other three block overriding.</details>

**18.** Hiding internal state and allowing access only through methods is:
(a) Abstraction (b) Encapsulation (c) Polymorphism (d) Inheritance
<details><summary>Answer</summary>(b)</details>

**19.** What does `A a();` do in C++?
(a) Creates object with default constructor (b) Declares a function (c) Compile error (d) Creates a null object
<details><summary>Answer</summary>(b)</details>

**20.** Which must be initialised in a constructor's member initializer list?
(a) `static` members (b) `const` and reference members (c) Pointer members (d) Public members
<details><summary>Answer</summary>(b) — also members/bases with no default constructor.</details>

**21.** In Java, `List<Integer>` is a subtype of:
(a) `List<Number>` (b) `List<Object>` (c) `List<? extends Number>` (d) None of these
<details><summary>Answer</summary>(c). Generics are invariant; wildcards add flexibility.</details>

**22.** Java generics are implemented with:
(a) Code generation per type (b) Type erasure (c) Reflection (d) Macros
<details><summary>Answer</summary>(b)</details>

**23.** The vtable exists:
(a) Per object (b) Per class with virtual functions (c) Per function (d) Per program
<details><summary>Answer</summary>(b). vptr is per object.</details>

**24.** Default arguments in a virtual function call through a base pointer come from:
(a) Derived class (b) Base class (static type) (c) Whichever is larger (d) Compile error
<details><summary>Answer</summary>(b)</details>

**25.** Which OOP feature does `Square extends Rectangle` commonly violate?
(a) Encapsulation (b) Liskov Substitution Principle (c) Abstraction (d) DRY
<details><summary>Answer</summary>(b). Setting width independently breaks Square's invariant.</details>

**26.** A private destructor in C++ means objects:
(a) Cannot be created (b) Can only be created on the heap (practically) (c) Can only be created on the stack (d) Are singletons
<details><summary>Answer</summary>(b). Stack objects need an accessible destructor at scope end.</details>

**27.** Which Java keyword prevents a class from being inherited?
(a) `static` (b) `abstract` (c) `final` (d) `private`
<details><summary>Answer</summary>(c)</details>

**28.** Member-wise copy of an object with a pointer member leads to:
(a) Deep copy (b) Shallow copy — both objects share the pointed memory (c) Compile error (d) Memory is duplicated automatically
<details><summary>Answer</summary>(b)</details>

---

## Part B — Cheatsheet

### Pillars in one line each
- **Encapsulation:** bundle data + methods, hide data behind access control.
- **Abstraction:** expose *what*, hide *how* (abstract classes / interfaces).
- **Inheritance:** is-a reuse; derived gets base members.
- **Polymorphism:** one interface, many forms (overloading = compile time, overriding = runtime).

### Relationships
| Dependency | Association | Aggregation ◇ | Composition ◆ | Inheritance ▷ |
|---|---|---|---|---|
| uses temporarily | knows | weak has-a, part survives | strong has-a, part dies | is-a |

### Overloading vs overriding
| | Overloading | Overriding |
|---|---|---|
| Where | Same scope | Base ↔ derived |
| Signature | Must differ | Must match (covariant return OK) |
| Resolved | Compile time | Runtime (virtual) |
| Return type alone | Not enough | — |

### Must-remember C++ rules
- Construction: virtual bases → bases (class-head order) → members (declaration order) → body. Destruction is reverse.
- Virtual call in constructor/destructor → current class's version.
- Default args are statically bound.
- Derived function with same name **hides** all base overloads (`using Base::f;` to fix).
- Pass polymorphic objects by pointer/reference to avoid **slicing**.
- Base class with virtual functions → give it a **virtual destructor**.
- Pure virtual destructor still needs a body.
- Compiler-generated default constructor disappears once any constructor is declared.
- Rule of 3/5/0. Copy constructor param: `const A&`.
- `A b = a;` → copy ctor; `b = a;` → assignment.
- sizeof: empty = 1, vptr adds 8 (64-bit), padding matters, static members & functions add nothing.
- Can't overload: `::` `.` `.*` `?:` `sizeof` `typeid`.
- Can't be virtual: constructors, static functions.
- Friend: not inherited, not transitive, not mutual.

### Must-remember Java rules
- No multiple class inheritance; interfaces yes. Default-method conflict: class wins → sub-interface wins → else must override.
- All non-static, non-private, non-final methods are virtual.
- Static methods: hidden. Fields: never polymorphic.
- Overloading uses **declared** type; `null` goes to most specific.
- Virtual call from a constructor reaches the subclass (fields still default).
- `protected` includes package access. Overrides can't reduce visibility.
- `this()`/`super()` must be first line.
- Interface fields = `public static final`. Abstract methods can't be private/static/final.
- Generics: type erasure, no primitives, invariant, PECS.
- `Integer` cache −128..127 → use `.equals()`.

### Abstract class vs interface (Java)
| | Abstract class | Interface |
|---|---|---|
| State | Yes | Constants only |
| Constructors | Yes | No |
| Multiple | No | Yes |
| Use for | Related family sharing code | Capability across unrelated classes |

### Encapsulation vs abstraction
Encapsulation = **data hiding** (implementation level). Abstraction = **complexity hiding** (design level).
