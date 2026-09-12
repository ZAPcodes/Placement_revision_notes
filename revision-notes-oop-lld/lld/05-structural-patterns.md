# 05 — Structural Patterns ★★

Structural patterns are about **how classes and objects are composed** into larger structures while keeping them flexible.

**GoF structural patterns (7):** Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy.

---

## 1. Adapter ★★
**Convert the interface of a class into another interface clients expect.** Lets incompatible classes work together.
- **Analogy:** a travel plug adapter.
- **Participants:** `Target` (interface client uses), `Adaptee` (existing incompatible class), `Adapter` (implements Target, translates calls to Adaptee).
- **Examples:** integrating a third-party payment gateway whose API differs from your `PaymentProcessor` interface; wrapping a legacy XML service to return JSON; Java `InputStreamReader` (byte stream → character reader), `Arrays.asList()` (array → List).
- **Object adapter:** holds the adaptee (composition) — preferred, works in any language.
- **Class adapter:** inherits from both target and adaptee — needs multiple inheritance (C++), not possible with classes in Java.

## 2. Composite ★★
**Compose objects into tree structures to represent part-whole hierarchies, and let clients treat individual objects and compositions uniformly.**
- **Participants:** `Component` interface (e.g. `getSize()`), `Leaf` (no children), `Composite` (holds children, implements operations by delegating to them).
- **Examples:** file system (`File` and `Folder` both have `getSize()`; folder sums its children), org chart (employee / manager), UI component trees, menus and submenus, arithmetic expression trees.
- **Key benefit:** client calls `root.getSize()` without caring whether it's a file or a folder — recursion is hidden.

> [!WARNING]
> **Tricky:** where do `add()`/`remove()` go? In `Component` → uniform treatment but a leaf must reject `add()` (an LSP smell). Only in `Composite` → type-safe but the client must know which it has. Name this trade-off if asked.

## 3. Facade ★★
**Provide a single, simplified interface to a complex subsystem.**
- **Examples:** `HomeTheaterFacade.watchMovie()` turns on projector, amplifier, lights, player; `Computer.start()` hides CPU/memory/disk boot steps; an `OrderFacade.placeOrder()` that coordinates inventory, payment, shipping, notification services; SLF4J over logging libraries.
- **Pros:** decouples clients from subsystem details; easier to use.

> [!WARNING]
> **Tricky:** a facade doesn't *hide* the subsystem — advanced clients can still use subsystem classes directly. It's a convenience layer, not an access-control layer (that would be a Proxy).

## 4. Decorator ★★★
**Attach additional responsibilities to an object dynamically by wrapping it.** An alternative to subclassing for extending behaviour.
- **Participants:** `Component` interface, `ConcreteComponent`, abstract `Decorator` (implements Component **and** holds a Component), concrete decorators that add behaviour before/after delegating.
- **Examples:** coffee with add-ons (`Milk(Sugar(Espresso))`, each adds to `cost()`), pizza toppings, Java I/O (`new BufferedReader(new InputStreamReader(new FileInputStream(f)))`), adding logging/compression/encryption layers to a data stream, notification channels stacked on a base notifier.
- **Why:** avoids a **class explosion** — with 5 add-ons you'd need up to 2⁵ subclasses for every combination; with decorators you need 5 classes.

> [!WARNING]
> **Tricky:**
> - A decorator **is-a** Component and **has-a** Component — both at once. That's how wrapping can be nested.
> - **Order of wrapping matters** (encrypt-then-compress ≠ compress-then-encrypt).
> - Decorated object's identity changes: `decorated != original`, so identity checks break.

## 5. Bridge ★
**Decouple an abstraction from its implementation so the two can vary independently.**
- **Problem:** two independent dimensions of variation. `Shape` × `Color` → `RedCircle`, `BlueCircle`, `RedSquare`, `BlueSquare`... that's **m × n** classes.
- **Fix:** `Shape` holds a reference to a `Color` (the "bridge"). Now **m + n** classes.
- **Examples:** `RemoteControl` (basic, advanced) × `Device` (TV, radio); JDBC API (abstraction) × database drivers (implementation); message types × sending channels (email, SMS).
- Designed **up front**, unlike Adapter which is applied **after the fact**.

## 6. Proxy ★★
**Provide a surrogate or placeholder that controls access to another object.** Same interface as the real object.

| Proxy type | Purpose | Example |
|---|---|---|
| **Virtual** | Lazy creation of an expensive object | Load a high-res image only when displayed; Hibernate lazy loading |
| **Protection** | Access control / permissions | Only admins can call `deleteUser()` |
| **Remote** | Represent an object in another address space | RMI stubs, gRPC client stubs |
| **Caching** | Return cached results | Cache in front of a slow service |
| **Smart / logging** | Extra housekeeping around calls | Reference counting (`shared_ptr` is a smart proxy), logging, metrics; Spring AOP proxies for `@Transactional` |

## 7. Flyweight ★
**Share common state among a large number of fine-grained objects to save memory.**
- **Intrinsic state:** shared, immutable, stored in the flyweight (e.g. a character's font and glyph; a tree type's texture and mesh).
- **Extrinsic state:** unique per use, passed in by the client (e.g. the character's position; the tree's coordinates).
- A **flyweight factory** keeps a pool and returns existing instances.
- **Examples:** characters in a text editor, trees/bullets/particles in a game, Java **String pool**, `Integer.valueOf()` caching −128..127.

> [!WARNING]
> **Tricky:** flyweights **must be immutable** — they're shared, so modifying one changes it everywhere. The classic trap `Integer a=127,b=127; a==b` → `true`, but with 128 → `false` comes from this cache.

---

## Summary

| Pattern | One-liner | Tell-tale sign |
|---|---|---|
| Adapter | Make incompatible interface fit | "Integrate third-party / legacy API" |
| Composite | Tree; treat leaf and group alike | "Folders and files", "hierarchy" |
| Facade | Simple front for complex subsystem | "One call does many steps" |
| Decorator | Add behaviour by wrapping | "Add-ons", "toppings", "layers" |
| Bridge | Split two dimensions of variation | "m × n class explosion" |
| Proxy | Controlled stand-in | "Lazy load", "access control", "cache" |
| Flyweight | Share state among many objects | "Millions of similar objects", memory |
