# Method Overloading vs Overriding

> **Tag: Theory + "what does this print"** — the definitions are easy; the tricky output questions are where candidates fall. Drill the traps section.

## Concept

- **Overloading:** same method name, **different parameter lists**, same class (or inherited). Resolved at **compile time** from *reference/argument static types*. ("Static/compile-time polymorphism.")
- **Overriding:** subclass redefines an inherited method with the **same signature**. Resolved at **runtime** from the *actual object type*. ("Dynamic/runtime polymorphism.")

```java
class Calculator {                       // OVERLOADING
    int add(int a, int b)        { return a + b; }
    double add(double a, double b){ return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}

class Animal { void sound() { System.out.println("..."); } }
class Dog extends Animal {               // OVERRIDING
    @Override void sound() { System.out.println("woof"); }
}
Animal a = new Dog();
a.sound();                               // "woof" — runtime dispatch
```

## The Table

| | Overloading | Overriding |
|---|---|---|
| Where | Same class | Sub vs superclass |
| Signature | Must differ (params) | Must match |
| Return type | Can differ freely (params must still differ) | Same or **covariant** (subtype) |
| Binding | Compile time (static types) | Runtime (actual object) |
| `static` methods | Can overload | **Cannot override** (hiding instead) |
| `private`/`final` | Can overload | Cannot override |
| Access modifier | Anything | **Cannot be more restrictive** |
| Exceptions | Anything | Cannot throw broader **checked** exceptions |

## The Traps ("what does this print")

**1. Overload resolution uses the REFERENCE type:**
```java
void f(Animal a) { System.out.println("animal"); }
void f(Dog d)    { System.out.println("dog"); }
Animal x = new Dog();
f(x);                       // prints "animal"! — chosen at compile time
```

**2. Static methods are HIDDEN, not overridden:**
```java
class A { static void hi() { System.out.println("A"); } }
class B extends A { static void hi() { System.out.println("B"); } }
A obj = new B();
obj.hi();                   // prints "A" — static binds to reference type
```

**3. Fields are never polymorphic:**
```java
class A { String name = "A"; }
class B extends A { String name = "B"; }
A obj = new B();
System.out.println(obj.name);   // "A" — fields bind statically, only methods dispatch
```

**4. Overload ambiguity:**
```java
void g(int x, long y) {...}   void g(long x, int y) {...}
g(1, 1);                    // compile error: ambiguous
```

**5. null with overloads:** `h(String s)` vs `h(Object o)`; `h(null)` → picks the *most specific* type (String). Two unrelated specific overloads + null → ambiguous error.

## Most-Asked Interview Questions

1. **Overloading vs overriding?** → table; lead with compile-time-signature vs runtime-object.
2. **Predict the output.** → traps 1–3 above cover ~90% of what's asked.
3. **Can you override static/private/final methods?** No / no (not inherited) / no. Static redefinition = hiding (trap 2).
4. **Can overloaded methods differ only in return type?** No — parameters must differ; return type isn't part of the resolution signature.
5. **What is covariant return?** Override may return a *subtype* of the parent's return type (`Animal reproduce()` → `Dog reproduce()`).
6. **Why can't an override be more restrictive / throw broader checked exceptions?** LSP: callers holding the parent reference must not be surprised — the contract can only stay equal or widen.
7. **Constructor overloading vs overriding?** Constructors can overload; never override (not inherited); `this(...)` chains overloads.
