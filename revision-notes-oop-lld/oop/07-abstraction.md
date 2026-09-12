# 07 — Abstraction ★★★

**Abstraction:** exposing only the essential behaviour (the *what*) and hiding implementation details (the *how*). The user of `List.add()` doesn't need to know whether it's an array or a linked list underneath.

**Achieved by:** abstract classes, interfaces, and more generally any well-designed public API (header files, access control).

## Abstract class (C++)
- A class with **at least one pure virtual function**: `virtual void area() = 0;`
- **Cannot be instantiated**, but you can have pointers and references to it.
- Can have constructors, data members, and normal (implemented) functions.
- A derived class must override **all** pure virtual functions, otherwise it is also abstract.

> [!WARNING]
> **Tricky (C++):**
> - A pure virtual function **can have a body**, callable explicitly as `Base::f()`.
> - A **pure virtual destructor must have a body** (`Base::~Base() {}`), because every derived destructor calls it — otherwise you get a **linker error**. It's used to make a class abstract when it has no other natural pure virtual function.
> - An abstract class can have a constructor — it runs when a derived object is created.
> - Calling a pure virtual function from a constructor (with no body) → undefined behaviour / "pure virtual call" crash.

## Interface
- **C++:** no keyword; by convention a class with only pure virtual functions and a virtual destructor.
- **Java:** `interface` keyword. A class can implement many interfaces.

## Java: abstract class vs interface

| | Abstract class | Interface |
|---|---|---|
| Methods | Abstract + concrete | Abstract + `default` + `static` (Java 8) + `private` (Java 9) |
| Fields | Any kind (instance, static, final or not) | Only `public static final` constants |
| Constructors | Yes | No |
| Inheritance | A class extends **one** | A class implements **many** |
| Method access | Any modifier | Implicitly `public` (except private helpers) |
| State | Can hold instance state | No instance state |
| Relationship | **is-a** (family of related classes) | **can-do** (capability across unrelated classes) |

**When to use which:**
- Abstract class: closely related classes that share **state or code** (`Vehicle` with common `fuelLevel` and `refuel()`).
- Interface: a **capability** that unrelated classes can have (`Comparable`, `Runnable`, `Serializable`), or when multiple inheritance of type is needed.

> [!WARNING]
> **Tricky (Java):**
> - An abstract class **can have zero abstract methods** (declared `abstract` just to prevent instantiation).
> - An abstract method **cannot** be `private`, `static`, or `final` (all prevent overriding).
> - An abstract class can have a `main` method and can be run.
> - A class with even one abstract method **must** be declared abstract.
> - Interface fields are implicitly `public static final` — `int x = 5;` in an interface is a constant.
> - **Functional interface** = exactly one abstract method (default/static methods don't count); usable with lambdas.
> - **Marker interface** = no methods (`Serializable`, `Cloneable`) — signals a capability to the JVM/framework.

## Quick interview answers
- *Can we create an object of an abstract class?* No. But an anonymous subclass (Java) or derived-class object is fine, and a base pointer/reference can refer to it.
- *Abstraction vs encapsulation?* See the table in 06.
- *Can an interface extend another interface?* Yes, and it can extend **multiple** interfaces.
- *Can an interface implement a class?* No.
