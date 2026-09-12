# 03 — Creational Patterns ★★★

Creational patterns deal with **how objects are created**, decoupling client code from the concrete classes it instantiates.

**GoF creational patterns (5):** Factory Method, Abstract Factory, Builder, Singleton, Prototype. *Simple Factory* is a common idiom, not one of the 23 GoF patterns.

---

## 1. Factory ★★★

### Simple Factory (idiom)
One class with a method that returns different concrete objects based on input.
`VehicleFactory.create("car")` → returns a `Car`, `Bike`, or `Truck` behind a `Vehicle` interface.
- **Pros:** client no longer does `new Car()`; creation logic in one place.
- **Cons:** the factory's `switch` must be edited for every new type → violates OCP.

### Factory Method (GoF)
**Define an interface for creating an object, but let subclasses decide which class to instantiate.**
- **Participants:** `Product` (interface), `ConcreteProduct`, `Creator` (declares the abstract factory method, often has business logic that uses the product), `ConcreteCreator` (overrides the factory method).
- **Example:** `Logistics.planDelivery()` calls `createTransport()`; `RoadLogistics` returns `Truck`, `SeaLogistics` returns `Ship`.
- **Real-world:** `Collection.iterator()` (each collection returns its own iterator), `NumberFormat.getInstance()`-style static factories.
- **Use when:** the class can't anticipate which objects it must create, or you want subclasses to specify them.

> [!WARNING]
> **Tricky:** Simple Factory uses **one class with a conditional**; Factory Method uses **inheritance** — one subclass per product, no conditional. Interviewers often ask for this difference.

## 2. Abstract Factory ★★
**Provide an interface for creating families of related objects without specifying their concrete classes.**
- **Example:** `GUIFactory` with `createButton()` and `createCheckbox()`. `WindowsFactory` returns Windows-styled widgets, `MacFactory` returns Mac-styled ones. The app picks one factory at start-up and all widgets stay consistent.
- Other examples: database families (connection + command + reader for MySQL vs Postgres), car-part families by manufacturer.
- **Pros:** guarantees products from one family are used together; swap entire family by swapping one object.
- **Cons:** adding a **new product type** (e.g. `createSlider()`) means changing the interface and **every** factory. Adding a **new family** is easy.
- Often each factory method inside an Abstract Factory is itself a Factory Method, and the concrete factory is often a Singleton.

## 3. Builder ★★
**Separate the construction of a complex object from its representation, so the same construction process can build different representations.** Construct step by step.
- **Problem it solves:** the **telescoping constructor** — `Pizza(size)`, `Pizza(size, cheese)`, `Pizza(size, cheese, pepperoni, ...)`: unreadable calls, many overloads, easy to swap arguments of the same type.
- **Shape:** `new Pizza.Builder("large").cheese().olives().build();` (fluent interface, methods return the builder).
- **Participants (GoF):** Builder interface, ConcreteBuilder, **Director** (optional; knows the sequence of steps, e.g. `buildMargherita()`), Product.
- **Benefits:** readable optional parameters, validation in `build()`, makes **immutable** objects easy (all fields final, set only via builder).
- **Real-world:** `StringBuilder`, Lombok `@Builder`, HTTP client request builders, `Stream.Builder`.

## 4. Singleton ★★★
**Ensure a class has only one instance and provide a global access point to it.**
- **Use cases:** configuration manager, logger, connection pool, cache, thread pool, hardware access (printer spooler).
- **Recipe:** private constructor + private static instance + public static `getInstance()`.

### Implementation variants (know the trade-offs, not necessarily the code)

| Variant | Lazy? | Thread-safe? | Notes |
|---|---|---|---|
| Eager (`static final INSTANCE = new X()`) | ❌ | ✅ | Created at class load even if never used |
| Lazy, unsynchronised | ✅ | ❌ | Two threads can both see `null` and create two instances |
| `synchronized getInstance()` | ✅ | ✅ | Every call pays the lock cost |
| **Double-checked locking** | ✅ | ✅ *with `volatile`* | Lock only on first creation |
| **Bill Pugh / static holder class** | ✅ | ✅ | Uses JVM class loading guarantees; no locks. Preferred in Java |
| **Enum singleton** | ❌ | ✅ | Safe against reflection and serialization. Recommended by *Effective Java* |
| C++ **Meyers singleton** (`static X inst;` inside `getInstance()`) | ✅ | ✅ since C++11 | Function-local static init is thread-safe |

Double-checked locking (Java) — the shape interviewers expect:
```java
private static volatile Singleton instance;
public static Singleton getInstance() {
    if (instance == null) {                  // 1st check, no lock
        synchronized (Singleton.class) {
            if (instance == null)            // 2nd check, with lock
                instance = new Singleton();
        }
    }
    return instance;
}
```

> [!WARNING]
> **Tricky (most-asked Singleton follow-ups):**
> - **Why `volatile` in DCL?** `instance = new Singleton()` is three steps: allocate, construct, assign. The JIT/CPU may **reorder** to allocate → assign → construct, so another thread can see a non-null but **half-constructed** object. `volatile` forbids that reordering and ensures visibility.
> - **Why the second null check?** Two threads can both pass the first check; the second one, after getting the lock, must not create again.
> - **Ways to break a singleton (Java):**
>   - **Reflection** — `setAccessible(true)` on the private constructor. Guard: throw from the constructor if `instance != null`, or use an enum.
>   - **Serialization** — deserialising creates a new object. Guard: implement `readResolve()` returning `instance`, or use an enum.
>   - **Cloning** — guard: override `clone()` to throw or return `instance`.
>   - **Multiple class loaders** — each loader can load its own copy.
> - **Criticisms:** global state, hidden dependencies, hard to unit-test/mock, tight coupling, arguably violates SRP. Prefer passing a single instance via DI when possible.
> - Singleton ≠ static class: a singleton can implement interfaces, be passed around, be lazily created, and be subclassed/mocked; a static utility class can't.

## 5. Prototype ★
**Create new objects by copying (cloning) an existing instance** instead of building from scratch.
- **Use when:** creation is expensive (DB/network loading, heavy computation), or you need many objects differing only slightly from a preset configuration, or the concrete class isn't known to the client.
- **Participants:** `Prototype` interface with `clone()`, concrete prototypes, optional **prototype registry** (map of named, pre-configured prototypes to clone from).
- **Example:** game spawning many enemies from a configured template; document templates; shape editors copy-paste.

> [!WARNING]
> **Tricky:** shallow vs deep cloning is the whole difficulty. Java's `Object.clone()` is **shallow**; mutable fields (lists, nested objects) end up shared. `Cloneable` is a marker interface with a famously awkward design — many prefer copy constructors or copy factories.

---

## Summary

| Pattern | One-liner | Tell-tale sign in a question |
|---|---|---|
| Factory Method | Subclass decides which object to create | "Create X based on type", `switch` on type for `new` |
| Abstract Factory | Factory of related factories / families | "Themes", "platforms", "product families must match" |
| Builder | Step-by-step construction | Many optional params, immutable object |
| Singleton | Exactly one instance | "Only one config/logger/pool" |
| Prototype | Clone instead of new | "Expensive to create", "copy a template" |
