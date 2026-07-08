# Composition vs Inheritance

> **Tag: Theory + design judgment** — "favor composition over inheritance" is the expected stance; be able to *show* why with the Stack example.

## Concept

Two ways to reuse code:
- **Inheritance (is-a):** `class Car extends Vehicle` — Car *is* a Vehicle, inherits everything, compile-time-fixed relationship.
- **Composition (has-a):** `class Car { private Engine engine; }` — Car *has* an Engine, delegates to it, can be swapped at runtime.

```java
// Composition + interface = flexible behavior (this is also the Strategy pattern)
interface Engine { void start(); }
class PetrolEngine implements Engine { public void start() { System.out.println("vroom"); } }
class ElectricEngine implements Engine { public void start() { System.out.println("hum"); } }

class Car {
    private Engine engine;                       // HAS-A
    Car(Engine e) { this.engine = e; }
    void setEngine(Engine e) { this.engine = e; }  // swappable at runtime!
    void start() { engine.start(); }             // delegation
}
new Car(new ElectricEngine()).start();           // hum
```

## Why "Favor Composition" (the arguments)

1. **Fragile base class:** a superclass change silently breaks subclasses coupled to its internals.
2. **The Java Stack fiasco (canonical example):** `Stack extends Vector` → a Stack exposes `insertElementAt()`, letting anyone violate LIFO. Stack *has-a* list would expose only push/pop. Inheritance leaked the parent's whole API.
3. **Runtime flexibility:** composition swaps behavior at runtime (setEngine); inheritance fixes it at compile time — and you can't multiply-inherit your way to combinations (`FlyingSwimmingDuck` class explosion vs composing `FlyBehavior` + `SwimBehavior`).
4. **Encapsulation:** delegation exposes a chosen surface; inheritance exposes the parent's.
5. **Testability:** injected components mock easily.

## When Inheritance IS Right

Genuine is-a with stable contract (LSP holds!), you *want* polymorphic substitution, shared skeletons (Template Method — `AbstractList`), framework extension points designed for it. Test: "would substituting the child anywhere the parent appears always be correct?" No → composition.

| | Inheritance | Composition |
|---|---|---|
| Relationship | is-a | has-a |
| Binding | Compile time | Runtime (swappable) |
| Coupling | Tight (internals leak) | Loose (interface surface) |
| Reuse across hierarchies | ❌ | ✅ (inject anywhere) |
| Polymorphic substitution | Built-in | Via interfaces |
| Risk | Fragile base class, LSP breaks | More boilerplate (delegation) |

## Most-Asked Interview Questions

1. **Composition vs inheritance + when each?** → table + the LSP substitution test.
2. **Why does "favor composition" exist?** → fragile base class + Stack/Vector story + runtime swapping.
3. **Refactor this inheritance to composition.** Pattern: parent becomes an interface + injected strategy; child's unique code stays; shared code moves to a component.
4. **Aggregation vs composition (UML nuance)?** Both has-a; composition = owner controls lifecycle (House–Room), aggregation = independent lifecycles (Team–Player). Light trivia; know the one-liner.
5. **How does this relate to Strategy pattern / DIP?** Composition + interface + injection IS the Strategy pattern and satisfies DIP — one mechanism, three vocabulary words (great connective answer).
6. **Is Java's single inheritance a limitation?** Rarely — interfaces give multiple *typing*, composition gives multiple *behavior reuse*; the constraint pushes better design.
