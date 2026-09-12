# 10 — Tricky Output Questions ★★★

Predict the output before opening the answer. Assume `#include <iostream>` and `using namespace std;` for C++.

---

### Q1. Virtual call inside a constructor (C++)
```cpp
struct B { B() { f(); } virtual void f() { cout << "B"; } };
struct D : B { void f() override { cout << "D"; } };
int main() { D d; }
```
<details><summary>Answer</summary>

**`B`**. While `B`'s constructor runs, the object is still a `B` (vptr points to B's vtable), so the base version is called.
</details>

### Q2. Same thing in Java
```java
class P { P() { show(); } void show() { System.out.print("P"); } }
class C extends P { int x = 5; void show() { System.out.print(x); } }
// new C();
```
<details><summary>Answer</summary>

**`0`**. Java dispatches to `C.show()` even from `P`'s constructor, but `C`'s field initialiser (`x = 5`) hasn't run yet, so `x` is still the default `0`.
</details>

### Q3. Default arguments + virtual
```cpp
struct B { virtual void f(int x = 1) { cout << "B" << x; } };
struct D : B { void f(int x = 2) override { cout << "D" << x; } };
int main() { B* p = new D; p->f(); }
```
<details><summary>Answer</summary>

**`D1`**. The function body is chosen at runtime (D), but default arguments are taken from the static type (B) at compile time.
</details>

### Q4. Missing virtual destructor
```cpp
struct B { ~B() { cout << "~B"; } };
struct D : B { ~D() { cout << "~D"; } };
int main() { B* p = new D; delete p; }
```
<details><summary>Answer</summary>

Typically **`~B`** only — `~D` never runs (formally undefined behaviour). Making `~B` virtual prints `~D~B`.
</details>

### Q5. Object slicing
```cpp
struct B { virtual void f() { cout << "B"; } };
struct D : B { void f() override { cout << "D"; } };
void show(B b) { b.f(); }
int main() { D d; show(d); }
```
<details><summary>Answer</summary>

**`B`**. `d` is copied into a `B` by value; the derived part is sliced off. Passing `B&` would print `D`.
</details>

### Q6. Name hiding
```cpp
struct B { void f(int) { cout << "int"; } };
struct D : B { void f(double) { cout << "double"; } };
int main() { D d; d.f(5); }
```
<details><summary>Answer</summary>

**`double`**. `D::f` hides `B::f`; overload resolution only sees `D::f(double)`, and 5 converts to 5.0. Add `using B::f;` in D to get `int`.
</details>

### Q7. Construction and destruction order
```cpp
struct M { M() { cout << "M"; } ~M() { cout << "~M"; } };
struct B { B() { cout << "B"; } ~B() { cout << "~B"; } };
struct D : B { M m; D() { cout << "D"; } ~D() { cout << "~D"; } };
int main() { D d; }
```
<details><summary>Answer</summary>

**`BMD~D~M~B`**. Base → members → body; destruction is the exact reverse.
</details>

### Q8. Initializer list order
```cpp
struct A {
    int x, y;
    A(int v) : y(v), x(y + 1) {}
};
int main() { A a(5); cout << a.x << " " << a.y; }
```
<details><summary>Answer</summary>

`x` is **garbage**, `y` is `5`. Members initialise in declaration order (`x` first), so `x` reads `y` before `y` is set.
</details>

### Q9. Copy constructor or assignment?
```cpp
struct A {
    A() {}
    A(const A&) { cout << "C"; }
    A& operator=(const A&) { cout << "="; return *this; }
};
int main() { A a; A b = a; A c; c = a; }
```
<details><summary>Answer</summary>

**`C=`**. `A b = a;` is copy construction; `c = a;` on an existing object is copy assignment.
</details>

### Q10. Static counter
```cpp
struct A { static int c; A() { c++; } };
int A::c = 0;
int main() { A a, b[3], *p; cout << A::c; }
```
<details><summary>Answer</summary>

**`4`**. `a` (1) + `b[3]` (3); `p` is a pointer, no object is constructed.
</details>

### Q11. sizeof (64-bit)
```cpp
struct E {};
struct S { char c; int i; };
struct V { virtual void f() {} };
cout << sizeof(E) << sizeof(S) << sizeof(V);
```
<details><summary>Answer</summary>

**`188`**. Empty class = 1; `char` + 3 padding + `int` = 8; vptr = 8.
</details>

### Q12. Diamond without virtual inheritance
```cpp
struct A { void hi() { cout << "A"; } };
struct B : A {};
struct C : A {};
struct D : B, C {};
int main() { D d; d.hi(); }
```
<details><summary>Answer</summary>

**Compile error** — `hi` is ambiguous (two `A` subobjects). With `B : virtual A` and `C : virtual A` it prints `A`.
</details>

### Q13. Private override called via base pointer
```cpp
struct B { virtual void f() { cout << "B"; } };
struct D : B { private: void f() override { cout << "D"; } };
int main() { B* p = new D; p->f(); }
```
<details><summary>Answer</summary>

**`D`**. Access is checked against the static type `B` (where `f` is public); dispatch still goes to `D::f`. Calling `D d; d.f();` would be a compile error.
</details>

### Q14. Most vexing parse
```cpp
struct A { A() { cout << "ctor"; } };
int main() { A a(); }
```
<details><summary>Answer</summary>

**Prints nothing.** `A a();` declares a function named `a` returning `A`.
</details>

### Q15. Java static method hiding
```java
class P { static void f() { System.out.print("P"); } }
class C extends P { static void f() { System.out.print("C"); } }
// P p = new C(); p.f();
```
<details><summary>Answer</summary>

**`P`**. Static methods are hidden, not overridden; the reference type decides.
</details>

### Q16. Java fields aren't polymorphic
```java
class P { String n = "P"; String get() { return n; } }
class C extends P { String n = "C"; String get() { return n; } }
// P p = new C(); System.out.print(p.n + p.get());
```
<details><summary>Answer</summary>

**`PC`**. `p.n` uses the reference type (P's field); `p.get()` is dynamically dispatched to C.
</details>

### Q17. Java overloading with static types and null
```java
static void f(Object o) { System.out.print("Obj"); }
static void f(String s) { System.out.print("Str"); }
// Object o = "hi"; f(o); f(null);
```
<details><summary>Answer</summary>

**`ObjStr`**. Overloads are picked from the declared type (`Object`). For `null`, the most specific applicable overload (`String`) wins. Adding `f(Integer)` would make `f(null)` ambiguous.
</details>

### Q18. Java Integer cache
```java
Integer a = 127, b = 127, c = 128, d = 128;
System.out.print((a == b) + " " + (c == d));
```
<details><summary>Answer</summary>

**`true false`**. `Integer.valueOf` caches −128..127 (Flyweight), so `a` and `b` are the same object; 128 creates new objects. Always compare wrappers with `.equals()`.
</details>

### Q19. Pure virtual destructor without a body
```cpp
struct B { virtual ~B() = 0; };
struct D : B {};
int main() { D d; }
```
<details><summary>Answer</summary>

**Linker error** — `~D` calls `~B`, which has no definition. Add `B::~B() {}`.
</details>

### Q20. Constructor order with multiple inheritance
```cpp
struct A { A() { cout << "A"; } };
struct B { B() { cout << "B"; } };
struct C : B, A { C() : A(), B() { cout << "C"; } };
int main() { C c; }
```
<details><summary>Answer</summary>

**`BAC`**. Bases are constructed in the order they appear in the class head (`B, A`), not the initializer list.
</details>
