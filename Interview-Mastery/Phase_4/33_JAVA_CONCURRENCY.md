# 33_JAVA_CONCURRENCY

1. Introduction

Key topics

- Executors and thread pools
- CompletableFuture and async patterns
- Concurrent collections (ConcurrentHashMap)


2. Example: thread pool

```java
ExecutorService ex = Executors.newFixedThreadPool(8);
ex.submit(() -> doWork());
ex.shutdown();
```


3. Interview Qs

- Implement producer-consumer using BlockingQueue
- Use CompletableFuture to compose async tasks


4. 5-min revision

Prefer thread pools, avoid raw Thread creation, use concurrent collections and avoid manual locking when possible.