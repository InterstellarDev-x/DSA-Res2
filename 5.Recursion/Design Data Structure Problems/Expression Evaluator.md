# Expression Evaluator

> **Topic:** [Recursion](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [282 — Expression Add Operators](https://leetcode.com/problems/expression-add-operators/), [241 — Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

---

## Problem 1 — Expression Add Operators (LC 282)

Given a string of digits, insert `+`, `-`, `*` operators to reach a target. Return all valid expressions.

```java
public List<String> addOperators(String num, int target) {
    List<String> result = new ArrayList<>();
    backtrack(num, target, 0, 0, 0, new StringBuilder(), result);
    return result;
}

// pos: current position in num
// eval: current evaluated value of expression so far
// mult: value of the "last term" — needed to undo multiplication on next '*'
private void backtrack(String num, int target, int pos, long eval, long mult,
                        StringBuilder expr, List<String> res) {
    if (pos == num.length()) {
        if (eval == target) res.add(expr.toString());
        return;
    }

    int len = expr.length();
    long cur = 0;

    for (int i = pos; i < num.length(); i++) {
        // Skip leading zeros (but allow single "0")
        if (i != pos && num.charAt(pos) == '0') break;
        cur = cur * 10 + (num.charAt(i) - '0');

        if (pos == 0) {
            // First number: no operator before it
            expr.append(cur);
            backtrack(num, target, i + 1, cur, cur, expr, res);
            expr.setLength(len);
        } else {
            // '+' operator
            expr.append('+').append(cur);
            backtrack(num, target, i + 1, eval + cur, cur, expr, res);
            expr.setLength(len);

            // '-' operator
            expr.append('-').append(cur);
            backtrack(num, target, i + 1, eval - cur, -cur, expr, res);
            expr.setLength(len);

            // '*' operator — must undo previous term's contribution and re-apply with multiplication
            expr.append('*').append(cur);
            backtrack(num, target, i + 1, eval - mult + mult * cur, mult * cur, expr, res);
            expr.setLength(len);
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

```java
public List<Integer> diffWaysToCompute(String expression) {
    List<Integer> result = new ArrayList<>();
    for (int i = 0; i < expression.length(); i++) {
        char c = expression.charAt(i);
        if (c == '+' || c == '-' || c == '*') {
            // Divide: left of operator, right of operator
            List<Integer> left  = diffWaysToCompute(expression.substring(0, i));
            List<Integer> right = diffWaysToCompute(expression.substring(i + 1));
            // Conquer: combine all pairs
            for (int l : left) {
                for (int r : right) {
                    if      (c == '+') result.add(l + r);
                    else if (c == '-') result.add(l - r);
                    else               result.add(l * r);
                }
            }
        }
    }
    // Base case: no operator found → pure number
    if (result.isEmpty()) result.add(Integer.parseInt(expression));
    return result;
}
```

**Memoized version:** Cache by `expression` substring to avoid recomputation.

```java
private Map<String, List<Integer>> memo = new HashMap<>();

public List<Integer> diffWaysToCompute(String expression) {
    if (memo.containsKey(expression)) return memo.get(expression);
    // ... same logic ...
    memo.put(expression, result);
    return result;
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
