> **Topic:** [Tries](../README.md) · **Design Data Structure**

# Word Dictionary (Add and Search Word)

LeetCode [211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) · Medium · Pattern: [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md)

## Problem

Design a data structure that supports:
- `void addWord(string word)`
- `bool search(string word)` — `word` may contain `.`, which matches **any single letter**.

## Full Solution (LC 211)

`addWord` is an ordinary trie insert. `search` is a DFS: a literal character follows one child, a `.` branches into **all** existing children.

```rust
struct TrieNode {
    children: [Option<Box<TrieNode>>; 26],
    is_end: bool,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            children: std::array::from_fn(|_| None),
            is_end: false,
        }
    }
}

struct WordDictionary {
    root: Box<TrieNode>,
}

impl WordDictionary {
    fn new() -> Self {
        WordDictionary {
            root: Box::new(TrieNode::new()),
        }
    }

    fn add_word(&mut self, word: String) {
        let mut node = &mut self.root;
        for c in word.bytes() {
            let idx = (c - b'a') as usize;
            if node.children[idx].is_none() {
                node.children[idx] = Some(Box::new(TrieNode::new()));
            }
            node = node.children[idx].as_mut().unwrap();
        }
        node.is_end = true;
    }

    fn search(&self, word: String) -> bool {
        Self::dfs(word.as_bytes(), 0, &self.root)
    }

    fn dfs(word: &[u8], i: usize, node: &TrieNode) -> bool {
        if i == word.len() {
            return node.is_end;
        }

        let c = word[i];
        if c == b'.' {
            for child in &node.children {        // try every child
                if let Some(child_node) = child {
                    if Self::dfs(word, i + 1, child_node) {
                        return true;
                    }
                }
            }
            false
        } else {
            let idx = (c - b'a') as usize;
            if let Some(child) = &node.children[idx] {
                Self::dfs(word, i + 1, child)  // single concrete child
            } else {
                false
            }
        }
    }
}
```

The base case `i == word.len()` returns `node.is_end` — we matched every character and the path must terminate a real word, not just be a prefix.

## Complexity

| Operation | Time | Notes |
|-----------|------|-------|
| `addWord` | O(L) | L = word length |
| `search` (no `.`) | O(L) | single path walk |
| `search` (k dots) | **O(26^k · L)** worst case | each `.` branches into up to 26 subtrees |
| Space | O(L) recursion stack; O(ALPHABET · N · L) for the trie | |

### Why the wildcard makes search exponential

A `.` at some position forces the DFS to try **all 26** children, and each of those may hit *another* `.` deeper down. With `k` dots, the branching multiplies: in the pathological case (a densely populated trie and a query like `".....`") you explore up to `26^k` paths, each of length `L`. Concrete characters between dots only narrow the search; consecutive dots are the expensive case.

In practice the trie is sparse — most child slots are `None` — so the `child != None` guard prunes the vast majority of branches and real-world searches are far faster than the worst-case bound.

## Follow-ups

1. **Multiple wildcards / leading wildcard:** the same DFS already handles any number of `.` anywhere (including the first position, which simply branches at the root). The only cost is the exponential blow-up above. To bound it, you can cap depth or memoize `(node, i)` states if many repeated queries share structure.

2. **Prefix wildcards / suffix search:** for queries like `"*ing"` (match by suffix), store words reversed in a second trie and query the reversed pattern, or use a generalized suffix structure. A single forward trie cannot answer arbitrary-position wildcards efficiently.

3. **`*` matching zero-or-more characters:** that turns the problem into regex matching against a dictionary — typically solved by combining the trie walk with the wildcard DP from string-matching, not a plain DFS.

4. **Bounding worst case:** if dots dominate, an alternative is to bucket words by **length** (a `HashMap<i32, Vec<String>>`) and linearly scan only same-length words with a char-by-char `.`-aware compare — O(matching-length · count) which can beat `26^k` when there are few words.

See also: [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Implement Trie.md](Implement%20Trie.md).

---

> **Last Updated:** 2026-06-26
