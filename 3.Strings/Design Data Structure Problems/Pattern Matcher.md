# Design: Pattern Matcher (Multi-Pattern Search)

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `aho-corasick` `multi-pattern` `trie`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Approaches](#approaches)
4. [Rust Implementation](#rust-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design a system that, given a set of `k` patterns, can find **all occurrences of any pattern** in a text string efficiently.

- `PatternMatcher(string[] patterns)` — preprocess patterns
- `Vec<(usize, usize)> search(text: &str)` — return `(start, patternIndex)` for each match

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Know naive approach | Run KMP for each pattern: O(n×k + m×k) |
| Know optimal | Aho-Corasick: build once O(m×k), search O(n + total_matches) |
| Understand failure links | How they mirror KMP's LPS but across a Trie |
| Discuss `str::find` | Uses two-way algorithm internally, O(n×m) worst |

---

## Approaches

| Approach | Build | Search | Notes |
|----------|-------|--------|-------|
| Run KMP per pattern | O(m×k) | O(n×k) | Simple, good for few patterns |
| Trie + DFS | O(m×k) | O(n × trie_depth) | No failure links |
| Aho-Corasick | O(m×k × Σ) | O(n + matches) | Optimal for multi-pattern |

---

## Rust Implementation — Aho-Corasick

```rust
use std::collections::VecDeque;

struct TrieNode {
    next: [i32; 26],
    fail: usize,
    output: Vec<usize>,
}

impl TrieNode {
    fn new() -> Self {
        TrieNode {
            next: [-1; 26],
            fail: 0,
            output: Vec::new(),
        }
    }
}

struct AhoCorasick {
    trie: Vec<TrieNode>,
    patterns: Vec<String>,
}

impl AhoCorasick {
    fn new(patterns: Vec<String>) -> Self {
        let mut ac = AhoCorasick {
            trie: vec![TrieNode::new()], // root
            patterns,
        };

        // Build Trie
        for pi in 0..ac.patterns.len() {
            let mut cur = 0usize;
            let pat: Vec<u8> = ac.patterns[pi].bytes().collect();
            for b in pat {
                let idx = (b - b'a') as usize;
                if ac.trie[cur].next[idx] == -1 {
                    ac.trie[cur].next[idx] = ac.trie.len() as i32;
                    ac.trie.push(TrieNode::new());
                }
                cur = ac.trie[cur].next[idx] as usize;
            }
            ac.trie[cur].output.push(pi);
        }

        // Build failure links (BFS)
        let mut q: VecDeque<usize> = VecDeque::new();
        ac.trie[0].fail = 0;
        for c in 0..26usize {
            let child = ac.trie[0].next[c];
            if child == -1 {
                ac.trie[0].next[c] = 0; // redirect to root
            } else {
                ac.trie[child as usize].fail = 0;
                q.push_back(child as usize);
            }
        }
        while let Some(u) = q.pop_front() {
            let fail_u = ac.trie[u].fail;
            // Merge output of failure link into current node
            let fail_output: Vec<usize> = ac.trie[fail_u].output.clone();
            ac.trie[u].output.extend(fail_output);
            for c in 0..26usize {
                let v = ac.trie[u].next[c];
                let fail_next = ac.trie[fail_u].next[c];
                if v == -1 {
                    ac.trie[u].next[c] = fail_next;
                } else {
                    ac.trie[v as usize].fail = fail_next as usize;
                    q.push_back(v as usize);
                }
            }
        }

        ac
    }

    fn search(&self, text: &str) -> Vec<(usize, usize)> {
        let mut result = Vec::new();
        let mut cur = 0usize;
        for (i, b) in text.bytes().enumerate() {
            let c = (b - b'a') as usize;
            cur = self.trie[cur].next[c] as usize;
            for &pi in &self.trie[cur].output {
                let start = i + 1 - self.patterns[pi].len();
                result.push((start, pi));
            }
        }
        result
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
3. **Unicode / large alphabet?** → Use `HashMap<char, usize>` at each node instead of `[i32; 26]`.

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
