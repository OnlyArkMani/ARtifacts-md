# Keys in DBMS

> **Tag: Theory/recall** — quick-fire definitions; five minutes to learn, asked in some form constantly.

## The Hierarchy

- **Superkey:** any attribute set that uniquely identifies rows (may have extras).
- **Candidate key:** *minimal* superkey (no removable attribute). A table can have several.
- **Primary key:** the candidate key you choose. NOT NULL + unique; one per table.
- **Alternate key:** the candidate keys you didn't choose (get UNIQUE constraints).
- **Composite key:** key of 2+ columns (e.g., (student_id, course_id) in a junction table).
- **Foreign key:** column(s) referencing another table's PK — enforces **referential integrity**; can be NULL (unless constrained); many per table.
- **Surrogate key:** artificial identifier (auto-increment, UUID) with no business meaning, vs **natural key** (email, SSN).

```
superkeys ⊇ candidate keys ∋ primary key (chosen) 
                          ∖ alternate keys (the rest)
```

## The Table

| Key | Unique? | NULL? | Count per table | Purpose |
|---|---|---|---|---|
| Primary | ✅ | ❌ | 1 | Row identity |
| Candidate | ✅ | (should not) | ≥1 | PK candidates |
| Foreign | ❌ (usually) | ✅ allowed | many | Cross-table integrity |
| Composite | ✅ (as a set) | ❌ if PK | — | Multi-column identity |
| Unique/Alternate | ✅ | ✅ (one NULL, DB-dependent) | many | Enforce other identities |

## Surrogate vs Natural (the judgment question)

| | Surrogate (id BIGINT/UUID) | Natural (email, ISBN) |
|---|---|---|
| Stability | Never changes ✅ | Business facts change (emails do!) ❌ |
| Size/joins | Small, fast | Possibly long strings |
| Meaning | None (needs the row) | Self-descriptive |
| Practice | Default choice | Keep as UNIQUE constraint instead |

Auto-increment vs UUID: auto-increment = compact, ordered (B-tree friendly), but guessable + awkward across shards; UUID = generatable anywhere (distributed-friendly), but 16 bytes and random inserts fragment B-trees (UUIDv7/ULID fix by time-ordering — nice modern flourish, ties to System Design ID generation).

## Referential Integrity Actions

`ON DELETE / ON UPDATE`: `RESTRICT`/`NO ACTION` (block if children exist — default safety), `CASCADE` (propagate — convenient, dangerous on big graphs), `SET NULL` (orphan gracefully). Expect: "what happens when you delete a parent row?"

## Most-Asked Interview Questions

1. **Primary vs unique key?** PK: one, NOT NULL, the identity, usually clustered index. Unique: many allowed, NULL allowed, alternate identities.
2. **Primary vs foreign key?** Identity within table vs reference across tables enforcing integrity.
3. **Candidate vs super key?** Minimality — candidate = superkey with nothing removable.
4. **Can a foreign key be NULL? Reference the same table?** Yes (optional relationship); yes (self-referencing, e.g., employee.manager_id).
5. **Surrogate vs natural key — which and why?** Surrogate for stability/joins; enforce the natural one with UNIQUE. Emails change — bad PKs.
6. **Composite key — when?** Junction tables (M:N); or when the natural identity is genuinely multi-column. Watch column order for index usefulness (leftmost prefix — cross-ref indexing).
7. **What happens on inserting a duplicate PK / violating an FK?** Constraint violation error — the DB is the last line of defense, not the app.
