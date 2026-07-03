> **Topic:** [Tries](../README.md) · **Design Data Structure**

# Implement Trie (Prefix Tree)

LeetCode [208](https://leetcode.com/problems/implement-trie-prefix-tree/) · Medium · Pattern: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md)

## Problem

Design a trie supporting:
- `void insert(string word)`
- `bool search(string word)` — exact word present?
- `bool startsWith(string prefix)` — any word with this prefix?

## Full Solution (LC 208)

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    bool isEnd = false;
    TrieNode() {
        fill(children, children + 26, nullptr);
    }
};

class Trie {
private:
    TrieNode* root;

    TrieNode* find(const string& s) {
        TrieNode* node = root;
        for (char c : s) {
            int idx = c - 'a';
            if (node->children[idx] == nullptr) return nullptr;
            node = node->children[idx];
        }
        return node;
    }

public:
    Trie() {
        root = new TrieNode();
    }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (node->children[idx] == nullptr) {
                node->children[idx] = new TrieNode();
            }
            node = node->children[idx];
        }
        node->isEnd = true;
    }

    bool search(const string& word) {
        TrieNode* node = find(word);
        return node != nullptr && node->isEnd;
    }

    bool startsWith(const string& prefix) {
        return find(prefix) != nullptr;
    }
};
```

The only difference between `search` and `startsWith` is the `isEnd` check: a word must *terminate* at the final node, a prefix only needs the path to *exist*.

## Trie II Extension — count words / prefixes, erase

Many follow-ups ask for multiset semantics: how many times was a word inserted, how many words share a prefix, and remove an instance.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[26];
    int countWord = 0;     // words ending exactly here
    int countPrefix = 0;   // words passing through here
    TrieNode() {
        fill(children, children + 26, nullptr);
    }
};

class TrieII {
private:
    TrieNode* root;

    TrieNode* find(const string& s) {
        TrieNode* node = root;
        for (char c : s) {
            int idx = c - 'a';
            if (node->children[idx] == nullptr) return nullptr;
            node = node->children[idx];
        }
        return node;
    }

public:
    TrieII() {
        root = new TrieNode();
    }

    void insert(const string& word) {
        TrieNode* node = root;
        for (char c : word) {
            int idx = c - 'a';
            if (node->children[idx] == nullptr) node->children[idx] = new TrieNode();
            node = node->children[idx];
            node->countPrefix++;
        }
        node->countWord++;
    }

    int countWordsEqualTo(const string& word) {
        TrieNode* n = find(word);
        return n == nullptr ? 0 : n->countWord;
    }

    int countWordsStartingWith(const string& prefix) {
        TrieNode* n = find(prefix);
        return n == nullptr ? 0 : n->countPrefix;
    }

    void erase(const string& word) {
        if (countWordsEqualTo(word) == 0) return;
        TrieNode* node = root;
        for (char c : word) {
            node = node->children[c - 'a'];
            node->countPrefix--;
        }
        node->countWord--;
    }
};
```

`erase` here is **lazy deletion**: it decrements counters but leaves nodes in place. This is correct (queries read the counters) and simple. Physically removing nodes is the next follow-up.

## Array vs unordered_map children — trade-off

| Aspect | `TrieNode*[26]` array | `unordered_map<char, TrieNode*>` |
|--------|----------------------|-----------------------------|
| Child lookup | O(1) array index `c - 'a'` | O(1) amortized + hashing constant |
| Memory / node | 26 pointers always (sparse waste) | only existing children stored |
| Alphabet | fixed small (a–z) | arbitrary, Unicode, large |
| Ordered traversal | free (index order = lexicographic) | needs sorting |
| Constant factor | smallest — best for hot loops (Word Search II) | larger; more overhead |
| Cache behaviour | contiguous, cache-friendly | scattered allocations |

**Pick the array** for the standard 26-lowercase LeetCode setting and any performance-critical board DFS. **Pick the map** for large/sparse/Unicode alphabets or when memory per node matters more than constant factor.

## Complexity

| Operation | Time | Space (auxiliary) |
|-----------|------|-------------------|
| `insert` | O(L) | O(L) new nodes worst case |
| `search` | O(L) | O(1) |
| `startsWith` | O(L) | O(1) |
| Trie II counters | O(L) | O(1) |

**Whole-structure space:** **O(ALPHABET · N · L)** worst case — `N` words of length `L` with no shared prefixes, each node carrying a 26-slot array. With heavy prefix sharing the real footprint is far smaller.

## Follow-ups

1. **Delete operation (physical):** to actually unlink, recurse to the word's terminal node, clear `isEnd`, and on the way back up remove any child that has no children *and* is not itself a word end. Must be careful not to delete nodes that are prefixes of other words.

   ```cpp
   bool isEmpty(TrieNode* node) {
       for (TrieNode* c : node->children) if (c != nullptr) return false;
       return true;
   }

   bool deleteWord(TrieNode* node, const string& word, int i) {
       if (i == (int)word.length()) {
           if (!node->isEnd) return false;   // word not present
           node->isEnd = false;
           return isEmpty(node);            // can prune if no children
       }
       int idx = word[i] - 'a';
       TrieNode* child = node->children[idx];
       if (child == nullptr) return false;
       if (deleteWord(child, word, i + 1)) {
           node->children[idx] = nullptr;       // prune dead child
           return !node->isEnd && isEmpty(node);
       }
       return false;
   }
   ```

2. **Memory optimization — compressed / radix trie (Patricia trie):** collapse chains of single-child nodes into one edge labelled with a substring. Dramatically reduces node count for sparse dictionaries (e.g. URLs, long unique suffixes), at the cost of more complex insert/split logic.

3. **Ternary search tree:** an alternative balancing memory and speed for large alphabets.

See also: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Word Dictionary.md](Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
