# Joins — Types & Performance

> **Tag: Applied** — expect to write join queries AND predict row counts; LEFT vs INNER and "why is this join slow" are staples.

## The Types (with the two-table picture)

```
A: (1,alice) (2,bob) (3,carol)        B: (1,math) (1,cs) (4,bio)
```

| Join | Returns | Result on A⋈B (on id) |
|---|---|---|
| **INNER** | Only matching pairs | alice-math, alice-cs |
| **LEFT (outer)** | All of A + matches (NULLs if none) | + bob-NULL, carol-NULL |
| **RIGHT (outer)** | All of B + matches | + NULL-bio |
| **FULL (outer)** | Everything from both, NULL-padded | all of the above |
| **CROSS** | Cartesian product | 3×3 = 9 rows |
| **SELF** | Table joined to itself (alias!) | employees ↔ managers |

- Row-count intuition: INNER ≤ LEFT ≤ FULL; CROSS = |A|×|B|. 1:N joins *multiply* rows — the classic "why did my SUM double" bug (joining orders to items duplicates order rows).
- **Anti-join** ("in A but not B"): `LEFT JOIN … WHERE b.id IS NULL` or `NOT EXISTS` — a favorite query-writing task.
- **The LEFT JOIN + WHERE trap:** a `WHERE b.col = x` filter turns LEFT into INNER (NULLs fail the predicate) — put it in the `ON` clause to keep left semantics. Interviewers love this.

## How Engines Execute Joins (performance depth)

| Algorithm | How | Best when | Complexity |
|---|---|---|---|
| **Nested loop** | For each row of outer, probe inner | Small outer + **index on inner's join key** | O(N×M) unindexed, O(N log M) indexed |
| **Hash join** | Build hash table on smaller side, probe with larger | Large unsorted inputs, equality joins | O(N+M) |
| **Merge join** | Both sides sorted on key, zip together | Pre-sorted (indexes) inputs, range-friendly | O(N+M) after sort |

Practical implications: **index your FK/join columns** (nested loop becomes cheap; missing FK index = #1 slow-join cause); join column types must match (implicit casts kill index use); filter early (planner pushes predicates down — help it).

## Worked Prediction

`SELECT d.name, COUNT(e.id) FROM departments d LEFT JOIN employees e ON e.dept_id = d.id GROUP BY d.name`
→ counts *including* zero-employee departments — but `COUNT(e.id)` (not `COUNT(*)`) makes empty departments show 0 rather than 1. Two subtleties in one query; be able to explain both.

## Most-Asked Interview Questions

1. **INNER vs LEFT join + example output.** → the picture above; state row counts.
2. **Write: employees with no orders / second-highest salary / department-wise max.** Anti-join pattern; `LIMIT/OFFSET` or window functions (`DENSE_RANK`); `GROUP BY` + `HAVING`.
3. **WHERE vs ON in a LEFT JOIN?** ON = match condition (keeps unmatched left rows), WHERE = post-join filter (can silently convert to INNER).
4. **WHERE vs HAVING?** Row filter before grouping vs group filter after aggregation.
5. **Why is this join slow?** No index on join key, joining on mismatched types, huge intermediate result (join order), SELECT * dragging wide rows, OR conditions defeating indexes.
6. **Nested loop vs hash vs merge join?** → table; planner picks by sizes/indexes/sortedness; read `EXPLAIN`.
7. **UNION vs UNION ALL vs JOIN?** Vertical stacking (dedup vs keep-dups) vs horizontal matching — different axes entirely.
8. **Subquery vs join — which is faster?** Modern planners often rewrite one to the other; correlated subqueries executed per-row are the real danger → prefer joins/CTEs, verify with EXPLAIN.
