> **Topic:** [Tries](../README.md) · **Design Data Structure**

# Word Dictionary (Add and Search Word)

LeetCode [211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) · Medium · Pattern: [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md)

## Problem

Design a data structure that supports:
- `void addWord(string word)`
- `bool search(string word)` — `word` may contain `.`, which matches **any single letter**.

## Full Solution (LC 211)

`addWord` is an ordinary trie insert. `search` is a DFS: a literal character follows one child, a `.` branches into **all** existing children.

```cpp
#include <bits/stdc++.h>
using namespace std;

class WordDictionary {
    struct TrieNode {
        TrieNode* children[26];
        bool isEnd = false;
        TrieNode() {
            fill(children, children + 26, nullptr);
        }
    };

    TrieNode* root;

public:
    WordDictionary() {
        root = new TrieNode();
    }

    void addWord(string word) {
        TrieNode* node = root;
        for (int i = 0; i < (int)word.length(); i++) {
            int idx = word[i] - 'a';
            if (node->children[idx] == nullptr) {
                node->children[idx] = new TrieNode();
            }
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(string word) {
        return dfs(word, 0, root);
    }

private:
    bool dfs(string& word, int i, TrieNode* node) {
        if (node == nullptr) return false;
        if (i == (int)word.length()) return node->isEnd;

        char c = word[i];
        if (c == '.') {
            for (auto child : node->children) {        // try every child
                if (child != nullptr && dfs(word, i + 1, child)) return true;
            }
            return false;
        }
        return dfs(word, i + 1, node->children[c - 'a']);  // single concrete child
    }
};
```

The base case `i == word.length()` returns `node->isEnd` — we matched every character and the path must terminate a real word, not just be a prefix.

## Complexity

| Operation | Time | Notes |
|-----------|------|-------|
| `addWord` | O(L) | L = word length |
| `search` (no `.`) | O(L) | single path walk |
| `search` (k dots) | **O(26^k · L)** worst case | each `.` branches into up to 26 subtrees |
| Space | O(L) recursion stack; O(ALPHABET · N · L) for the trie | |

### Why the wildcard makes search exponential

A `.` at some position forces the DFS to try **all 26** children, and each of those may hit *another* `.` deeper down. With `k` dots, the branching multiplies: in the pathological case (a densely populated trie and a query like `".....`") you explore up to `26^k` paths, each of length `L`. Concrete characters between dots only narrow the search; consecutive dots are the expensive case.

In practice the trie is sparse — most child slots are `null` — so the `child != nullptr` guard prunes the vast majority of branches and real-world searches are far faster than the worst-case bound.

## Follow-ups

1. **Multiple wildcards / leading wildcard:** the same DFS already handles any number of `.` anywhere (including the first position, which simply branches at the root). The only cost is the exponential blow-up above. To bound it, you can cap depth or memoize `(node, i)` states if many repeated queries share structure.

2. **Prefix wildcards / suffix search:** for queries like `"*ing"` (match by suffix), store words reversed in a second trie and query the reversed pattern, or use a generalized suffix structure. A single forward trie cannot answer arbitrary-position wildcards efficiently.

3. **`*` matching zero-or-more characters:** that turns the problem into regex matching against a dictionary — typically solved by combining the trie walk with the wildcard DP from string-matching, not a plain DFS.

4. **Bounding worst case:** if dots dominate, an alternative is to bucket words by **length** (an `unordered_map<int, vector<string>>`) and linearly scan only same-length words with a char-by-char `.`-aware compare — O(matching-length · count) which can beat `26^k` when there are few words.

See also: [Trie with Search Variants](../Patterns/Trie%20with%20Search%20Variants.md), [Implement Trie.md](Implement%20Trie.md).

---

> **Last Updated:** 2026-06-26
