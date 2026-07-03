# Design: Autocomplete System

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `trie` `hashmap` `autocomplete`
> **LeetCode:** [LC 642](https://leetcode.com/problems/design-search-autocomplete-system/)

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

Design a search autocomplete system. Given a list of historical sentences with their times searched, implement:

- `AutocompleteSystem::new(sentences: Vec<String>, times: Vec<i32>)` — initialise
- `Vec<String> input(c: char)` — accepts one character at a time; returns top 3 most searched sentences matching the current prefix. `'#'` ends the current input and saves it.

Top 3 ranked by: **times searched descending**, then **lexicographic order ascending** for ties.

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core structure | Trie for prefix lookup + HashMap for frequency |
| Ranking | Custom comparator: `candidates.sort_by(\|a, b\| { let fa = freq[a]; let fb = freq[b]; if fa != fb { fb.cmp(&fa) } else { a.cmp(b) } })` |
| Live typing | Accumulate typed characters into `input_str` |
| Save on `#` | Insert `input_str` into trie with updated frequency |
| Trade-off discussion | Trie (fast prefix) vs HashMap (simple but O(n) scan on each char) |

---

## Approaches

| Approach | `input` Time | Space | Notes |
|----------|-------------|-------|-------|
| HashMap + scan | O(n×k) per char | O(n×k) | Scan all sentences for prefix match |
| Trie + DFS | O(p + r) per char | O(n×k) | p = prefix len, r = results |
| Trie + top-k at each node | O(1) per char | O(n×k×3) | Pre-cache top 3 at each node |

---

## Rust Implementation

```rust
use std::collections::HashMap;

struct TrieNode {
    children: HashMap<char, usize>,
    // Could pre-cache top-k here for O(1) lookup
}

struct AutocompleteSystem {
    nodes: Vec<TrieNode>,
    freq: HashMap<String, i32>,
    cur: Option<usize>,
    input_str: String,
}

impl AutocompleteSystem {
    fn new(sentences: Vec<String>, times: Vec<i32>) -> Self {
        let root = TrieNode { children: HashMap::new() };
        let mut sys = AutocompleteSystem {
            nodes: vec![root],
            freq: HashMap::new(),
            cur: Some(0),
            input_str: String::new(),
        };
        for (i, s) in sentences.iter().enumerate() {
            *sys.freq.entry(s.clone()).or_insert(0) = times[i];
            sys.insert(s);
        }
        sys
    }

    fn insert(&mut self, s: &str) {
        let mut node_idx = 0;
        for c in s.chars() {
            if !self.nodes[node_idx].children.contains_key(&c) {
                let new_idx = self.nodes.len();
                self.nodes.push(TrieNode { children: HashMap::new() });
                self.nodes[node_idx].children.insert(c, new_idx);
            }
            node_idx = *self.nodes[node_idx].children.get(&c).unwrap();
        }
    }

    fn dfs(&self, node_idx: usize, sb: &mut String, results: &mut Vec<String>) {
        if self.freq.contains_key(sb.as_str()) {
            results.push(sb.clone());
        }
        // Collect children to avoid borrow conflict during recursion
        let children: Vec<(char, usize)> = self.nodes[node_idx]
            .children
            .iter()
            .map(|(&c, &idx)| (c, idx))
            .collect();
        for (c, child_idx) in children {
            sb.push(c);
            self.dfs(child_idx, sb, results);
            sb.pop();
        }
    }

    fn input(&mut self, c: char) -> Vec<String> {
        if c == '#' {
            *self.freq.entry(self.input_str.clone()).or_insert(0) += 1;
            let s = self.input_str.clone();
            self.insert(&s);
            self.input_str.clear();
            self.cur = Some(0);
            return vec![];
        }

        self.input_str.push(c);
        if let Some(node_idx) = self.cur {
            self.cur = self.nodes[node_idx].children.get(&c).copied();
        }
        let cur_idx = match self.cur {
            Some(idx) => idx,
            None => return vec![],
        };

        // Collect all sentences under cur node via DFS
        let mut candidates: Vec<String> = Vec::new();
        let mut prefix = self.input_str.clone();
        self.dfs(cur_idx, &mut prefix, &mut candidates);

        // Sort: freq desc, then lex asc
        candidates.sort_by(|a, b| {
            let fa = *self.freq.get(a).unwrap_or(&0);
            let fb = *self.freq.get(b).unwrap_or(&0);
            if fa != fb {
                fb.cmp(&fa)
            } else {
                a.cmp(b)
            }
        });
        candidates.truncate(3);
        candidates
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

1. **Top 3 cached at each Trie node — how?** → Store a `BinaryHeap<String>` (size 3) at each node; update on insert. `input()` becomes O(1) per char.
2. **Thread-safe for multiple users?** → Use `Mutex` with `MutexGuard` around the Trie and `freq` map; or use a per-prefix-subtree lock for finer granularity.
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
