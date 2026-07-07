# 08 — Tries (Prefix Trees)

## 1. Pattern Overview

A trie stores strings **character-by-character along paths from the root**, so all strings sharing a prefix share a path. That's the entire value proposition: **prefix operations become O(L)** (L = word length) regardless of how many words are stored — a hash set can check "is this exact word present?" but cannot efficiently answer "does anything start with `pre`?" or "walk all words while I walk a grid."

Structure: each node = one character position, holding (a) links to child characters and (b) an `isWord` flag marking "a stored word ends here." (A word's presence is a *path + flag*, not a node type.)

Why it beats hashing for prefixes: hashing destroys structure (similar strings scatter); the trie preserves shared structure, letting you **search incrementally** — one character at a time, pausing mid-word, branching — which is exactly what wildcard matching and grid searches need.

**Complexity:** insert/search/startsWith all O(L). Space O(total characters) worst case — the real cost; alphabet-width child arrays (26×) can be heavy, hence maps for sparse data.

## 2. Recognition Triggers

- The word **"prefix"** anywhere in the problem → trie until proven otherwise.
- "Autocomplete", "type-ahead", "search suggestions" → trie.
- "Design a dictionary supporting wildcard `.` matching" → trie + DFS over branches.
- **Many words to find inside one grid/text simultaneously** (Word Search II) → put the words in a trie, walk it in lockstep with the grid — turns W independent searches into one shared search.
- "Longest common prefix" over many strings, "shortest unique prefix" → trie node counts.
- "Count words with given prefix" → trie with per-node counters.
- XOR problems: "maximum XOR of two numbers" → **binary trie** over bits (each number = 32-bit path; greedily pick opposite bits).
- Constraint smell: ≤ 3·10^4 words × length ≤ 10, queried repeatedly by prefix → trie is intended. One-shot exact lookups → just use a set.

## 3. Template Code

### Trie (Java)
```java
class Trie {
    // Array works when alphabet is known & dense (26 lowercase). Use HashMap for unicode.
    private static class Node {
        Node[] children = new Node[26];
        boolean isWord = false;           // a word ENDS at this node
    }
    private final Node root = new Node();

    public void insert(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            int i = c - 'a';
            if (cur.children[i] == null)
                cur.children[i] = new Node();   // create path lazily
            cur = cur.children[i];
        }
        cur.isWord = true;                      // flag, don't create a special node
    }

    public boolean search(String word) {
        Node n = walk(word);
        return n != null && n.isWord;           // path exists AND a word ends here
    }

    public boolean startsWith(String prefix) {
        return walk(prefix) != null;            // path existing is enough
    }

    private Node walk(String s) {               // follow the path; null if it breaks
        Node cur = root;
        for (char c : s.toCharArray()) {
            cur = cur.children[c - 'a'];
            if (cur == null) return null;
        }
        return cur;
    }
}
```

### Trie (Python)
```python
class Trie:
    def __init__(self):
        self.root = {}              # nested dicts; "$" key marks end-of-word

    def insert(self, word: str) -> None:
        node = self.root
        for c in word:
            node = node.setdefault(c, {})   # create-or-descend in one call
        node["$"] = True                    # sentinel: a word ends here

    def search(self, word: str) -> bool:
        node = self._walk(word)
        return node is not None and "$" in node

    def starts_with(self, prefix: str) -> bool:
        return self._walk(prefix) is not None

    def _walk(self, s: str):
        node = self.root
        for c in s:
            if c not in node:
                return None
            node = node[c]
        return node
```

## 4. Language-Specific Gotchas

- **Python:** the nested-dict trie is idiomatic and fast to write; pick a sentinel key (`"$"` or `"#"`) that can't collide with real characters. A `class Node` version is clearer for interviews where you'll extend it (counters, word refs).
- **Java:** `Node[26]` assumes lowercase a–z — *say that assumption out loud*; switch to `HashMap<Character, Node>` for general alphabets.
- **Java memory:** 26-pointer arrays × many nodes is heavy; for Word Search II, **prune**: store the matched word on its end node and null it after finding (dedup + speed), and optionally delete child links when a subtree is exhausted.
- **Python:** `setdefault` is the elegant descend-or-create; don't write the 3-line if/else.
- Wildcard search (`.`): must DFS all children — easy in both, but Python's recursion limit applies for pathological inputs.
- Binary tries for XOR: Java use `(num >> b) & 1` from bit 31 down; Python same, but note Python negatives are infinite-bit — mask with `& 0xFFFFFFFF` if negatives possible.

## 5. Worked Examples

### Easy — Implement Trie (LC 208)

**Problem:** Implement `insert`, `search`, `startsWith`.

**Recognition trace:** the definitional problem — the templates above are complete solutions.

**Trace** after `insert("app")`, `insert("apple")`, then queries:
```
root ── a ── p ── p($) ── l ── e($)

search("app")      → walk a,p,p → node exists, has $ → true
search("appl")     → walk a,p,p,l → node exists, NO $ → false
startsWith("appl") → path exists → true
search("apx")      → walk a,p → no 'x' child → false
```

### Medium — Design Add and Search Words (LC 211)

**Problem:** `addWord(word)`, `search(word)` where search may contain `.` matching any single letter.

**Recognition trace:** dictionary + single-char wildcard → trie; on `.`, branch into *every* child → DFS with the remaining suffix.

**Java:**
```java
class WordDictionary {
    private static class Node {
        Node[] children = new Node[26];
        boolean isWord = false;
    }
    private final Node root = new Node();

    public void addWord(String word) {
        Node cur = root;
        for (char c : word.toCharArray()) {
            if (cur.children[c - 'a'] == null) cur.children[c - 'a'] = new Node();
            cur = cur.children[c - 'a'];
        }
        cur.isWord = true;
    }

    public boolean search(String word) {
        return dfs(word, 0, root);
    }
    private boolean dfs(String w, int i, Node node) {
        if (node == null) return false;
        if (i == w.length()) return node.isWord;
        char c = w.charAt(i);
        if (c == '.') {                        // wildcard: try every existing branch
            for (Node child : node.children)
                if (dfs(w, i + 1, child)) return true;
            return false;
        }
        return dfs(w, i + 1, node.children[c - 'a']);  // normal: follow the one path
    }
}
```

**Python:**
```python
class WordDictionary:
    def __init__(self):
        self.root = {}

    def add_word(self, word: str) -> None:
        node = self.root
        for c in word:
            node = node.setdefault(c, {})
        node["$"] = True

    def search(self, word: str) -> bool:
        def dfs(i: int, node: dict) -> bool:
            if i == len(word):
                return "$" in node
            c = word[i]
            if c == ".":
                # branch into every child (skip the "$" sentinel)
                return any(dfs(i + 1, child)
                           for k, child in node.items() if k != "$")
            return c in node and dfs(i + 1, node[c])
        return dfs(0, self.root)
```

**Trace** — dictionary `{bad, dad, mad}`, `search(".ad")`:
```
root ── b ── a ── d($)
     ── d ── a ── d($)
     ── m ── a ── d($)
i=0 '.': branch → try b: dfs("ad" under b) → a→d→$ ✓ → true (short-circuits d, m)
search("b..") : b, then '.': a, then '.': d($) → true
```

### Hard — Word Search II (LC 212)

**Problem:** Given an m×n board of letters and a word list, return every word constructible from adjacent (up/down/left/right) cells without reusing a cell.

**Recognition trace:** many words, one grid. Searching each word separately = W × O(grid DFS) — dead at scale. Invert it: **build a trie of the words, DFS the grid while walking the trie** — one traversal serves all words, and the trie prunes dead prefixes instantly.

**Java:**
```java
public class Solution {
    private static class Node {
        Node[] children = new Node[26];
        String word = null;               // store the full word at its end node
    }

    public List<String> findWords(char[][] board, String[] words) {
        Node root = new Node();
        for (String w : words) {          // build the trie
            Node cur = root;
            for (char c : w.toCharArray()) {
                if (cur.children[c - 'a'] == null) cur.children[c - 'a'] = new Node();
                cur = cur.children[c - 'a'];
            }
            cur.word = w;
        }
        List<String> res = new ArrayList<>();
        for (int r = 0; r < board.length; r++)
            for (int c = 0; c < board[0].length; c++)
                dfs(board, r, c, root, res);
        return res;
    }

    private void dfs(char[][] b, int r, int c, Node node, List<String> res) {
        if (r < 0 || r >= b.length || c < 0 || c >= b[0].length) return;
        char ch = b[r][c];
        if (ch == '#' || node.children[ch - 'a'] == null) return; // visited or dead prefix
        node = node.children[ch - 'a'];
        if (node.word != null) {          // found a complete word at this cell
            res.add(node.word);
            node.word = null;             // dedup: never report it again
        }
        b[r][c] = '#';                    // mark visited IN the board (O(1) space)
        dfs(b, r + 1, c, node, res);
        dfs(b, r - 1, c, node, res);
        dfs(b, r, c + 1, node, res);
        dfs(b, r, c - 1, node, res);
        b[r][c] = ch;                     // backtrack: restore the cell
    }
}
```

**Python:**
```python
def find_words(board: list[list[str]], words: list[str]) -> list[str]:
    root = {}
    for w in words:                       # build trie; store word at end node
        node = root
        for c in w:
            node = node.setdefault(c, {})
        node["$"] = w

    rows, cols = len(board), len(board[0])
    res = []

    def dfs(r: int, c: int, node: dict) -> None:
        ch = board[r][c]
        if ch not in node:                # prefix dies here → prune entire branch
            return
        nxt = node[ch]
        if "$" in nxt:                    # complete word found
            res.append(nxt.pop("$"))      # pop = report once only
        board[r][c] = "#"                 # mark visited
        for dr, dc in ((1, 0), (-1, 0), (0, 1), (0, -1)):
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != "#":
                dfs(nr, nc, nxt)
        board[r][c] = ch                  # backtrack
        if not nxt:                       # optional: prune exhausted subtree
            node.pop(ch)

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)
    return res
```

**Trace** — board `[[o,a],[e,t]]`, words `["oat","eta","oax"]`:
```
board:  o a        trie:  o─a─t($oat)
        e t               o─a─x($oax)
                          e─t─a($eta)

start (0,0)='o': root has 'o' → descend        mark: # a / e t
  nbr (0,1)='a': node has 'a' → descend        mark: # # / e t
    nbr (1,1)='t': node has 't' → "$oat" ✓ report oat, pop $
      nbr (1,0)='e': 't' node has no 'e' → prune
    nbr (1,1) done; 'x' branch never matched (no x on board) → dies for free
start (0,1)='a': root has no 'a' → prune instantly (this is the trie's power)
start (1,0)='e': descend e → nbr (1,1)='t' → descend → nbr (0,1)='a' → "$eta" ✓
result: ["oat", "eta"]
```

## 6. Common Mistakes & Edge Cases

- Treating "path exists" as "word exists" — forgetting the `isWord`/`$` flag (search("app") true just because "apple" is stored).
- Word Search II: reporting duplicates (same word reachable two ways) — null/pop the word marker after first find.
- Forgetting to restore the board cell after DFS (backtracking bug → misses words).
- Wildcard DFS iterating the `"$"` sentinel as if it were a child node (Python dict version).
- Java: assuming lowercase; `c - 'a'` on an uppercase letter silently indexes garbage/negative.
- Building a trie when a `HashSet` suffices (exact lookups only) — over-engineering costs interview time.
- Memory limit on huge dictionaries — mention map-children or pruning if the interviewer pushes scale.
- Empty word list, single-cell board, word longer than total cells.

## 7. Fallback Map

- Exact-match lookups only, no prefix queries → **HashSet** (`01`) — simpler and faster.
- Single word in a grid → plain **backtracking DFS** (`10`), no trie needed.
- Suffix questions ("longest repeated substring") → suffix arrays/automata — beyond interview scope usually; mention, don't implement.
- "Maximum XOR" → binary trie (this pattern over bits) or bit-by-bit greedy with a set (`16`).
- Sorted-order prefix queries ("words in range") → sorted list + **binary search** (`05`).
- Frequency-ranked suggestions → trie nodes carrying top-k lists, or hashmap + heap (`09`).
