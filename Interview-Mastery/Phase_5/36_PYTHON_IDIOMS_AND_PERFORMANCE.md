# 36_PYTHON_IDIOMS_AND_PERFORMANCE

1. Introduction

What this concept is

Pythonic idioms, interpreter details affecting performance, memory model, and best practices for clean, efficient Python.

Why it exists

Writing idiomatic Python yields concise, readable code with good performance characteristics.


2. Key ideas

- Use list comprehensions and generator expressions
- Avoid repeated attribute lookups in hot loops
- Use built-in functions and libraries (itertools, functools)
- Be mindful of object allocations and caching


3. Performance tips

- Use PyPy for long-running workloads when compatible
- Use numpy for numeric heavy lifting; use C extensions for hot paths


4. 5-min revision

Write readable code; profile hotspots; use appropriate libraries for heavy tasks.