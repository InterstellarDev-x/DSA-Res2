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

```java
if (open < n)    // can add '(' — haven't used all open brackets
if (close < open) // can add ')' — can only close what's been opened
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

```java
public List<String> wordBreak(String s, List<String> wordDict) {
    Set<String> dict = new HashSet<>(wordDict);
    Map<Integer, List<String>> memo = new HashMap<>();
    return backtrack(s, 0, dict, memo);
}

private List<String> backtrack(String s, int start, Set<String> dict, Map<Integer, List<String>> memo) {
    if (memo.containsKey(start)) return memo.get(start);
    List<String> result = new ArrayList<>();
    if (start == s.length()) { result.add(""); return result; }

    for (int end = start + 1; end <= s.length(); end++) {
        String word = s.substring(start, end);
        if (dict.contains(word)) {
            List<String> suffixes = backtrack(s, end, dict, memo);
            for (String suffix : suffixes) {
                result.add(word + (suffix.isEmpty() ? "" : " " + suffix));
            }
        }
    }
    memo.put(start, result);
    return result;
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
