# Design Patterns: Singleton, Factory, Observer, Strategy

> **Tag: Theory + code** — know each pattern's problem→solution in one line, a compact Java implementation, and one real usage. These four cover ~80% of fresher pattern questions.

## 1. Singleton — *exactly one instance, globally accessible*

Problem: shared resource (config, connection pool, logger) must have one instance.

```java
public class Config {
    private static volatile Config instance;          // volatile: safe publication
    private Config() {}                               // block outside construction

    public static Config getInstance() {
        if (instance == null) {                       // fast path, no lock
            synchronized (Config.class) {
                if (instance == null)                 // double-checked locking
                    instance = new Config();
            }
        }
        return instance;
    }
}
// Simpler + recommended alternatives: eager init (static final field),
// holder idiom, or enum singleton (serialization/reflection-proof)
```

Follow-ups: **why double-checked + volatile** (instruction reordering can publish a half-built object); **how to break it** (reflection, serialization, multiple classloaders) and that **enum** resists all three; **why it's controversial** (global state, hidden dependency, hard to test — DI containers manage "singletons" better).

## 2. Factory — *centralize object creation, hide concrete classes*

Problem: `new ConcreteClass()` scattered everywhere couples callers to implementations.

```java
interface Notification { void send(String msg); }
class EmailNotification implements Notification { public void send(String m){...} }
class SmsNotification   implements Notification { public void send(String m){...} }

class NotificationFactory {
    static Notification create(String type) {
        return switch (type) {
            case "EMAIL" -> new EmailNotification();
            case "SMS"   -> new SmsNotification();
            default -> throw new IllegalArgumentException(type);
        };
    }
}
Notification n = NotificationFactory.create("EMAIL");  // caller never sees concretes
```

Variants (one-liners): **Simple factory** (above) · **Factory Method** (subclasses decide what to create — `createButton()` overridden per platform) · **Abstract Factory** (family of related products — `UIFactory` → button+menu+dialog per OS). Real examples: `Integer.valueOf()`, JDBC `DriverManager.getConnection()`.

## 3. Observer — *one-to-many change notification*

Problem: object state changes; many others must react — without the subject knowing them concretely. (This is pub/sub in the small — cross-ref System Design queues.)

```java
interface Observer { void update(double price); }

class Stock {
    private final List<Observer> observers = new ArrayList<>();
    private double price;
    void subscribe(Observer o) { observers.add(o); }
    void setPrice(double p) {
        this.price = p;
        observers.forEach(o -> o.update(p));   // notify all
    }
}
class AlertService implements Observer {
    public void update(double p) { if (p > 100) System.out.println("ALERT " + p); }
}
```

Real examples: GUI listeners (`addActionListener`), Kafka consumers conceptually, reactive streams. Follow-ups: push vs pull models; memory-leak via forgotten unsubscribes (why weak references/`removeListener` matter).

## 4. Strategy — *swap an algorithm at runtime*

Problem: multiple ways to do one thing (payment methods, sort orders, pricing rules) — avoid if/else chains.

```java
interface PricingStrategy { double price(double base); }
class RegularPricing implements PricingStrategy { public double price(double b){ return b; } }
class SalePricing    implements PricingStrategy { public double price(double b){ return b * 0.8; } }

class Checkout {
    private PricingStrategy strategy;
    Checkout(PricingStrategy s) { this.strategy = s; }
    double total(double base) { return strategy.price(base); }
}
new Checkout(new SalePricing()).total(100);   // 80.0
```

Real examples: `Comparator` in `Collections.sort(list, cmp)` (a strategy you pass in!), lambda-friendly. Note: Strategy = composition + interface = OCP + DIP in one pattern — say this sentence.

## Quick-Scan

| Pattern | Category | One-liner | Java-library example |
|---|---|---|---|
| Singleton | Creational | One instance | `Runtime.getRuntime()` |
| Factory | Creational | Delegate creation | `Integer.valueOf`, JDBC |
| Observer | Behavioral | Notify subscribers | GUI listeners |
| Strategy | Behavioral | Pluggable algorithm | `Comparator` |

## Most-Asked Interview Questions

1. **Implement a thread-safe Singleton.** → DCL + volatile, and name the simpler alternatives (holder/enum).
2. **Factory vs Abstract Factory?** One product with variants vs families of related products.
3. **Observer vs pub/sub middleware?** Same idea; middleware adds broker, durability, async, decoupled lifecycles (bridge to System Design).
4. **Strategy vs if-else — why bother?** OCP: new strategies without touching existing code; testable in isolation; runtime swap.
5. **Which patterns does Java's standard library use?** Iterator (Collections), Decorator (`BufferedReader(new FileReader(...))` — worth knowing as a bonus 5th), Factory (valueOf), Strategy (Comparator), Observer (listeners), Builder (`StringBuilder`, `Stream.builder`).
6. **When are patterns overkill?** When the problem's variability doesn't exist yet — patterns solve *recurring change*; applying them speculatively adds indirection (YAGNI). Judgment scores.
