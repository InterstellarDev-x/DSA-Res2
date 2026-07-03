# Expression Evaluator

> **Topic:** [Recursion](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [282 — Expression Add Operators](https://leetcode.com/problems/expression-add-operators/), [241 — Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

---

## Problem 1 — Expression Add Operators (LC 282)

Given a string of digits, insert `+`, `-`, `*` operators to reach a target. Return all valid expressions.

```rust
fn add_operators(num: &str, target: i64) -> Vec<String> {
    let mut result = Vec::new();
    let mut expr = String::new();
    let num_bytes = num.as_bytes();
    backtrack(num_bytes, target, 0, 0, 0, &mut expr, &mut result);
    result
}

// pos: current position in num
// eval: current evaluated value of expression so far
// mult: value of the "last term" — needed to undo multiplication on next '*'
fn backtrack(num: &[u8], target: i64, pos: usize, eval: i64, mult: i64,
             expr: &mut String, res: &mut Vec<String>) {
    if pos == num.len() {
        if eval == target {
            res.push(expr.clone());
        }
        return;
    }

    let len = expr.len();
    let mut cur: i64 = 0;

    for i in pos..num.len() {
        // Skip leading zeros (but allow single "0")
        if i != pos && num[pos] == b'0' { break; }
        cur = cur * 10 + (num[i] - b'0') as i64;

        if pos == 0 {
            // First number: no operator before it
            expr.push_str(&cur.to_string());
            backtrack(num, target, i + 1, cur, cur, expr, res);
            expr.truncate(len);
        } else {
            // '+' operator
            expr.push_str(&format!("+{}", cur));
            backtrack(num, target, i + 1, eval + cur, cur, expr, res);
            expr.truncate(len);

            // '-' operator
            expr.push_str(&format!("-{}", cur));
            backtrack(num, target, i + 1, eval - cur, -cur, expr, res);
            expr.truncate(len);

            // '*' operator — must undo previous term's contribution and re-apply with multiplication
            expr.push_str(&format!("*{}", cur));
            backtrack(num, target, i + 1, eval - mult + mult * cur, mult * cur, expr, res);
            expr.truncate(len);
        }
    }
}
```

**Why track `mult` (last term)?**
When inserting `*`, the `eval` already included `mult` as an additive term. Multiplication has higher precedence, so we must subtract `mult` and add `mult * cur` instead.

Example: `2 + 3 * 5` — when we reach `*5`, `eval = 5`, `mult = 3`.
- Correct: `2 + 3*5 = 2 + 15 = 17`
- Formula: `eval - mult + mult * cur = 5 - 3 + 3*5 = 2 + 15 = 17` ✅

---

## Problem 2 — Different Ways to Add Parentheses (LC 241)

Every operator can be a "split point" — recurse on left and right subexpressions.

```rust
fn diff_ways_to_compute(expression: &str) -> Vec<i32> {
    let mut result = Vec::new();
    for (i, c) in expression.char_indices() {
        if c == '+' || c == '-' || c == '*' {
            // Divide: left of operator, right of operator
            let left  = diff_ways_to_compute(&expression[..i]);
            let right = diff_ways_to_compute(&expression[i + 1..]);
            // Conquer: combine all pairs
            for &l in &left {
                for &r in &right {
                    if      c == '+' { result.push(l + r); }
                    else if c == '-' { result.push(l - r); }
                    else             { result.push(l * r); }
                }
            }
        }
    }
    // Base case: no operator found → pure number
    if result.is_empty() {
        result.push(expression.parse::<i32>().unwrap());
    }
    result
}
```

**Memoized version:** Cache by `expression` substring to avoid recomputation.

```rust
use std::collections::HashMap;

fn diff_ways_to_compute_memo(expression: &str, memo: &mut HashMap<String, Vec<i32>>) -> Vec<i32> {
    if let Some(cached) = memo.get(expression) {
        return cached.clone();
    }
    // ... same logic ...
    let result: Vec<i32> = Vec::new();
    memo.insert(expression.to_string(), result.clone());
    result
}
```

---

## How Many Ways? — Catalan Number

The number of distinct parenthesizations of n+1 numbers with n operators is the nth Catalan number: `C(n) = C(2n, n) / (n+1)`.

For n=3 operators: `C(3) = 5` ways.

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Expression Add Operators | O(4^n × n) | O(n) stack |
| Different Ways (no memo) | O(n × Catalan(n)) = O(n × 4^n / n^1.5) | O(n) |
| Different Ways (memoized) | O(n^3) — each substring computed once | O(n^2) cache |

---

## Related Files

- [Recursive Descent Parser](./Recursive%20Descent%20Parser.md)
- [Backtracking](../Patterns/Backtracking.md)
- [Divide & Conquer](../Patterns/Divide%20and%20Conquer.md)

---

**Back:** [Recursion README](../README.md)

> **Last Updated:** 2026-06-26
