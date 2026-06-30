# Design: Pattern Matcher (Multi-Pattern Search)

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `aho-corasick` `multi-pattern` `trie`

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

Design a system that, given a set of `k` patterns, can find **all occurrences of any pattern** in a text string efficiently.

- `PatternMatcher(String[] patterns)` — preprocess patterns
- `List<int[]> search(String text)` — return `[start, patternIndex]` for each match

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Know naive approach | Run KMP for each pattern: O(n×k + m×k) |
| Know optimal | Aho-Corasick: build once O(m×k), search O(n + total_matches) |
| Understand failure links | How they mirror KMP's LPS but across a Trie |
| Discuss Java's `String.contains` | Uses Boyer-Moore-Horspool internally, O(n×m) worst |

---

## Approaches

| Approach | Build | Search | Notes |
|----------|-------|--------|-------|
| Run KMP per pattern | O(m×k) | O(n×k) | Simple, good for few patterns |
| Trie + DFS | O(m×k) | O(n × trie_depth) | No failure links |
| Aho-Corasick | O(m×k × Σ) | O(n + matches) | Optimal for multi-pattern |

---

## Java Implementation — Aho-Corasick

```java
public class AhoCorasick {
    static class TrieNode {
        int[] next = new int[26];
        int fail;                  // failure link
        List<Integer> output;      // pattern indices ending here

        TrieNode() {
            Arrays.fill(next, -1);
            output = new ArrayList<>();
        }
    }

    private final List<TrieNode> trie = new ArrayList<>();
    private final List<String> patterns;

    public AhoCorasick(String[] patterns) {
        this.patterns = Arrays.asList(patterns);
        trie.add(new TrieNode()); // root

        // Build Trie
        for (int pi = 0; pi < patterns.length; pi++) {
            int cur = 0;
            for (char c : patterns[pi].toCharArray()) {
                int idx = c - 'a';
                if (trie.get(cur).next[idx] == -1) {
                    trie.get(cur).next[idx] = trie.size();
                    trie.add(new TrieNode());
                }
                cur = trie.get(cur).next[idx];
            }
            trie.get(cur).output.add(pi);
        }

        // Build failure links (BFS)
        Queue<Integer> queue = new LinkedList<>();
        trie.get(0).fail = 0;
        for (int c = 0; c < 26; c++) {
            int child = trie.get(0).next[c];
            if (child == -1) {
                trie.get(0).next[c] = 0; // redirect to root
            } else {
                trie.get(child).fail = 0;
                queue.add(child);
            }
        }
        while (!queue.isEmpty()) {
            int u = queue.poll();
            // Merge output of failure link into current node
            trie.get(u).output.addAll(trie.get(trie.get(u).fail).output);
            for (int c = 0; c < 26; c++) {
                int v = trie.get(u).next[c];
                if (v == -1) {
                    trie.get(u).next[c] = trie.get(trie.get(u).fail).next[c];
                } else {
                    trie.get(v).fail = trie.get(trie.get(u).fail).next[c];
                    queue.add(v);
                }
            }
        }
    }

    public List<int[]> search(String text) {
        List<int[]> result = new ArrayList<>();
        int cur = 0;
        for (int i = 0; i < text.length(); i++) {
            int c = text.charAt(i) - 'a';
            cur = trie.get(cur).next[c];
            for (int pi : trie.get(cur).output) {
                int start = i - patterns.get(pi).length() + 1;
                result.add(new int[]{start, pi});
            }
        }
        return result;
    }
}
// Build: O(total_pattern_len × 26)
// Search: O(n + total_matches)
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Build Aho-Corasick | O(Σ × total_pattern_len) | O(Σ × total_pattern_len) |
| Search | O(n + total_matches) | O(n) output |

Where Σ = alphabet size (26 for lowercase).

---

## Edge Cases

- Empty pattern in list → match every position
- Overlapping patterns (`"ab"` and `"abc"`) → both reported independently
- Pattern longer than text → no matches
- Duplicate patterns → report each occurrence multiple times

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Word Search II](https://leetcode.com/problems/word-search-ii/) | LC 212 — Trie + DFS |
| [Stream of Characters](https://leetcode.com/problems/stream-of-characters/) | LC 1032 |
| [Multi Pattern Matching](https://leetcode.com/problems/string-matching-in-an-array/) | LC 1408 |

---

## Follow-up Questions

1. **Aho-Corasick vs running KMP for each pattern?** → AC wins when patterns are many and text is reused; KMP is simpler for a small fixed set.
2. **Case insensitive?** → Lowercase all chars before insertion.
3. **Unicode / large alphabet?** → Use `HashMap<Character, Integer>` at each node instead of `int[26]`.

---

## Company Tags

`Google` `Amazon` `Palantir` `Competitive Programming`

---

## Navigation

| Related |
|---------|
| [Strings README](../README.md) |
| [KMP and Z-Algorithm](../Patterns/KMP%20and%20Z%20Algorithm.md) |
| [Autocomplete System](./Autocomplete%20System.md) |

> **Last Updated:** 2026-06-26
