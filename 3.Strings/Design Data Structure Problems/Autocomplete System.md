# Design: Autocomplete System

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `trie` `hashmap` `autocomplete`
> **LeetCode:** [LC 642](https://leetcode.com/problems/design-search-autocomplete-system/)

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Approaches](#approaches)
4. [Java Implementation](#java-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design a search autocomplete system. Given a list of historical sentences with their times searched, implement:

- `AutocompleteSystem(String[] sentences, int[] times)` — initialise
- `List<String> input(char c)` — accepts one character at a time; returns top 3 most searched sentences matching the current prefix. `'#'` ends the current input and saves it.

Top 3 ranked by: **times searched descending**, then **lexicographic order ascending** for ties.

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core structure | Trie for prefix lookup + HashMap for frequency |
| Ranking | Custom comparator: `(a, b) -> freqMap.get(b) != freqMap.get(a) ? freqMap.get(b) - freqMap.get(a) : a.compareTo(b)` |
| Live typing | Accumulate typed characters into `curInput` |
| Save on `#` | Insert `curInput` into trie with updated frequency |
| Trade-off discussion | Trie (fast prefix) vs HashMap (simple but O(n) scan on each char) |

---

## Approaches

| Approach | `input` Time | Space | Notes |
|----------|-------------|-------|-------|
| HashMap + scan | O(n×k) per char | O(n×k) | Scan all sentences for prefix match |
| Trie + DFS | O(p + r) per char | O(n×k) | p = prefix len, r = results |
| Trie + top-k at each node | O(1) per char | O(n×k×3) | Pre-cache top 3 at each node |

---

## Java Implementation

```java
class AutocompleteSystem {
    private final TrieNode root = new TrieNode();
    private final Map<String, Integer> freq = new HashMap<>();
    private TrieNode cur;
    private final StringBuilder input = new StringBuilder();

    static class TrieNode {
        Map<Character, TrieNode> children = new HashMap<>();
        // Could pre-cache top-k here for O(1) lookup
    }

    public AutocompleteSystem(String[] sentences, int[] times) {
        for (int i = 0; i < sentences.length; i++) {
            freq.put(sentences[i], times[i]);
            insert(sentences[i]);
        }
        cur = root;
    }

    private void insert(String s) {
        TrieNode node = root;
        for (char c : s.toCharArray()) {
            node.children.putIfAbsent(c, new TrieNode());
            node = node.children.get(c);
        }
    }

    public List<String> input(char c) {
        if (c == '#') {
            String sentence = input.toString();
            freq.merge(sentence, 1, Integer::sum);
            insert(sentence);
            input.setLength(0);
            cur = root;
            return new ArrayList<>();
        }

        input.append(c);
        if (cur != null) cur = cur.children.get(c);
        if (cur == null) return new ArrayList<>();

        // Collect all sentences under cur node via DFS
        List<String> candidates = new ArrayList<>();
        String prefix = input.toString();
        dfs(cur, new StringBuilder(prefix), candidates);

        // Sort: freq desc, then lex asc
        candidates.sort((a, b) -> {
            int fa = freq.getOrDefault(a, 0), fb = freq.getOrDefault(b, 0);
            return fa != fb ? fb - fa : a.compareTo(b);
        });
        return candidates.subList(0, Math.min(3, candidates.size()));
    }

    private void dfs(TrieNode node, StringBuilder sb, List<String> results) {
        if (freq.containsKey(sb.toString())) results.add(sb.toString());
        for (Map.Entry<Character, TrieNode> e : node.children.entrySet()) {
            sb.append(e.getKey());
            dfs(e.getValue(), sb, results);
            sb.deleteCharAt(sb.length() - 1);
        }
    }
}
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Constructor | O(n × k) | O(n × k) |
| `input(c)` — normal char | O(p + r log r) | O(r) — results list |
| `input('#')` | O(k) | O(1) |

Where: `n` = sentences, `k` = avg length, `p` = current prefix length, `r` = matching results.

---

## Edge Cases

- Typing `#` as first character → save empty string (handle gracefully)
- Same sentence typed multiple times → frequency increments correctly
- No matching sentences → return empty list
- Prefix matches sentence exactly → include it in results
- Tie in frequency → lexicographic order (test with identical-freq sentences)

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/) | LC 208 |
| [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/) | LC 1268 |
| [Word Search II](https://leetcode.com/problems/word-search-ii/) | LC 212 |

---

## Follow-up Questions

1. **Top 3 cached at each Trie node — how?** → Store a `PriorityQueue<String>` (size 3) at each node; update on insert. `input()` becomes O(1) per char.
2. **Thread-safe for multiple users?** → `synchronized` on the Trie or use `ConcurrentHashMap` for `freq`; lock per prefix subtree for finer granularity.
3. **Disk-based for 10 billion sentences?** → LSM-tree or prefix-sorted file with binary search; cache hot prefixes in memory.
4. **Handle typos / fuzzy matching?** → BK-tree or Levenshtein automaton instead of Trie.

---

## Company Tags

`Google` `Amazon` `Microsoft` `Uber` `Airbnb`

---

## Navigation

| Related |
|---------|
| [Strings README](../README.md) |
| [KMP and Z-Algorithm](../Patterns/KMP%20and%20Z%20Algorithm.md) |
| [Pattern Matcher](./Pattern%20Matcher.md) |

> **Last Updated:** 2026-06-26
