# OOP — Index & Fast Revision

## Tiered Priority

**Tier 1 (asked constantly):**
- `01_FOUR_PILLARS.md` — THE OOP question
- `04_OVERLOADING_VS_OVERRIDING.md` — definitions + output traps
- `03_ABSTRACT_CLASS_VS_INTERFACE.md` — guaranteed Java comparison
- `07_CODE_QUESTIONS.md` — drill before any Java-flavored interview

**Tier 2 (commonly asked):**
- `02_SOLID_PRINCIPLES.md` — spot-the-violation format
- `06_DESIGN_PATTERNS.md` — Singleton (thread-safe!) most asked

**Tier 3 (differentiator):**
- `05_COMPOSITION_VS_INHERITANCE.md` — design-judgment territory

## All Comparison Tables in One Scan

**Encapsulation vs Abstraction:** hide data/internals (access modifiers, implementation-level) vs hide complexity (interfaces, design-level); means vs goal.

**Overloading vs Overriding:** same name different params, compile-time, static types ↔ same signature, runtime, actual object. Statics hide (don't override); fields never polymorphic; overrides can't narrow access or broaden checked exceptions; covariant returns OK.

**Abstract class vs Interface:** extends one + state + constructors + any modifiers ↔ implements many + constants only + default/static methods; is-a with shared code vs can-do capability. Diamond with defaults → explicit `A.super.m()`.

**Composition vs Inheritance:** has-a, runtime-swappable, loose ↔ is-a, compile-time, tight/fragile-base. Test: does LSP substitution truly hold? Stack-extends-Vector = the cautionary tale.

**SOLID smells:** god class (SRP) · instanceof chains (OCP) · overrides that throw (LSP) · empty interface impls (ISP) · `new` concrete deps inside logic (DIP).

**Patterns:** Singleton (one instance; DCL+volatile or enum) · Factory (hide creation) · Observer (1→N notify; GUI listeners) · Strategy (pluggable algorithm; Comparator). Strategy = composition+interface = OCP+DIP.

**Java trap rules:** parent ctor first + virtual call in ctor sees uninitialized child fields · Integer cache −128..127 · Strings immutable · pass-by-value of references · equals ⟺ hashCode together · finally always runs.

## Pre-Interview 10-Minute Drill
Four pillars, one example each (2 min) → overload/override traps #1–3 from file 07 (3 min) → thread-safe singleton from memory (2 min) → abstract-vs-interface table (1 min) → SOLID smell list (2 min).
