# ER Model & Relational Mapping

> **Tag: Theory + applied (draw/map an ER diagram)** — usually a warm-up; the mapping rules are what's actually tested.

## Concept

The **ER model** is the design blueprint: **entities** (things — Student, Course), **attributes** (properties), **relationships** (associations — Enrolls) with **cardinality** (1:1, 1:N, M:N) and **participation** (total = must participate / partial). The **relational model** is the implementation: everything becomes tables (relations) with rows (tuples), columns (attributes), and constraints.

Special pieces to recognize: **weak entity** — can't exist without its owner, identified by owner's key + its partial key (e.g., `Installment` belongs to `Loan`); **multivalued attribute** (person's phone numbers); **composite attribute** (address = street+city); **derived attribute** (age from DOB — usually don't store).

## Mapping Rules (the testable core)

| ER construct | Relational mapping |
|---|---|
| Strong entity | Own table, PK = its key |
| Weak entity | Table with owner's PK (FK) + partial key as composite PK |
| **1:1 relationship** | FK on either side (prefer the total-participation side); or merge tables |
| **1:N relationship** | FK on the **N side** pointing to the 1 side |
| **M:N relationship** | **Junction table**: (FK₁, FK₂) composite PK + relationship attributes |
| Multivalued attribute | Separate table (entity_PK, value) |
| Composite attribute | Flatten into columns |
| Specialization (ISA) | 3 options: table-per-hierarchy (nulls + discriminator), table-per-subclass (joins), table-per-concrete-class (duplication) |

**The one rule everyone gets asked:** *M:N needs a junction table; 1:N just needs a foreign key on the many side.* Relationship attributes (e.g., `grade` in Enrolls) live in the junction table.

## Worked Example

Students enroll in Courses (M:N, with grade); each Course taught by one Instructor (1:N).

```
STUDENT(student_id PK, name, email)
INSTRUCTOR(instructor_id PK, name)
COURSE(course_id PK, title, instructor_id FK → INSTRUCTOR)   ← 1:N as FK
ENROLLMENT(student_id FK, course_id FK, grade,
           PK(student_id, course_id))                        ← M:N junction
```

## Keys vocabulary (bridge to `03_KEYS.md`)

Candidate key (minimal unique) → pick one as primary key; others become unique constraints. FK enforces referential integrity — what happens on delete? `RESTRICT` (block), `CASCADE` (delete children), `SET NULL`.

## Most-Asked Interview Questions

1. **Map this ER diagram to tables.** → apply the table above mechanically; junction table for M:N is the checkpoint.
2. **1:1 vs 1:N vs M:N — how does each become schema?** → FK placement rules; be crisp.
3. **What is a weak entity? Example?** Existence-dependent, composite PK including owner's key; order line-items, loan installments.
4. **Where do relationship attributes go?** M:N → junction table; 1:N → the N-side table.
5. **How do you model inheritance in tables?** Three strategies + tradeoffs: single table (fast, nulls), joined (clean, join cost), concrete (fast reads, duplicated columns) — same debate as ORM inheritance.
6. **Why not one big table?** Redundancy → update/insert/delete anomalies → leads directly into normalization (next file).
