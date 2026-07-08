# Memory Management: Paging, Segmentation, Virtual Memory

> **Tag: Theory + one standard numerical** (address translation / effective access time). Paging vs segmentation comparison is a top-5 OS question.

## Concept

Programs are written as if they own all memory (addresses 0…max). The OS + MMU translate these **virtual (logical) addresses** to **physical** ones at runtime. This gives: isolation (processes can't touch each other), the illusion of large contiguous memory, and the ability to keep only *needed* parts in RAM (**virtual memory**).

## Paging

- Physical memory split into fixed-size **frames** (e.g., 4 KB); virtual memory into same-size **pages**.
- **Page table** per process maps page# → frame#. Virtual address = (page number, offset); translation replaces page# with frame#, offset unchanged.
- Any page → any free frame ⇒ **no external fragmentation**; but the last page is half-empty on average ⇒ **internal fragmentation** (~½ page per allocation).
- **TLB** (Translation Lookaside Buffer): tiny cache of recent page→frame translations. Without it every memory access = 2 accesses (page table + data).

**Effective Access Time numerical:** TLB hit ratio 90%, TLB lookup 10ns, memory access 100ns:
EAT = 0.9×(10+100) + 0.1×(10+100+100) = 99 + 21 = **120ns**. Know this formula shape.

**Multi-level page tables:** a flat table for 2⁴⁸ addresses is astronomically big; hierarchical tables allocate only used branches (x86-64 uses 4 levels). Inverted page tables: one entry per *frame* — small but slower lookup.

## Segmentation

- Memory divided by *logical meaning*: code, stack, heap segments, each variable-length with (base, limit).
- Matches programmer's view; enables per-segment protection/sharing.
- Variable sizes ⇒ **external fragmentation** (free memory in scattered unusable holes) → compaction needed.

| | Paging | Segmentation |
|---|---|---|
| Division | Fixed-size, physical | Variable-size, logical |
| Fragmentation | Internal | External |
| Visible to programmer? | No | Yes (logical units) |
| Table | page → frame | segment → (base, limit) |
| Modern use | Dominant (with VM) | Mostly legacy/combined |

## Virtual Memory (demand paging)

Pages loaded **only when touched**. Access to a non-resident page → **page fault** → OS finds the page on disk (swap/file), picks a frame (evicting via page-replacement policy if full — see `04_PAGE_REPLACEMENT.md`), loads, updates table, retries the instruction.

- Enables: programs bigger than RAM, fast startup, higher multiprogramming, copy-on-write `fork`.
- **Thrashing:** too many processes, too little RAM → constant faulting, CPU mostly waits on disk. Detect: high fault rate + low CPU use. Fix: fewer processes (reduce multiprogramming), more RAM, **working-set model** (keep each process's actively-used page set resident).

```
Virtual addr:  [ page # | offset ]
                   │
             TLB? hit → frame#         miss → page table walk
                   │                     (not present → PAGE FAULT → load from disk)
Physical addr: [ frame # | offset ]
```

## Most-Asked Interview Questions

1. **Paging vs segmentation?** → the table; lead with fixed/physical vs variable/logical, and internal vs external fragmentation.
2. **What is a page fault and what happens on one?** → the 5-step walk above; distinguish minor (page in memory, not mapped) vs major (disk read).
3. **Internal vs external fragmentation?** Internal: wasted space *inside* an allocated unit (paging). External: free space scattered *between* allocations (segmentation/contiguous allocation).
4. **What is the TLB and why does it matter?** Translation cache; without it VM would double every memory access. Locality makes 95%+ hit rates typical.
5. **What is thrashing? How do you detect/fix it?** Fault-storm from over-committed RAM; working sets don't fit; reduce degree of multiprogramming.
6. **Why multi-level page tables?** Sparse address spaces: only allocate table nodes for used regions — massive space saving over a flat table.
7. **What is copy-on-write?** fork() shares pages read-only; first write by either process faults and copies that page — makes fork cheap.
8. **Stack vs heap?** Stack: automatic, per-thread, LIFO frames, fast, size-limited. Heap: dynamic (malloc/new), shared, fragmentation and leaks possible.
