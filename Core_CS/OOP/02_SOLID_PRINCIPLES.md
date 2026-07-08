# SOLID Principles (with Java)

> **Tag: Theory + code** — each principle: one line, one violation, one fix. Interviewers want you to *spot violations*, not recite definitions.

## S — Single Responsibility: *one class, one reason to change*

```java
// ❌ Invoice does business logic AND persistence AND printing
class Invoice { double total(){...} void saveToDb(){...} void printPdf(){...} }

// ✅ three reasons to change → three classes
class Invoice { double total() {...} }
class InvoiceRepository { void save(Invoice i) {...} }
class InvoicePrinter { void printPdf(Invoice i) {...} }
```
Symptom of violation: "AND" in the class description; DB change forces edits to business logic.

## O — Open/Closed: *open for extension, closed for modification*

```java
// ❌ every new shape edits this method
double area(Object s){ if (s instanceof Circle) ... else if (s instanceof Square) ... }

// ✅ new shapes = new classes; existing code untouched
interface Shape { double area(); }
class Circle implements Shape { public double area(){ return Math.PI*r*r; } }
class Square implements Shape { public double area(){ return side*side; } }
```
Mechanism: polymorphism replaces type-switches. Spot-the-smell: chained `instanceof`/`switch` on type.

## L — Liskov Substitution: *subtypes must be usable wherever the parent is*

```java
// ❌ classic: Square extends Rectangle breaks setters' contract
Rectangle r = new Square();
r.setWidth(4); r.setHeight(5);
assert r.area() == 20;   // fails! Square forced width==height
```
A subclass that *weakens* what the parent promised (throws new exceptions, ignores parameters, returns nulls) violates LSP. Fix: don't force the is-a; Square and Rectangle are siblings under Shape. Rule of thumb: if you override a method to throw `UnsupportedOperationException`, the hierarchy is wrong.

## I — Interface Segregation: *many small interfaces > one fat one*

```java
// ❌ Robot forced to implement eat()
interface Worker { void work(); void eat(); }

// ✅ split by capability
interface Workable { void work(); }
interface Eatable  { void eat(); }
class Human implements Workable, Eatable {...}
class Robot implements Workable {...}
```
Symptom: empty/`throw` implementations of interface methods.

## D — Dependency Inversion: *depend on abstractions, not concretions*

```java
// ❌ high-level policy hard-wired to a low-level detail
class OrderService { private MySqlDb db = new MySqlDb(); }

// ✅ both depend on an interface; implementation injected
class OrderService {
    private final Database db;
    OrderService(Database db) { this.db = db; }   // constructor injection
}
```
Enables: swapping implementations, unit testing with mocks. Note: *Dependency Injection* is a technique that implements this principle (frameworks: Spring); DIP is the design rule.

## Quick-Scan Table

| Principle | One-liner | Violation smell |
|---|---|---|
| SRP | One reason to change | "AND" classes, god objects |
| OCP | Extend without editing | instanceof/switch-on-type chains |
| LSP | Subtype honors parent's contract | overrides that throw/ignore |
| ISP | No forced unused methods | empty implementations |
| DIP | Abstractions between layers | `new` of concrete deps inside logic |

## Most-Asked Interview Questions

1. **Explain SOLID with examples.** → one violation+fix each; the Square/Rectangle and instanceof examples are expected currency.
2. **Which principle does this code violate?** → use the smell column; often multiple (god class = SRP, and probably DIP).
3. **How do OCP and polymorphism relate?** OCP is the goal, polymorphism the mechanism (strategy pattern is its purest form — cross-ref design patterns).
4. **DIP vs dependency injection vs IoC?** Principle vs technique vs the general inversion concept a framework provides.
5. **Can principles conflict / be overdone?** Yes — speculative interfaces everywhere (YAGNI), 20 one-method classes for a script. Principles serve change-resilience; apply where change is expected. Judgment scores.
6. **Why is LSP the "hardest" one?** It's about behavioral contracts, not syntax — the compiler can't catch it; requires thinking about invariants, pre/postconditions.
