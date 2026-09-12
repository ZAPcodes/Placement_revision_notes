# 06 — Pattern Comparisons ★★★

Most "tricky" LLD interview questions are comparisons. Learn the **one-sentence distinction** for each.

## Strategy vs State
Both: a context delegates to an interchangeable object with the same interface. Class diagrams look almost identical.

| | Strategy | State |
|---|---|---|
| Intent | Choose **how** to do a task | Behaviour depends on **what state** the object is in |
| Who switches | The **client** picks/injects the strategy | The **states themselves** trigger transitions |
| Awareness | Strategies don't know each other | States know about other states (next state) |
| Changes | Rarely during the object's life | Frequently, as events happen |
| Example | Payment method, sort algorithm | Vending machine, order lifecycle |

## Factory Method vs Abstract Factory vs Simple Factory

| | Simple Factory | Factory Method | Abstract Factory |
|---|---|---|---|
| Creates | One product, chosen by a parameter | One product | A **family** of related products |
| Mechanism | One class + conditional | **Inheritance**: subclass overrides a method | **Composition**: client holds a factory object |
| Adding a type | Edit the conditional (breaks OCP) | Add a subclass | New family: add a factory. New product kind: edit all factories |

## Factory vs Strategy
- **Factory** is **creational**: decides *which object to create*. Returns an object and is done.
- **Strategy** is **behavioural**: decides *which algorithm to run*. The chosen object is *used* for behaviour.
- They're often combined: a factory creates the right strategy (`PaymentStrategyFactory.get("UPI")`).

## Factory vs Builder
- **Factory:** creates an object in **one call**, hides *which* class.
- **Builder:** creates **one complex** object in **many steps**, hides *how* it's assembled.

## Decorator vs Proxy vs Adapter
All three wrap another object.

| | Adapter | Decorator | Proxy |
|---|---|---|---|
| Interface | **Changes** it (to what client expects) | **Same** interface | **Same** interface |
| Purpose | Compatibility | **Add** behaviour | **Control access** |
| Stacking | Usually one | Often many layers | Usually one |
| Who creates the wrapped object | Client passes it | Client passes it | Proxy often creates/manages it itself (e.g. lazy load) |

## Facade vs Adapter
- **Adapter** makes **one** existing interface match another expected interface.
- **Facade** defines a **new, simpler** interface over **many** classes.

## Facade vs Mediator
- **Facade:** one-way. Clients → facade → subsystem. Subsystem classes don't know the facade exists.
- **Mediator:** two-way. Colleagues talk **to** the mediator, and the mediator talks **back** to them.

## Observer vs Pub-Sub

| | Observer | Publish–Subscribe |
|---|---|---|
| Coupling | Subject holds direct references to observers | Publishers and subscribers don't know each other |
| Middleman | None | **Broker / event bus / topic** (e.g. Kafka) |
| Delivery | Usually synchronous | Usually asynchronous, can be across processes |
| Filtering | Everyone gets every update | Subscribers pick topics |

## Composite vs Decorator
Both use recursive composition. **Composite** aggregates **many** children to represent a tree; **Decorator** wraps **one** component to add behaviour.

## Bridge vs Adapter vs Strategy
- **Bridge:** designed **up front** to separate two hierarchies so both grow independently.
- **Adapter:** applied **after the fact** to make existing incompatible classes work.
- **Strategy:** swaps one **algorithm**; Bridge separates a whole **implementation dimension**.

## Template Method vs Strategy
- **Template Method** varies steps via **inheritance** (compile-time, whole class).
- **Strategy** varies the algorithm via **composition** (runtime, swappable).
- Prefer Strategy if you need to change behaviour at runtime or avoid a subclass per variant.

## Command vs Strategy
Both wrap behaviour in objects. **Command** represents **a request/action** (what to do, often with undo, queueing). **Strategy** represents **a way of doing** the same task.

## Command vs Memento (undo)
Command undoes via **inverse operations**; Memento undoes via **stored snapshots**.

## Chain of Responsibility vs Decorator
Structurally similar (chains of objects). In **CoR**, a handler may **stop** the request and usually one handler handles it. In **Decorator**, every layer adds behaviour and **always** forwards.

## Singleton vs static class
Singleton: an object — can implement interfaces, be lazily created, passed as a parameter, and mocked. Static class: just a namespace of functions — no instance, no polymorphism.

## Prototype vs Builder
Prototype **copies** an existing configured object. Builder **assembles** a new object step by step.
