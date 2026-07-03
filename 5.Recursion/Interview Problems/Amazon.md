# Amazon Interview Problems — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Generate Parentheses | SDE I/II | Medium | ⭐⭐⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [22](https://leetcode.com/problems/generate-parentheses/) |
| 2 | Word Search | SDE I | Medium | ⭐⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [79](https://leetcode.com/problems/word-search/) |
| 3 | Combination Sum | SDE I | Medium | ⭐⭐⭐ | [Subsets & Combos](../Patterns/Subsets%20and%20Combinations.md) | [39](https://leetcode.com/problems/combination-sum/) |
| 4 | Subsets II | SDE II | Medium | ⭐⭐⭐ | [Subsets & Combos](../Patterns/Subsets%20and%20Combinations.md) | [90](https://leetcode.com/problems/subsets-ii/) |
| 5 | Word Break II | SDE II | Hard | ⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [140](https://leetcode.com/problems/word-break-ii/) |

---

## Deep Dive: Generate Parentheses

### The Two Invariants

```rust
if open < n { }    // can add '(' — haven't used all open brackets
if close < open { } // can add ')' — can only close what's been opened
```

Why this generates all valid combinations without duplicates:
- Every valid string has exactly n `(` and n `)`.
- The constraints ensure `(` always precedes paired `)` — no invalid states explored.
- At each node, at most two choices → tree has ≤ 2^(2n) nodes, but valid leaves = Catalan(n).

### Follow-up: What's the Catalan Number for n=4?

`C(4) = 14` valid strings. For general n: `C(n) = C(2n, n) / (n+1)`.

### Complexity

Time: O(4^n / √n) — exact count of valid strings × O(n) to copy each.
Space: O(n) per stack frame, max depth 2n.

---

## Deep Dive: Word Break II (LC 140)

Backtracking + memoization. Separate **reachability** from **enumeration**:

```rust
use std::collections::{HashSet, HashMap};

fn backtrack(s: &str, start: usize, dict: &HashSet<String>, memo: &mut HashMap<usize, Vec<String>>) -> Vec<String> {
    if let Some(cached) = memo.get(&start) {
        return cached.clone();
    }
    let mut result: Vec<String> = Vec::new();
    if start == s.len() {
        result.push(String::new());
        return result;
    }

    for end in (start + 1)..=s.len() {
        let word = &s[start..end];
        if dict.contains(word) {
            let suffixes = backtrack(s, end, dict, memo);
            for suffix in &suffixes {
                result.push(if suffix.is_empty() {
                    word.to_string()
                } else {
                    format!("{} {}", word, suffix)
                });
            }
        }
    }
    memo.insert(start, result.clone());
    result
}

fn word_break(s: String, word_dict: Vec<String>) -> Vec<String> {
    let dict: HashSet<String> = word_dict.into_iter().collect();
    let mut memo: HashMap<usize, Vec<String>> = HashMap::new();
    backtrack(&s, 0, &dict, &mut memo)
}
```

**Note:** If memoized list at some `start` is empty → that path is dead. Empty list in memo means "no valid continuation" — this still prunes effectively.

---

## Amazon LP Alignment

| Problem | LP Principle |
|---------|-------------|
| Generate Parentheses | Bias for Action — explain decision tree upfront before coding |
| Word Search | Dive Deep — trace through board marking to show visited logic |
| Combination Sum | Deliver Results — pruning via sort + break is expected, not optional |

---

## Related Files

- [Amazon OA-Qns](../OA-Qns/Amazon.md)
- [Recursion README](../README.md)

> **Last Updated:** 2026-06-26
