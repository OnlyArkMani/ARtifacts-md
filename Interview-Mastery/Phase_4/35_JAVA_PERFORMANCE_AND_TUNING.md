# 35_JAVA_PERFORMANCE_AND_TUNING

1. Introduction

What this concept is

Profiling, GC tuning, hotspots, memory footprint, escape analysis, and JIT optimizations in Java.

Why it exists

Production Java apps require performance tuning; understanding GC and JIT helps diagnose CPU or memory issues.


2. Topics

- Use profilers (async-profiler, Flight Recorder)
- Choose GC algorithm (G1, ZGC) based on pause-time requirements
- Reduce allocations and object churn


3. 5-min revision

Profile before optimizing; prefer algorithmic improvements over micro-optimizations.