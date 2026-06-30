> **Topic:** [Tries](../README.md) · **Interview Tips**

# When to Use a Trie

Recognize the problems where a trie is the right data structure, and understand the trade-offs against simpler alternatives.

## Signal phrases

These keywords in a problem statement strongly hint at a trie:

- **prefix** queries or prefix counting
- **autocomplete** / type-ahead suggestions
- **dictionary** lookups and word validation
- **common prefix** among strings
- **"starts with"** checks
- **multiple string searches** against one large set
- **longest common prefix**

## Trie vs HashSet vs Sorted Array

| Operation            | Trie                  | HashSet              | Sorted Array            |
| -------------------- | --------------------- | -------------------- | ----------------------- |
| Exact lookup         | O(L)                  | O(L) average         | O(L · log N) binary search |
| Prefix query         | O(L) ✓                | Not supported        | O(L · log N) range scan |
| Ordered iteration    | O(total chars), sorted | Not ordered          | O(N), already ordered   |
| Memory               | High (≈26 refs/node)  | Moderate             | Low (compact)           |
| Build cost           | O(total chars)        | O(total chars)       | O(N log N) sort         |

## Trie vs Hashing for prefix queries

A hash set stores each whole string under a hash of its full contents, so it can only answer "does this exact key exist?" Given a prefix `p`, the hash of `p` tells you nothing about the hashes of the words that extend it, so there is no way to enumerate or test prefixes in O(L). You would have to scan every stored word, which is O(N · L). A trie, by contrast, lays out shared prefixes along a single path: walking the `L` characters of `p` lands you exactly at the subtree of all words beginning with `p`, answering the query in O(L) regardless of how many words exist.

## Space cost trade-off

An array trie allocates 26 child references per node (more for larger alphabets), even when most slots stay null, so memory can dominate for sparse data. The trade-off is worth it when prefix operations are frequent and the dataset shares many common prefixes, since shared paths amortize the per-node cost and prefix queries stay O(L). When the alphabet is large or the set is sparse, switch to a `HashMap`-backed node to store only the children that actually exist.

> **Last Updated:** 2026-06-26
