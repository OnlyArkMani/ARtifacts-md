# "What Does This Print?" — OOP Code Questions

> **Tag: Applied** — rapid-fire Java output/spot-the-bug questions. Each drills one rule. Cover the answer, predict, check.

## 1. Constructor + inheritance order
```java
class A { A() { System.out.println("A ctor"); show(); }
          void show() { System.out.println("A show"); } }
class B extends A { int x = 5;
          B() { System.out.println("B ctor"); }
          @Override void show() { System.out.println("B show, x=" + x); } }
new B();
```
**Output:** `A ctor` → `B show, x=0` → `B ctor`.
Rules: parent constructor runs first; virtual dispatch works even from the parent's constructor — but child fields aren't initialized yet (x=0). *Never call overridable methods from constructors.*

## 2. Static binding of fields & static methods
```java
class A { String name = "A"; static String who() { return "A"; } }
class B extends A { String name = "B"; static String who() { return "B"; } }
A obj = new B();
System.out.println(obj.name + " " + obj.who());
```
**Output:** `A A`. Fields and statics bind to the **reference type**; only instance methods dispatch dynamically.

## 3. Overload resolution is compile-time
```java
static void f(Object o) { System.out.println("Object"); }
static void f(String s) { System.out.println("String"); }
Object x = "hello";
f(x);            // ?
f("hello");      // ?
f(null);         // ?
```
**Output:** `Object`, `String`, `String` (most specific applicable type for null).

## 4. Integer caching + equality
```java
Integer a = 127, b = 127, c = 128, d = 128;
System.out.println((a == b) + " " + (c == d) + " " + c.equals(d));
```
**Output:** `true false true`. Autoboxing caches −128..127; `==` on objects compares references. Rule: **always `.equals()` for wrappers/Strings.** (String pool version: `"ab" == "ab"` true, `new String("ab") == "ab"` false.)

## 5. String immutability
```java
String s = "hello";
s.toUpperCase();
System.out.println(s);
```
**Output:** `hello` — Strings are immutable; methods return new objects. (`s = s.toUpperCase()` fixes.)

## 6. Pass-by-value (of references)
```java
static void change(StringBuilder sb, StringBuilder sb2) {
    sb.append(" world");        // mutates shared object
    sb2 = new StringBuilder("new");  // rebinding local copy only
}
StringBuilder x = new StringBuilder("hello"), y = new StringBuilder("orig");
change(x, y);
System.out.println(x + " | " + y);
```
**Output:** `hello world | orig`. Java is **always pass-by-value** — the value is a reference copy: you can mutate the object, not re-point the caller's variable.

## 7. Shared static state
```java
class Counter { static int count = 0; Counter() { count++; } }
new Counter(); new Counter(); new Counter();
System.out.println(Counter.count);
```
**Output:** `3` — statics are per-class, not per-instance. Follow-up: not thread-safe (`count++` races — cross-ref OS synchronization).

## 8. try/finally return trap
```java
static int f() {
    try { return 1; }
    finally { System.out.println("finally"); }
}
```
**Output:** prints `finally`, returns 1 — finally always runs; a `return` inside finally would *override* the try's return (and is a code smell).

## 9. LSP violation spot-the-bug
```java
class Bird { void fly() {...} }
class Penguin extends Bird {
    @Override void fly() { throw new UnsupportedOperationException(); }
}
```
**Bug:** any code holding `Bird` breaks on a Penguin — LSP violation. Fix: `Bird` → `FlyingBird extends Bird`; Penguin extends Bird only. (Cross-ref SOLID.)

## 10. equals/hashCode contract
```java
class Point { int x, y;  // equals overridden, hashCode NOT
    @Override public boolean equals(Object o) { /* compares x,y */ } }
Set<Point> set = new HashSet<>();
set.add(new Point(1,2));
System.out.println(set.contains(new Point(1,2)));
```
**Output:** almost certainly `false` — HashSet finds the bucket by hashCode first; unequal default hashCodes → never even calls equals. **Rule: override both, together, always.**

## The Rules Behind Everything (revision list)
Parent ctor first; ctors + virtual calls = danger · fields/statics bind statically, instance methods dynamically · overloads picked at compile time by static type · Integer cache −128..127; `.equals()` not `==` · Strings immutable · pass-by-value of references · finally always runs · equals ⟺ hashCode · LSP: overrides may not narrow contracts.
