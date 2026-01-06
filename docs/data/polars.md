---
title: "Polars"
---

Extremely fast DataFrame library for Python and Rust. It uses:

- Apache Arrow memory format for efficient data processing
- Lazy evaluation for query optimization
- Multithreading for parallel computations

```python
from polars import pl

df = pl.read_csv("data.csv")
```
