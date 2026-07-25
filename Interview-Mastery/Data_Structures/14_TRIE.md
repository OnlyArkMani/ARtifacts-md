# 14_TRIE

1. Introduction

What this concept is

A Trie (pronounced "try") — also called a prefix tree or digital tree — is a tree-like data structure that stores a dynamic set of strings where each node represents a common prefix. Edges represent characters (or tokens), and paths from the root spell keys. Tries make operations involving prefixes efficient: insertion, search, and prefix queries can be performed in O(m) time where m is the length of the key.

Why it exists

Tries were introduced to solve fast retrieval problems where keys share common prefixes. Unlike hash tables, tries provide ordered traversal, prefix queries, and predictable worst-case time proportional to key length rather than the number of stored keys.

What problem it solves

- Fast lookup of strings and prefixes
- Autocomplete / prefix-match problems
- Efficiently storing sparse string sets with shared prefixes to save space and time on lookups

Real-world analogy

Think of a dictionary where words are organized letter by letter down branches: you walk down letters and reach the node representing a full word or a set of words sharing a prefix.

Where it is used in industry

- Autocomplete engines (search boxes)
- Spell checkers and suggestions
- IP routing (Patricia tries / radix trees)
- Implementations of dictionaries and word games


2. Intuition Section

ASCII diagram storing the words: "to", "tea", "ted", "ten", "in"

root
 ├─ t
 │   └─ e
 │       ├─ a (tea*)
 │       ├─ d (ted*)
 │       └─ n (ten*)
 │
 └─ o (to*)

* indicates end-of-word marker

Explain: Shared prefix "t" and "te" reused for multiple words. Searching for a word is walking characters; prefixes stop earlier and return candidate list if needed.


3. Core Theory

Definitions

- Node: stores children mapping (character -> node) and a flag "isWord" (or count, or value)
- Edge: labeled by a character/token
- Depth of node: length of prefix it represents

Terminology

- Trie vs Radix Tree vs Patricia Trie: tries can be compressed (radix/patricia) to merge single-child chains for space efficiency. Patricia tries store edges labeled with strings rather than characters.

Operations

- Insert: iterate characters, create nodes as needed, mark end-of-word
- Search (exact): iterate characters, check isWord flag at end
- StartsWith / Prefix: iterate prefix and return true or gather subtree words
- Delete: unmark isWord and optionally prune nodes with no children

Memory vs time tradeoffs

Tries use O(sum of key lengths) memory. Hash tables store each key separately; tries share prefixes and can be more memory efficient when heavy prefix reuse exists. However, naive tries can waste memory when alphabet size is large and branching sparse — mitigated with maps/dictionaries or compressed/compact representations.


4. Java Implementation

Basic node

```java
public class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isWord = false;
}

public class Trie {
    private TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode cur = root;
        for (char ch : word.toCharArray()) {
            cur = cur.children.computeIfAbsent(ch, c -> new TrieNode());
        }
        cur.isWord = true;
    }

    public boolean search(String word) {
        TrieNode cur = root;
        for (char ch : word.toCharArray()) {
            cur = cur.children.get(ch);
            if (cur == null) return false;
        }
        return cur.isWord;
    }

    public boolean startsWith(String prefix) {
        TrieNode cur = root;
        for (char ch : prefix.toCharArray()) {
            cur = cur.children.get(ch);
            if (cur == null) return false;
        }
        return true;
    }
}
```

Explain every line

- `Map<Character, TrieNode>`: flexible child container; allows sparse branching. An alternative is an array of size 26 for lowercase latin letters, faster but less flexible.
- `computeIfAbsent`: concise Java 8+ idiom to create child if missing.
- `isWord`: boolean marker for end-of-word; can be replaced with count or value in applications like autocomplete weighting.

Production considerations

- Use arrays when alphabet small and dense (e.g., lowercase letters) to reduce object overhead and increase performance.
- Use compact/trie-compressed structures or DAWG (directed acyclic word graph) for large dictionaries.
- For memory-critical systems, use memory pools, packed arrays, or succinct tries.


5. Python Implementation

Basic Python trie

