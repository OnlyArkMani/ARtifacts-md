# 26_STRING_ALGORITHMS

1. Introduction

What this concept is

String algorithms cover pattern matching, string searching, suffix structures (suffix array/tree), Z-function, KMP (Knuth–Morris–Pratt), rolling hash (Rabin–Karp), suffix automata, and advanced problems like longest common substring, suffix array construction (SA-IS), and string combinatorics.

Why it exists

Efficient text processing is critical in search engines, bioinformatics, compilers, and compression. Linear or near-linear algorithms enable practical solutions to large-scale problems.

What problem it solves

- Fast substring search and matching under various constraints
- Longest repeated substrings, pattern frequency counting, edit distances

Real-world analogy

Finding a needle (pattern) in a haystack (text) efficiently using preprocessing of the needle or the haystack.

Where it is used in industry

- Search engines, DNA sequencing, plagiarism detection, compression algorithms


2. Intuition Section

KMP

- Build prefix-function (pi) that tells longest proper prefix equal to suffix for each prefix; use it while matching to skip redundant comparisons.

Suffix array

- Sort all suffixes of string; LCP (longest common prefix) array between adjacent suffixes helps find repeated substrings.

Rolling hash

- Use polynomial hash computed over sliding windows to compare strings quickly (with probabilistic collisions)


3. Core Theory

KMP prefix function

- pi[i] = length of longest proper prefix of s[0..i] which is suffix of s[0..i]
- Use automaton-like transitions to avoid rechecking characters

Z-function

- z[i] = longest substring starting at i that matches prefix; useful for pattern concatenation trick

Suffix arrays and LCP

- SA sorts suffixes lexicographically; LCP between adjacent suffixes reveals longest common substrings; SA can be built in O(n) or O(n log n) depending on algorithm

Suffix automaton

- Compact automaton representing all substrings; states correspond to endpos-equivalence classes and support many substring queries in linear time


4. Java Implementation (KMP)

```java
int[] prefixFunction(char[] s){ int n=s.length; int[] pi=new int[n]; for(int i=1;i<n;i++){ int j=pi[i-1]; while(j>0 && s[i]!=s[j]) j=pi[j-1]; if(s[i]==s[j]) j++; pi[i]=j; } return pi; }

List<Integer> kmpSearch(char[] text, char[] pat){ int[] pi = prefixFunction(pat); List<Integer> res=new ArrayList<>(); int j=0; for(int i=0;i<text.length;i++){ while(j>0 && text[i]!=pat[j]) j=pi[j-1]; if(text[i]==pat[j]) j++; if(j==pat.length){ res.add(i-j+1); j=pi[j-1]; } } return res; }
```

Explain: prefix function construction O(m), search O(n)


5. Python Implementation (Rabin–Karp rolling hash)

```python
MOD = 2**61-1  # use large mod for reduced collisions
BASE = 100007

def rolling_hash_search(text, pattern):
    n, m = len(text), len(pattern)
    if m > n: return []
    powB = [1]*(n+1)
    for i in range(n): powB[i+1] = (powB[i]*BASE) % MOD
    h = [0]*(n+1)
    for i,ch in enumerate(text): h[i+1] = (h[i]*BASE + ord(ch)) % MOD
    pat_h = 0
    for ch in pattern: pat_h = (pat_h*BASE + ord(ch)) % MOD
    res = []
    for i in range(n-m+1):
        cur = (h[i+m] - h[i]*powB[m]) % MOD
        if cur == pat_h: res.append(i)
    return res
```

Discuss collision probability and double hashing for safety.


6. Internal Working

Why linear-time pattern matching matters

For long texts and many patterns, naive O(n*m) fails; KMP and Z exploit border structure to skip comparisons.

Suffix structures trade space/time for powerful queries (longest common substring, number of different substrings, etc.)


7. Time Complexity Table

| Algorithm | Complexity |
| --------- | ---------- |
| KMP | O(n + m) |
| Z-function | O(n) |
| Rolling hash search | O(n) average, collisions possible |
| Suffix array (SA-IS) | O(n) |
| Suffix automaton | O(n) |


8. Space Complexity

- SA/LCP O(n), suffix tree O(n) with large constants; KMP O(m)


9. Common Interview Questions

- Implement KMP and explain prefix function
- Longest repeated substring via suffix array + LCP
- Use rolling hash to detect duplicated substrings of length k


10. Common Mistakes

Mistake: relying on single hash modulo with high collision probability for adversarial tests
Fix: double-hash or use randomized base/mod and static checks

Mistake: off-by-one errors in prefix function and matching loop


11. Real Interview Traps

Trap: build naive suffix tree in limited time; prefer suffix array or automaton outlines and explain tradeoffs

Trap: not explaining collision handling for hashing approaches


12. Real World Applications

- Search engines, genomics sequence analysis, compression, plagiarism detection


13. Common LeetCode Problems

- Implement strStr (KMP optional)
- Longest Duplicate Substring (use suffix array or rolling hash + binary search)


14. Pattern Recognition

- If repeated substring queries: build suffix structures
- If single pattern search: KMP or Rabin–Karp are effective


15. 5 Minute Revision

- KMP builds prefix-function to skip matches, Z-function finds prefix matches from pos
- Rolling hash for probabilistic substring matching
- Suffix arrays/trees and suffix automata for powerful substring queries


---
