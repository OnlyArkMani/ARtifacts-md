# 37_NUMPY

1. Introduction

What this concept is

NumPy provides fast array operations with vectorized computation and efficient memory layout (ndarray).

Why it exists

To perform numerical computations at C speeds while using Python for orchestration.


2. Key concepts

- Broadcasting rules
- Vectorized operations avoid Python loops
- Memory views and strides


3. Python examples

```python
import numpy as np
A = np.arange(1000000).reshape(1000,1000)
B = A.sum(axis=1)
```


4. 5-min revision

Use numpy for bulk numeric operations, prefer contiguous arrays, avoid Python-level loops.