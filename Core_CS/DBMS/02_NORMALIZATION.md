# Normalization (1NF → BCNF)

> **Tag: Applied** — you'll be given a table and asked "what normal form is this / normalize it." Practice the worked example's mechanics.

## Concept

Normalization removes redundancy by decomposing tables so every fact is stored once. Redundancy causes **anomalies**: *update* (change a fact in 10 rows, miss one), *insert* (can't add a course until a student enrolls), *delete* (removing the last enrollment erases the course's existence).

Tool: **functional dependency (FD)** — X → Y means X's value determines Y's (student_id → name).

## The Ladder

**1NF — atomic values:** no repeating groups/arrays in a cell. `phones = "123, 456"` violates → separate rows/table.

**2NF — 1NF + no partial dependency:** no non-key attribute depends on *part* of a composite key.
Violation: `ENROLL(student_id, course_id, grade, course_title)` — course_title depends only on course_id (part of the PK). Fix: split COURSE out.
*Only relevant when the PK is composite.*

**3NF — 2NF + no transitive dependency:** no non-key attribute depends on another *non-key* attribute.
Violation: `STUDENT(id, name, dept_id, dept_name)` — id → dept_id → dept_name. Fix: DEPARTMENT table.

**BCNF — for every nontrivial FD X → Y, X must be a superkey.** Stricter 3NF: even *key attributes* can't depend on non-superkeys. 3NF-but-not-BCNF happens with overlapping candidate keys (rare, but know one example).

One-line ladder: **atomic → whole key → nothing but the key** ("the key, the whole key, and nothing but the key").

## Worked Example (the standard drill)

```
ORDERS(order_id, product_id, customer_name, customer_city,
       product_name, price, qty)          PK = (order_id, product_id)

FDs: order_id → customer_name, customer_city
     product_id → product_name, price
     customer_name → customer_city
     (order_id, product_id) → qty
```

- 1NF ✓ (atomic). 
- 2NF ✗ — customer_name depends on order_id alone; product_name on product_id alone (partial).
  → `ORDER(order_id, customer_name, customer_city)`, `PRODUCT(product_id, product_name, price)`, `ORDER_ITEM(order_id, product_id, qty)`.
- 3NF ✗ in ORDER — order_id → customer_name → customer_city (transitive).
  → `CUSTOMER(customer_name, customer_city)`, `ORDER(order_id, customer_name FK)`.
- Result is BCNF. Show the FD reasoning at each step — that's what's graded.

## Denormalization (the judgment question)

Normalization optimizes *writes/integrity*; reads pay in joins. Deliberately denormalize when: read-heavy analytics, precomputed aggregates, avoiding hot joins at scale — accept redundancy + manage it (triggers, app logic, or rebuildable views). NoSQL data modeling is denormalization-first (cross-ref `08_SQL_VS_NOSQL.md` and System Design). Say: *normalize by default, denormalize with a measured reason.*

## Most-Asked Interview Questions

1. **What normal form is this table? Normalize it.** → identify FDs first, then walk the ladder; the example above is the template.
2. **Explain 2NF vs 3NF difference.** Partial (part of composite key) vs transitive (via another non-key attribute) — give the one-line mnemonic.
3. **What are the anomalies?** Update/insert/delete with one concrete example each.
4. **3NF vs BCNF?** BCNF: *every* determinant is a superkey; 3NF allows FDs into key attributes. BCNF decomposition may lose FD-preservation (why 3NF is the practical target).
5. **When would you denormalize?** → read-heavy/scale reasons + how you contain the redundancy risk.
6. **Is more normalization always better?** No — 4NF/5NF exist but past BCNF is academic for most work; join cost and complexity are real; balance per workload.
