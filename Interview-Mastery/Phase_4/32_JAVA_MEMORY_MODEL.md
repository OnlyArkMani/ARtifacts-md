# 32_JAVA_MEMORY_MODEL

1. Introduction

What this concept is

The Java Memory Model (JMM) describes how threads interact through memory and defines the semantics of volatile, synchronized, and final fields.

Why it exists

To provide visibility and ordering guarantees across threads in Java, enabling correct concurrent programs.

Key points

- Happens-before relationship
- volatile provides visibility and ordering (but not mutual exclusion)
- synchronized establishes mutual exclusion and memory barriers


2. Interview topics

- Explain visibility, reordering, and happens-before
- Why double-checked locking requires volatile for the instance reference


3. 5-min revision

Use immutable objects or proper synchronization; prefer higher-level concurrency utilities when possible.