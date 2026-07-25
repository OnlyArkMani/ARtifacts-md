# 34_JAVA_COLLECTIONS_AND_PERFORMANCE

1. Introduction

What this concept is

Details and performance characteristics of Java Collections: ArrayList, LinkedList, HashMap, TreeMap, ConcurrentHashMap, and when to use each.


2. Key comparisons

- ArrayList vs LinkedList: random access vs cheap mid-list insert
- HashMap vs TreeMap: unordered O(1) average vs ordered O(log n)


3. Performance tips

- Pre-size collections when possible
- Use primitive-specialized collections for hot loops (third-party libs)


4. 5-min revision

Know complexity and memory tradeoffs for collection choices.