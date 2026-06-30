> **Topic:** [Tries](../README.md) · **Interview Tips**

# Complexity Analysis

Time and space bounds for the seven core trie problems. Throughout, `L` is the length of a single word, `N` the number of words/integers, and `M·N` the dimensions of a board.

| Problem | Time | Space | Notes |
| ------- | ---- | ----- | ----- |
| Implement Trie (LC 208) | `insert`/`search`/`startsWith` O(L) | O(ALPHABET · total chars) | Each op walks one root-to-node path of length `L`. |
| Implement Trie II | O(L) per op | O(ALPHABET · total chars) | Adds prefix/word counts per node; per-op cost unchanged. |
| Word Dictionary (LC 211) | `addWord` O(L); `search` O(26^k · L) worst | O(ALPHABET · total chars) | `k` = number of `.` wildcards; each dot branches to all children. |
| Longest Word (LC 720) | O(total chars) build + DFS | O(ALPHABET · total chars) | DFS only descends paths buildable one char at a time. |
| Replace Words (LC 648) | O(sum root lengths) build + O(sum word lengths) replace | O(ALPHABET · total root chars) | Build trie of roots, then match shortest prefix per word. |
| Maximum XOR (LC 421) | O(32 · N) ≈ O(N) | O(32 · N) | Binary trie of 32-bit numbers; greedy MSB-first descent. |
| Word Search II (LC 212) | O(M · N · 4^Lmax) | O(total chars in words) | Trie of words; DFS from each cell with 4-way branching. |

## Key bounds restated

### Insert / search / startsWith — O(L)
Every basic trie operation follows a single path whose length equals the word (or prefix) length `L`, doing constant work per node. There is no dependence on the number of stored words.

### Whole-trie space — O(ALPHABET · total chars)
In the worst case each inserted character creates a new node, and each array-backed node holds `ALPHABET` (e.g. 26) child references, giving O(ALPHABET · total chars). A `HashMap`-backed node reduces the constant factor when the trie is sparse.

### Word Search II — O(M · N · 4^L)
DFS may start from any of the `M · N` board cells, and from each cell explore up to 4 directions per step for up to `Lmax` steps, bounded by the trie's depth. Trie pruning keeps the practical cost far below this worst case.

### Maximum XOR — O(32N)
Inserting and querying each of the `N` numbers walks the fixed 32-bit depth of the binary trie, so total time is O(32 · N), which is linear in `N`.

### Wildcard search is exponential in the number of dots
In Word Dictionary, each `.` forces branching into all (up to 26) children, so a query with `k` dots costs O(26^k · L) in the worst case — manageable for few wildcards but exponential as `k` grows.

> **Last Updated:** 2026-06-26
