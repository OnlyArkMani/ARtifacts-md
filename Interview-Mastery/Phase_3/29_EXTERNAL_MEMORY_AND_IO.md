# 29_EXTERNAL_MEMORY_AND_IO

1. Introduction

What this concept is

Algorithms and data structures optimized for data that does not fit in main memory, minimizing I/O between memory and external storage.

Why it exists

Datasets exceeding RAM require I/O-efficient algorithms (external-memory algorithms) and structures (B-trees, external sort).

Key ideas

- Minimize disk seeks and sequentialize accesses
- Use blocking (I/O buffers), multi-way merge for external sorting


2. Examples

- External merge sort
- B-tree and B+ tree for databases
- Bloom filters for approximate membership with small memory


3. Interview points

- Explain tradeoffs between RAM and I/O, analyze I/O complexity in terms of block transfers


4. 5-min revision

When data > memory: use external algorithms, design for sequential reads/writes, batch updates.