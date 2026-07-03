> **Topic:** [Tries](../README.md) · **Pattern 1 of 3**

# Trie Fundamentals

## Core Concept

A trie stores strings as paths in a tree. Each node owns up to 26 children (one per lowercase letter), and a boolean `isEnd` marks where a complete word terminates. Three operations form the backbone:

- **insert(word):** walk/create one node per character, set `isEnd` at the last node.
- **search(word):** walk the path; return `true` only if the path exists **and** the final node has `isEnd == true`.
- **startsWith(prefix):** walk the path; return `true` if the path simply exists (no `isEnd` check).

Every operation is **O(L)** in the length of the key, completely independent of the number of stored words.

**Recognition signals:** "implement a prefix tree", "support startsWith / hasPrefix", "autocomplete", "count words with a given prefix", "dictionary that grows dynamically".

---

## Problem 1 — Implement Trie (Prefix Tree) · LC 208 · Medium

```cpp
#include <bits/stdc++.h>
using namespace std;

class Trie {
    struct TrieNode {
        TrieNode* children[26];
        bool isEnd = false;
        TrieNode() {
            fill(children, children + 26, nullptr);
        }
    };

    TrieNode* root;

public:
    Trie() {
        root = new TrieNode();
    }

    void insert(string word) {
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
        TrieNode* node = find(word);
        return node != nullptr && node->isEnd;   // path AND terminal flag
    }

    bool startsWith(string prefix) {
        return find(prefix) != nullptr;          // path is enough
    }

private:
    // Walk the trie following the string; return the final node or nullptr.
    TrieNode* find(string s) {
        TrieNode* node = root;
        for (int i = 0; i < (int)s.length(); i++) {
            int idx = s[i] - 'a';
            if (node->children[idx] == nullptr) return nullptr;
            node = node->children[idx];
        }
        return node;
    }
};
```

**Complexity:** `insert` / `search` / `startsWith` are all **O(L)** time, **O(1)** auxiliary. Space for the whole structure is **O(26 · total characters inserted)** in the worst case (no shared prefixes).

---

## Problem 2 — Implement Trie II (count prefixes / word count) · Medium

Trie II turns the trie into a **multiset**: the same word can be inserted multiple times, and we must answer "how many times is exactly this word stored?" and "how many stored words have this prefix?". We add two integer counters per node and an `erase` that decrements them.

```cpp
#include <bits/stdc++.h>
using namespace std;

class TrieII {
    struct TrieNode {
        TrieNode* children[26];
        int countWord = 0;    // words ending exactly at this node
        int countPrefix = 0;  // words passing through this node (prefix)
        TrieNode() {
            fill(children, children + 26, nullptr);
        }
    };

    TrieNode* root = new TrieNode();

public:
    void insert(string word) {
        TrieNode* node = root;
        for (int i = 0; i < (int)word.length(); i++) {
            int idx = word[i] - 'a';
            if (node->children[idx] == nullptr) {
                node->children[idx] = new TrieNode();
            }
            node = node->children[idx];
            node->countPrefix++;           // every node on the path gains a prefix count
        }
        node->countWord++;                 // only the terminal node gains a word count
    }

    int countWordsEqualTo(string word) {
        TrieNode* node = find(word);
        return node == nullptr ? 0 : node->countWord;
    }

    int countWordsStartingWith(string prefix) {
        TrieNode* node = find(prefix);
        return node == nullptr ? 0 : node->countPrefix;
    }

    // Decrement counters along the path; assumes the word was inserted before.
    void erase(string word) {
        if (countWordsEqualTo(word) == 0) return;   // guard: nothing to erase
        TrieNode* node = root;
        for (int i = 0; i < (int)word.length(); i++) {
            int idx = word[i] - 'a';
            node = node->children[idx];
            node->countPrefix--;
        }
        node->countWord--;
    }

private:
    TrieNode* find(string s) {
        TrieNode* node = root;
        for (int i = 0; i < (int)s.length(); i++) {
            int idx = s[i] - 'a';
            if (node->children[idx] == nullptr) return nullptr;
            node = node->children[idx];
        }
        return node;
    }
};
```

**Why two counters?** `countWord` distinguishes the multiset multiplicity of an *exact* word; `countPrefix` answers prefix queries in O(L) without traversing subtrees. `erase` simply walks the same path decrementing both — note we keep the nodes (lazy deletion) rather than physically unlinking them, which is correct and far simpler. A physical delete is a follow-up (see [Implement Trie.md](../Design%20Data%20Structure%20Problems/Implement%20Trie.md)).

**Complexity:** all four operations are **O(L)**.

---

## std::unordered_map-based children variant

When the alphabet is large, sparse, or non-Latin (Unicode, mixed case, arbitrary symbols), a fixed `[26]` array either wastes memory or simply cannot index the character. Swap it for a map:

```cpp
#include <bits/stdc++.h>
using namespace std;

class TrieMap {
    struct TrieNode {
        unordered_map<char, TrieNode*> children;
        bool isEnd = false;
    };

    TrieNode* root = new TrieNode();

public:
    void insert(string word) {
        TrieNode* node = root;
        for (char c : word) {
            if (node->children.count(c) == 0) {
                node->children[c] = new TrieNode();
            }
            node = node->children[c];
        }
        node->isEnd = true;
    }

    bool search(string word) {
        TrieNode* node = find(word);
        return node != nullptr && node->isEnd;
    }

    bool startsWith(string prefix) {
        return find(prefix) != nullptr;
    }

private:
    TrieNode* find(string s) {
        TrieNode* node = root;
        for (char c : s) {
            if (node->children.count(c) == 0) return nullptr;
            node = node->children[c];
        }
        return node;
    }
};
```

`count(c) == 0` followed by `operator[]` is the idiomatic "get-or-create child" for the map variant. The trade-off table lives in [Implement Trie.md](../Design%20Data%20Structure%20Problems/Implement%20Trie.md) and the [README anatomy section](../README.md#triechildren-array26-vs-hashmap).

---

## Dry Run — insert "app", "apple"; search "app", "appl", "apple"

Start: `root` with all children null.

**insert("app"):**
```
root --a--> n1 --p--> n2 --p--> n3(isEnd=true)
```
**insert("apple"):** reuses `a-p-p`, extends with `l-e`:
```
root --a--> n1 --p--> n2 --p--> n3(isEnd=true) --l--> n4 --e--> n5(isEnd=true)
```

**search("app"):** walk `a→p→p` reaches `n3`; `n3.isEnd == true` → **true**.

**search("appl"):** walk `a→p→p→l` reaches `n4`; `n4.isEnd == false` → **false** (it's a prefix, not a stored word — this is exactly why `isEnd` exists).

**search("apple"):** walk to `n5`; `n5.isEnd == true` → **true**.

**startsWith("appl"):** path `a→p→p→l` exists → **true** (no `isEnd` check).

---

## Recognition Signals Recap

- "Prefix", "starts with", "autocomplete", "dictionary".
- Need *both* exact-word and prefix queries fast → trie with `isEnd` (+ counters for II).
- Many words sharing prefixes → trie compresses shared work; a `std::unordered_set` cannot answer prefix queries.

See also: [Trie with Search Variants](Trie%20with%20Search%20Variants.md), [Bitwise Trie (XOR)](Bitwise%20Trie%20(XOR).md).

---

> **Last Updated:** 2026-06-26
