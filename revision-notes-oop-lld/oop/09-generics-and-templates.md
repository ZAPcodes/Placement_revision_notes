# 09 — Generics & Templates ★

**Goal:** write type-safe code once and reuse it for many types (e.g. one `Stack<T>` instead of `IntStack`, `StringStack`...). This is **compile-time (parametric) polymorphism**.

## C++ templates
- **Function template:** `template <typename T> T maxOf(T a, T b);`
- **Class template:** `template <typename T> class Stack { ... };`
- **Non-type parameters:** `template <typename T, int N> class Array;` (like `std::array<int, 5>`)
- **Instantiation:** the compiler generates a **separate copy of the code for each type** used (`Stack<int>`, `Stack<string>`).
- **Specialisation:** full specialisation (`template<> class Stack<bool>`) and partial specialisation (class templates only, not functions).
- Templates are usually defined **entirely in header files**, since the compiler needs the full definition at each point of instantiation.
- Pros: zero runtime cost, works with primitives. Cons: **code bloat**, long compile times, cryptic errors (improved by C++20 concepts).

## Java generics
- `class Box<T> { T value; }`, `<T> void print(T x)`.
- **Type erasure:** type parameters exist only at compile time. The compiler checks types, inserts casts, then erases `T` to `Object` (or to its bound). At runtime `List<String>` and `List<Integer>` are the **same class**.

> [!WARNING]
> **Tricky — consequences of type erasure:**
> - Can't use primitives: `List<int>` ❌ → use `List<Integer>` (autoboxing).
> - Can't do `new T()` or `new T[10]`.
> - Can't check `obj instanceof List<String>` (only `List<?>`).
> - Can't overload `f(List<String>)` and `f(List<Integer>)` — same erasure → compile error.
> - A `static` field in `Box<T>` is **shared** across `Box<String>` and `Box<Integer>` (one class at runtime).

## Bounded types & wildcards (Java)
- **Bounded type parameter:** `<T extends Number>` → T must be `Number` or a subclass; you can call `Number` methods on it. Multiple bounds: `<T extends Number & Comparable<T>>` (class first).
- `?` — unknown type: `List<?>` (read as `Object`, can't add anything but `null`).
- `? extends T` — **upper bounded**: some subtype of T. You can **read** T out, but can't add (except `null`).
- `? super T` — **lower bounded**: some supertype of T. You can **add** T in, but reads give only `Object`.
- **PECS:** *Producer Extends, Consumer Super.* e.g. `Collections.copy(List<? super T> dest, List<? extends T> src)`.

> [!WARNING]
> **Tricky — invariance:**
> - `List<Integer>` is **not** a subtype of `List<Number>`, even though `Integer` is a subtype of `Number`. Otherwise you could add a `Double` into a list of Integers.
> - Arrays **are** covariant: `Number[] arr = new Integer[3]; arr[0] = 1.5;` compiles but throws `ArrayStoreException` at runtime. Generics move that error to compile time.
> - `List<? extends Number>` **does** accept a `List<Integer>`.

## C++ templates vs Java generics

| | C++ templates | Java generics |
|---|---|---|
| Implementation | Code generated per type | Type erasure, one shared class |
| Primitives | ✅ | ❌ (boxing) |
| Runtime type info | Each instantiation is a distinct type | Erased |
| Non-type params | ✅ (`int N`) | ❌ |
| Specialisation | ✅ | ❌ |
| Constraints | C++20 concepts / SFINAE | `extends` / `super` bounds |
| Cost | Code bloat, zero runtime overhead | Casts at runtime, boxing overhead |