```python
class TrieNode:
    __slots__ = ('children', 'is_word')
    def __init__(self):
        self.children = {}
        self.is_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word: str) -> None:
        cur = self.root
        for ch in word:
            cur = cur.children.setdefault(ch, TrieNode())
        cur.is_word = True

    def search(self, word: str) -> bool:
        cur = self.root
        for ch in word:
            cur = cur.children.get(ch)
            if cur is None:
                return False
        return cur.is_word

    def starts_with(self, prefix: str) -> bool:
        cur = self.root
        for ch in prefix:
            cur = cur.children.get(ch)
            if cur is None:
                return False
        return True
```

Explain: __slots__ reduces per-node dict overhead, making tries far more memory efficient in Python.

Production-grade

- For very large dictionaries, use DAWGs or compressed/patricia tries.
- Consider persisting trie to disk with a compact representation or using specialized libraries.


6. Internal Working

What happens internally

- Each insert walks and creates nodes — allocation cost O(m) for new word of length m. When words share prefixes, allocations are reduced.
- Searches walk at most m nodes.

Memory allocation

- Worst-case memory proportional to sum of lengths. If many keys share prefixes, the shared nodes reduce memory compared to storing each key separately.

Time complexity reasons

- Search/insert/delete: O(m) where m = key length, independent of number of keys stored (great for massive dictionaries with short keys).

Data movement

- Deletion requires careful pruning — after unmarking isWord, walk back and remove nodes with zero children to reclaim memory.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(m)       |
| Delete    | O(m)       |
| Search    | O(m)       |

m = length of key

Why: operations traverse characters and perform constant work per character.


8. Space Complexity

- Best case: O(1) per new identical key insertion (shared entirely) — unrealistic
- Average: O(sum of key lengths) for stored keys
- Worst case: O(ALPHABET * sum lengths) if using dense arrays per node; with maps it's O(sum lengths)


9. Common Interview Questions

Beginner

- Implement Trie with insert/search/prefix

Intermediate

- Implement autocomplete: return top-k words with a given prefix, possibly weighted by frequency
- Implement delete and pruning carefully

Advanced

- Build compressed trie / radix tree
- Build DAWG for extremely large dictionaries

Example Q: How to implement autocomplete? Answer: Walk prefix to node P, then DFS/BFS the subtree collecting words and apply a priority queue for top-k by weight.


10. Common Mistakes

Mistake: using naive list concatenations for collecting words causing high overhead
Why: repeated string concatenation is expensive
Correct: build words with mutable buffers or pass prefix string and append characters during DFS using a list/array

Mistake: not using __slots__ or arrays in Python leading to high memory usage for large dictionaries


11. Real Interview Traps

Trap: mixing tries with hash table approach expectations — tries guarantee O(m) time, but that m could be large; clarify key length constraints.

Trap: forgetting to handle non-lowercase characters and Unicode properly — clarify alphabet


12. Real World Applications

- Autocomplete systems (Google/IDE/command-line)
- Spell-checkers and fuzzy search (with modifications)
- IP routing using compressed tries (radix/patricia)


13. Common LeetCode Problems

Easy
- Implement Trie (Prefix Tree)

Medium
- Add and Search Word - Data structure design (supports '.' wildcard)
- Word Search II (use trie to speed up multiple word search on a board)

Hard
- Build compressed trie from large dictionary or implement DAWG (rare in interviews)


14. Pattern Recognition

When to choose a trie

- Problems with many short string keys and heavy prefix queries
- Autocomplete, prefix counting, or real-time suggestions

Decision framework

- Need prefix queries → trie
- Need prefix + memory constraints with large alphabet → consider hashing or compressed tries


15. Comparison Section

Trie vs HashMap of strings

- Trie: O(m) predictable lookup, supports prefix operations, memory share across prefixes
- HashMap: average O(m) to compute hash or O(1) after hashing, but cannot do prefix queries efficiently

Trie vs Patricia/Radix

- Patricia compresses chains of single-child nodes to save memory
- Use Patricia when dataset has long common chains


16. 5 Minute Revision

Trie

- Prefix tree for strings, O(m) operations
- Node: children map + isWord flag
- Use arrays for small alphabets, maps for flexible alphabets
- Use compression (radix) or DAWG for memory-limited large datasets


---
