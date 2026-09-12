# 01 — Foundations ★★

## LLD vs HLD

| | HLD (High-Level Design) | LLD (Low-Level Design) |
|---|---|---|
| Question | *What* components exist and how they talk | *How* each component is built internally |
| Scope | Whole system | A module / service |
| Artefacts | Architecture diagram, services, databases, caches, queues, load balancers | Class diagrams, interfaces, sequence diagrams, method signatures |
| Concerns | Scalability, availability, latency, data partitioning | Extensibility, readability, SOLID, design patterns, thread safety |
| Example | "Design WhatsApp" | "Design the classes for a parking lot" |
| Also called | System design / architecture | Object-oriented design (OOD), machine coding |

## UML class diagram notation

A class box has three compartments: **name**, **attributes**, **methods**.

```
+---------------------------+
|        ParkingLot         |
+---------------------------+
| - floors: List<Floor>     |
| - instance: ParkingLot    |  (underlined = static)
+---------------------------+
| + parkVehicle(v): Ticket  |
| # computeFee(t): double   |
+---------------------------+
```

| Symbol | Visibility |
|---|---|
| `+` | public |
| `-` | private |
| `#` | protected |
| `~` | package (Java default) |

- **Static** members are underlined. **Abstract** class/method names are *italic* (or marked `{abstract}`). Interfaces are marked `<<interface>>`.
- Attribute format: `name: Type`; method format: `name(param: Type): ReturnType`.

### Relationship arrows

| Relationship | Line | End | Read as |
|---|---|---|---|
| Inheritance | Solid | Hollow triangle at parent | "is-a" |
| Realisation | Dashed | Hollow triangle at interface | "implements" |
| Association | Solid | Plain arrow (optional) | "knows" |
| Aggregation | Solid | Hollow diamond at **whole** | "has-a (shared)" |
| Composition | Solid | Filled diamond at **whole** | "owns" |
| Dependency | Dashed | Plain arrow | "uses" |

**Multiplicity** at line ends: `1`, `0..1`, `*` / `0..*`, `1..*`.

> [!WARNING]
> **Tricky:** the diamond goes on the **container** side (the whole), not the part. The triangle points **to the parent/interface**.

## Sequence diagram basics
Shows interactions **over time** for one use case.
- **Lifelines:** vertical dashed lines, one per object/actor.
- **Messages:** horizontal arrows — solid arrow with filled head = synchronous call; open arrowhead = asynchronous; dashed = return.
- **Activation bar:** thin rectangle on a lifeline while the object is busy.
- **Fragments:** `alt` (if/else), `opt` (if), `loop`, `par` (parallel).

Other UML diagrams to know by name: use case, activity, state machine (useful for State pattern), component, deployment. UML diagrams split into **structural** (class, object, component, deployment) and **behavioural** (use case, sequence, activity, state).

## How to approach an LLD interview round
1. **Clarify requirements** (5 min): core features, out-of-scope features, scale of the single machine, concurrency needs. Write them down.
2. **Identify entities** — nouns in the requirements become candidate classes; verbs become methods.
3. **Define relationships** — is-a vs has-a, multiplicity, ownership.
4. **Define interfaces / APIs** — public methods of the main classes.
5. **Apply patterns where they solve a real problem** — e.g. Strategy for pricing, Observer for notifications, State for status transitions, Factory for creation by type. Don't force patterns.
6. **Walk through a use case** end-to-end (sequence).
7. **Handle concurrency** — shared mutable state, which locks, at what granularity.
8. **Discuss extensibility** — "how would you add X?" should mean adding a class, not editing ten.

> [!WARNING]
> **Common mistakes interviewers call out:**
> - Jumping into code before agreeing on requirements.
> - God class (one `Manager` doing everything) — violates SRP.
> - Using inheritance where the behaviour varies at runtime (use Strategy/State).
> - Big `if/else` or `switch` on a type field — a signal for polymorphism or Factory + Strategy.
> - Making everything a Singleton.
> - Ignoring thread safety when the problem clearly has concurrent users (booking, parking, inventory).
