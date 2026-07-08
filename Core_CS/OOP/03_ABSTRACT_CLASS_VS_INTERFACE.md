# Abstract Class vs Interface

> **Tag: Theory/recall + code** — a guaranteed Java question; the table + the "when to use which" judgment call.

## Concept

Both define contracts without full implementation. **Abstract class** = a partial class — can hold state, constructors, and mixed concrete/abstract methods; you `extend` one. **Interface** = a pure capability contract — what you can *do* (Comparable, Serializable); you `implement` many.

```java
abstract class Vehicle {                     // partial implementation + state
    protected String plate;                  // state allowed
    Vehicle(String plate) { this.plate = plate; }  // constructor allowed
    void register() { System.out.println(plate + " registered"); }  // concrete
    abstract double fuelEfficiency();        // subclass must fill in
}

interface Chargeable {                       // capability
    void charge();                           // implicitly public abstract
    default int chargeTimeMins() { return 60; }  // Java 8+: default impl
}

class ElectricCar extends Vehicle implements Chargeable {
    ElectricCar(String p) { super(p); }
    double fuelEfficiency() { return 0; }
    public void charge() { System.out.println("charging"); }
}
```

## The Table (know cold)

| | Abstract class | Interface |
|---|---|---|
| Inheritance | `extends` **one** | `implements` **many** |
| State | Instance fields ✅ | Only `public static final` constants |
| Constructors | ✅ | ❌ |
| Method bodies | Any mix | `default`/`static` (Java 8+), `private` (9+) |
| Access modifiers | Any | Methods implicitly public |
| Relationship it models | **is-a** (shared identity + code) | **can-do** (capability) |
| Example | AbstractList, InputStream | Comparable, Runnable, List |

## When to Use Which (the judgment answer)

- **Abstract class:** a family of closely-related classes sharing *state and code* (template with common fields, partial algorithm — Template Method pattern lives here).
- **Interface:** unrelated classes sharing a *capability* (a Bird and a Plane both `Flyable`); public APIs; when you need multiple inheritance of type.
- Modern default: **start with an interface** (looser coupling); introduce an abstract base class only when concrete shared state/code emerges. Java's own libraries pair them: `List` (interface) + `AbstractList` (skeleton).

## Java 8+ nuance (asked increasingly)

`default` methods gave interfaces behavior — so *why do abstract classes still exist?* State + constructors + non-public members. And the **diamond problem returns** for defaults: two interfaces with the same default method → class must override and disambiguate (`InterfaceA.super.method()`). Knowing this rule impresses.

## Most-Asked Interview Questions

1. **Differences?** → the table; lead with single-vs-multiple and state.
2. **When would you choose each?** → is-a + shared code vs can-do capability; the List/AbstractList pairing.
3. **Why can't Java classes extend multiple classes?** Diamond problem: two parents with same method/state → ambiguity. Interfaces were safe (no state) until defaults — hence the explicit-override rule.
4. **Can an abstract class have zero abstract methods? Be instantiated?** Yes (just prevents instantiation); No — but anonymous subclasses can be (`new Vehicle("x"){...}`).
5. **Can an interface extend another interface? A class?** Interfaces extend (multiple!) interfaces; never a class.
6. **What happens: two interfaces, same default method?** Compile error unless the class overrides and picks (`A.super.m()`).
7. **Marker interfaces?** Empty interfaces (Serializable) — pure type-tags checked at runtime; annotations are the modern alternative.
