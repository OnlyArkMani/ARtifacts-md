# The Four Pillars of OOP (with Java)

> **Tag: Theory + code** — THE OOP question. One crisp definition + one small Java example per pillar; polymorphism gets the deepest follow-ups.

## 1. Encapsulation — *hide state behind a controlled interface*

Bundle data + methods; expose only what's needed. Protects invariants (balance can't go negative if the field is unreachable).

```java
public class BankAccount {
    private double balance;                 // hidden state

    public void deposit(double amt) {
        if (amt <= 0) throw new IllegalArgumentException();
        balance += amt;                     // invariant enforced here
    }
    public double getBalance() { return balance; }  // read-only access
}
```

Access modifiers: `private` < (default/package) < `protected` (package + subclasses) < `public`.

## 2. Abstraction — *expose WHAT, hide HOW*

Users of a type see the essential operations, not the implementation. Achieved via abstract classes / interfaces.

```java
interface PaymentProcessor {
    void pay(double amount);        // WHAT
}
class UpiProcessor implements PaymentProcessor {
    public void pay(double amount) { /* HOW: UPI flow, retries, APIs */ }
}
// caller code depends only on PaymentProcessor — swap implementations freely
```

**Encapsulation vs abstraction (guaranteed question):** encapsulation hides *data/internals* (implementation-level, access modifiers); abstraction hides *complexity* (design-level, interfaces). Encapsulation is a means; abstraction is the goal.

## 3. Inheritance — *is-a reuse and hierarchy*

Subclass acquires superclass members; models "is-a" (Dog is-a Animal).

```java
class Animal {
    void eat() { System.out.println("eating"); }
}
class Dog extends Animal {
    void bark() { System.out.println("woof"); }
}
new Dog().eat();   // inherited
```

Java: single class inheritance only (diamond problem avoided); multiple *interface* implementation allowed. `super` calls parent constructor/methods. Beware overuse — prefer composition (see `05_COMPOSITION_VS_INHERITANCE.md`).

## 4. Polymorphism — *one interface, many behaviors*

```java
Animal a = new Dog();      // upcast
a.makeSound();             // calls Dog's version — decided at RUNTIME
```

- **Runtime (dynamic) polymorphism = method overriding:** JVM dispatches by the *actual object's* type via virtual method table. This is "true" OOP polymorphism.
- **Compile-time (static) = method overloading:** same name, different parameter lists; resolved by the compiler from *reference* types. (Full comparison: `04_OVERLOADING_VS_OVERRIDING.md`.)

```java
class Animal { void makeSound() { System.out.println("..."); } }
class Dog extends Animal { @Override void makeSound() { System.out.println("woof"); } }
class Cat extends Animal { @Override void makeSound() { System.out.println("meow"); } }

List<Animal> zoo = List.of(new Dog(), new Cat());
zoo.forEach(Animal::makeSound);   // woof, meow — no if/else on type!
```

That last line is the *point* of polymorphism: adding `Bird` requires no change to existing code — this is the Open/Closed Principle in action (cross-ref SOLID).

## Most-Asked Interview Questions

1. **Define the four pillars with examples.** → one line + snippet each; don't recite—show.
2. **Encapsulation vs abstraction?** → data-hiding mechanism vs complexity-hiding design; means vs goal.
3. **How does runtime polymorphism work internally?** Virtual method dispatch: object header → class → vtable → method address; why `static`/`private`/`final` methods aren't polymorphic (no vtable entry).
4. **Why "program to an interface, not an implementation"?** Swappability, testability (mocks), parallel development; the abstraction pillar operationalized.
5. **What breaks encapsulation?** Public fields, returning mutable internals (return a copy or unmodifiable view), excessive getters/setters that leak representation.
6. **Can you override a static method?** No — statics belong to the class, resolved at compile time; redefining hides, not overrides (classic trick question — see file 04).
7. **Real-world example of all four at once?** `List<String> l = new ArrayList<>()`: abstraction (List interface), encapsulation (internal array hidden), inheritance (ArrayList extends AbstractList), polymorphism (swap LinkedList, code unchanged).
