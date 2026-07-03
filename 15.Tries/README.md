# Tries (Prefix Trees)

> ← [Dynamic Programming](../14.Dynamic_Programming/README.md) | [Advanced Strings](../16.Advanced_Strings/README.md) →

A **Trie** (pronounced "try", from re**trie**val) is a tree-shaped data structure that stores a dynamic set of strings, where each node represents a single character and each root-to-node path spells a prefix. Tries trade memory for blazing-fast prefix queries: insert, search, and `startsWith` all run in **O(L)** where `L` is the length of the key, independent of how many keys are stored.

This topic covers **7 problems** spanning 3 patterns: classic trie construction, search variants (wildcards, board search, root replacement), and the bitwise trie used for XOR maximization.

---

## Problem Table

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---------|----|-----------|---------|-----------|
| 1 | Implement Trie (Prefix Tree) | [208](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium | [Trie Fundamentals](Patterns/Trie%20Fundamentals.md) | Amazon, Microsoft, Goldman Sachs, Adobe |
| 2 | Implement Trie II (count prefixes/words) | — | Medium | [Trie Fundamentals](Patterns/Trie%20Fundamentals.md) | Amazon, Google |
| 3 | Add and Search Word (Word Dictionary, `.` wildcard) | [211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | Medium | [Trie with Search Variants](Patterns/Trie%20with%20Search%20Variants.md) | Google, Microsoft, Amazon |
| 4 | Longest Word in Dictionary | [720](https://leetcode.com/problems/longest-word-in-dictionary/) | Medium | [Trie with Search Variants](Patterns/Trie%20with%20Search%20Variants.md) | Google, Adobe |
| 5 | Replace Words (root replacement) | [648](https://leetcode.com/problems/replace-words/) | Medium | [Trie with Search Variants](Patterns/Trie%20with%20Search%20Variants.md) | Amazon, Goldman Sachs |
| 6 | Maximum XOR of Two Numbers in an Array | [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) | Medium | [Bitwise Trie (XOR)](Patterns/Bitwise%20Trie%20(XOR).md) | Microsoft, Google |
| 7 | Word Search II (Trie + DFS on board) | [212](https://leetcode.com/problems/word-search-ii/) | Hard | [Trie with Search Variants](Patterns/Trie%20with%20Search%20Variants.md) | Google, Amazon, Microsoft |

---

## Patterns Summary

| Pattern | Core Idea | Problems |
|---------|-----------|----------|
| [Trie Fundamentals](Patterns/Trie%20Fundamentals.md) | Build a 26-ary tree node-per-character; `isEnd` marks a complete word; per-node counters extend it to multiset semantics. | 1, 2 |
| [Trie with Search Variants](Patterns/Trie%20with%20Search%20Variants.md) | Layer DFS over the trie: `.` wildcards branch to all children, board search prunes leaves, root-replacement finds the shortest matching prefix. | 3, 4, 5, 7 |
| [Bitwise Trie (XOR)](Patterns/Bitwise%20Trie%20(XOR).md) | Treat each integer as a fixed-width binary string (MSB first); a binary trie lets you greedily pick opposite bits to maximize XOR. | 6 |

---

## Design Data Structure Problems

These two problems are most often framed as "design a data structure" interview questions:

| Problem | File | What you implement |
|---------|------|---------------------|
| Implement Trie | [Implement Trie.md](Design%20Data%20Structure%20Problems/Implement%20Trie.md) | `insert`, `search`, `startsWith` (+ Trie II counters and `erase`) |
| Word Dictionary | [Word Dictionary.md](Design%20Data%20Structure%20Problems/Word%20Dictionary.md) | `addWord`, `search` with `.` wildcard via DFS |

---

## Company Coverage

| Company | OA-Qns | Interview Problems | Headline problems |
|---------|--------|--------------------|-------------------|
| Amazon | [link](OA-Qns/Amazon.md) | [link](Interview%20Problems/Amazon.md) | Implement Trie, Word Search II, Replace Words |
| Google | [link](OA-Qns/Google.md) | [link](Interview%20Problems/Google.md) | Word Search II, Word Dictionary, Maximum XOR |
| Microsoft | [link](OA-Qns/Microsoft.md) | [link](Interview%20Problems/Microsoft.md) | Implement Trie, Word Dictionary, Maximum XOR |
| Goldman Sachs | [link](OA-Qns/Goldman%20Sachs.md) | — | Implement Trie, Replace Words |
| Adobe | [link](OA-Qns/Adobe.md) | — | Implement Trie, Longest Word |

---

## Trie Anatomy

Every trie is built from a single node type. The two design choices that matter most are **how children are stored** and **how completed words are marked**.

### The TrieNode

```rust
#[derive(Debug)]
pub struct TrieNode {
    children: [Option<Box<TrieNode>>; 26], // one slot per lowercase letter
    is_end: bool,                           // true if a word ends here
}
```

- **`children` (array of 26):** index a child by `c as usize - 'a' as usize`. O(1) lookup, no hashing overhead, cache-friendly. Costs 26 references per node even when most are `None`.
- **`is_end` flag:** a node existing only means "some word passes through this prefix." `is_end == true` means "a word *terminates* exactly here." Without it you cannot distinguish the stored word `"app"` from the mere prefix of `"apple"`. **Forgetting `is_end` is the #1 trie bug** — every prefix would wrongly report as a complete word.

### children: array[26] vs HashMap

| Aspect | `[Option<Box<TrieNode>>; 26]` | `HashMap<char, Box<TrieNode>>` |
|--------|-------------------------------|--------------------------------|
| Lookup | O(1), array index `c as usize - 'a' as usize` | O(1) amortized, but with hashing constant |
| Memory per node | 26 pointers always allocated (sparse waste) | only allocates entries that exist |
| Alphabet | fixed, small (lowercase a–z) | arbitrary / Unicode / large alphabets |
| Iteration | fixed 0..25 order (lexicographic) | needs sorting for ordered traversal |
| Constant factor | smallest (best for `Word Search II` hot loop) | larger overhead per node |

**Rule of thumb:** use the **array** when the alphabet is a known small fixed set (the LeetCode default of 26 lowercase letters), and use `HashMap` when the alphabet is large, sparse, or unknown (Unicode, full ASCII, mixed case, words with separators).

### When a Trie beats the alternatives

A trie shines whenever a problem mentions **prefixes**: "starts with", autocomplete, dictionary lookups, common prefixes, or running *many* string queries against a shared set. For pure membership with no prefix needs, a `HashSet<String>` is simpler. See [When to Use a Trie](Interview%20Tips/When%20to%20Use%20a%20Trie.md).

---

## Cross-links

- The XOR/binary trie used in [Maximum XOR](Patterns/Bitwise%20Trie%20(XOR).md) is the same structure described in the Bit Manipulation topic: [Bit Search and Trie](../6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md).

---

## Directory Map

- **[Patterns/](Patterns/)** — the three core patterns with full Rust and dry-runs.
- **[Design Data Structure Problems/](Design%20Data%20Structure%20Problems/)** — Implement Trie and Word Dictionary as design questions.
- **[OA-Qns/](OA-Qns/)** — online-assessment frequency by company.
- **[Interview Problems/](Interview%20Problems/)** — deep dives with follow-ups by company.
- **[Interview Tips/](Interview%20Tips/)** — coding tips, common mistakes, decision guidance, complexity.
- **[Most Recent Questions/](Most%20Recent%20Questions/)** — 2024–2026 trend tracking.

---

> **Last Updated:** 2026-06-26
