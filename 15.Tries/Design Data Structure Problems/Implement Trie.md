> **Topic:** [Tries](../README.md) · **Design Data Structure**

# Implement Trie (Prefix Tree)

LeetCode [208](https://leetcode.com/problems/implement-trie-prefix-tree/) · Medium · Pattern: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md)

## Problem

Design a trie supporting:
- `void insert(String word)`
- `boolean search(String word)` — exact word present?
- `boolean startsWith(String prefix)` — any word with this prefix?

## Full Solution (LC 208)

```java
class Trie {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        boolean isEnd = false;
    }

    private final TrieNode root;

    public Trie() {
        root = new TrieNode();
    }

    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.isEnd = true;
    }

    public boolean search(String word) {
        TrieNode node = find(word);
        return node != null && node.isEnd;
    }

    public boolean startsWith(String prefix) {
        return find(prefix) != null;
    }

    private TrieNode find(String s) {
        TrieNode node = root;
        for (int i = 0; i < s.length(); i++) {
            int idx = s.charAt(i) - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

The only difference between `search` and `startsWith` is the `isEnd` check: a word must *terminate* at the final node, a prefix only needs the path to *exist*.

## Trie II Extension — count words / prefixes, erase

Many follow-ups ask for multiset semantics: how many times was a word inserted, how many words share a prefix, and remove an instance.

```java
class TrieII {
    static class TrieNode {
        TrieNode[] children = new TrieNode[26];
        int countWord = 0;     // words ending exactly here
        int countPrefix = 0;   // words passing through here
    }

    private final TrieNode root = new TrieNode();

    public void insert(String word) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) node.children[idx] = new TrieNode();
            node = node.children[idx];
            node.countPrefix++;
        }
        node.countWord++;
    }

    public int countWordsEqualTo(String word) {
        TrieNode n = find(word);
        return n == null ? 0 : n.countWord;
    }

    public int countWordsStartingWith(String prefix) {
        TrieNode n = find(prefix);
        return n == null ? 0 : n.countPrefix;
    }

    public void erase(String word) {
        if (countWordsEqualTo(word) == 0) return;
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            node = node.children[word.charAt(i) - 'a'];
            node.countPrefix--;
        }
        node.countWord--;
    }

    private TrieNode find(String s) {
        TrieNode node = root;
        for (int i = 0; i < s.length(); i++) {
            int idx = s.charAt(i) - 'a';
            if (node.children[idx] == null) return null;
            node = node.children[idx];
        }
        return node;
    }
}
```

`erase` here is **lazy deletion**: it decrements counters but leaves nodes in place. This is correct (queries read the counters) and simple. Physically removing nodes is the next follow-up.

## Array vs HashMap children — trade-off

| Aspect | `TrieNode[26]` array | `Map<Character, TrieNode>` |
|--------|----------------------|-----------------------------|
| Child lookup | O(1) array index `c - 'a'` | O(1) amortized + hashing constant |
| Memory / node | 26 references always (sparse waste) | only existing children stored |
| Alphabet | fixed small (a–z) | arbitrary, Unicode, large |
| Ordered traversal | free (index order = lexicographic) | needs sorting |
| Constant factor | smallest — best for hot loops (Word Search II) | larger; `Character` boxing, GC pressure |
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

   ```java
   private boolean delete(TrieNode node, String word, int i) {
       if (i == word.length()) {
           if (!node.isEnd) return false;   // word not present
           node.isEnd = false;
           return isEmpty(node);            // can prune if no children
       }
       int idx = word.charAt(i) - 'a';
       TrieNode child = node.children[idx];
       if (child == null) return false;
       if (delete(child, word, i + 1)) {
           node.children[idx] = null;       // prune dead child
           return !node.isEnd && isEmpty(node);
       }
       return false;
   }
   private boolean isEmpty(TrieNode node) {
       for (TrieNode c : node.children) if (c != null) return false;
       return true;
   }
   ```

2. **Memory optimization — compressed / radix trie (Patricia trie):** collapse chains of single-child nodes into one edge labelled with a substring. Dramatically reduces node count for sparse dictionaries (e.g. URLs, long unique suffixes), at the cost of more complex insert/split logic.

3. **Ternary search tree:** an alternative balancing memory and speed for large alphabets.

See also: [Trie Fundamentals](../Patterns/Trie%20Fundamentals.md), [Word Dictionary.md](Word%20Dictionary.md).

---

> **Last Updated:** 2026-06-26
