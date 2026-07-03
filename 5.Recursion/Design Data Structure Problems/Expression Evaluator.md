# Expression Evaluator

> **Topic:** [Recursion](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [282 — Expression Add Operators](https://leetcode.com/problems/expression-add-operators/), [241 — Different Ways to Add Parentheses](https://leetcode.com/problems/different-ways-to-add-parentheses/)

---

## Problem 1 — Expression Add Operators (LC 282)

Given a string of digits, insert `+`, `-`, `*` operators to reach a target. Return all valid expressions.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> addOperators(string num, int target) {
    vector<string> result;
    string expr;
    backtrack(num, target, 0, 0, 0, expr, result);
    return result;
}

// pos: current position in num
// eval: current evaluated value of expression so far
// mult: value of the "last term" — needed to undo multiplication on next '*'
void backtrack(string& num, int target, int pos, long long eval, long long mult,
               string& expr, vector<string>& res) {
    if (pos == (int)num.length()) {
        if (eval == target) res.push_back(expr);
        return;
    }

    int len = expr.length();
    long long cur = 0;

    for (int i = pos; i < (int)num.length(); i++) {
        // Skip leading zeros (but allow single "0")
        if (i != pos && num[pos] == '0') break;
        cur = cur * 10 + (num[i] - '0');

        if (pos == 0) {
            // First number: no operator before it
            expr += to_string(cur);
            backtrack(num, target, i + 1, cur, cur, expr, res);
            expr.resize(len);
        } else {
            // '+' operator
            expr += "+" + to_string(cur);
            backtrack(num, target, i + 1, eval + cur, cur, expr, res);
            expr.resize(len);

            // '-' operator
            expr += "-" + to_string(cur);
            backtrack(num, target, i + 1, eval - cur, -cur, expr, res);
            expr.resize(len);

            // '*' operator — must undo previous term's contribution and re-apply with multiplication
            expr += "*" + to_string(cur);
            backtrack(num, target, i + 1, eval - mult + mult * cur, mult * cur, expr, res);
            expr.resize(len);
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

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> diffWaysToCompute(string expression) {
    vector<int> result;
    for (int i = 0; i < (int)expression.length(); i++) {
        char c = expression[i];
        if (c == '+' || c == '-' || c == '*') {
            // Divide: left of operator, right of operator
            vector<int> left  = diffWaysToCompute(expression.substr(0, i));
            vector<int> right = diffWaysToCompute(expression.substr(i + 1));
            // Conquer: combine all pairs
            for (auto l : left) {
                for (auto r : right) {
                    if      (c == '+') result.push_back(l + r);
                    else if (c == '-') result.push_back(l - r);
                    else               result.push_back(l * r);
                }
            }
        }
    }
    // Base case: no operator found → pure number
    if (result.empty()) result.push_back(stoi(expression));
    return result;
}
```

**Memoized version:** Cache by `expression` substring to avoid recomputation.

```cpp
#include <bits/stdc++.h>
using namespace std;

unordered_map<string, vector<int>> memo;

vector<int> diffWaysToCompute(string expression) {
    if (memo.count(expression)) return memo[expression];
    // ... same logic ...
    memo[expression] = result;
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
