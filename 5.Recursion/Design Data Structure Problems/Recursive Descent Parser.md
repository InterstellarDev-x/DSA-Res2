# Recursive Descent Parser

> **Topic:** [Recursion](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [224 — Basic Calculator](https://leetcode.com/problems/basic-calculator/), [227 — Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/)

---

## Problem

Parse and evaluate a string expression containing integers, operators (`+`, `-`, `*`, `/`), and parentheses. No spaces vs spaces handled separately.

---

## Design

A **recursive descent parser** mirrors the grammar of the expression:

```
expr   → term (('+' | '-') term)*
term   → factor (('*' | '/') factor)*
factor → number | '(' expr ')'
```

Each grammar rule becomes a method. Operator precedence falls naturally from method call depth: `expr` handles lowest precedence (`+`, `-`), `term` handles higher (`*`, `/`).

---

## Full Implementation — Expression with all operators + parentheses

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    string s;
    int pos;

    // expr → term (('+' | '-') term)*
    int parseExpr() {
        int result = parseTerm();
        while (pos < (int)s.length() && (s[pos] == '+' || s[pos] == '-')) {
            char op = s[pos++];
            int term = parseTerm();
            result = (op == '+') ? result + term : result - term;
        }
        return result;
    }

    // term → factor (('*' | '/') factor)*
    int parseTerm() {
        int result = parseFactor();
        while (pos < (int)s.length() && (s[pos] == '*' || s[pos] == '/')) {
            char op = s[pos++];
            int factor = parseFactor();
            result = (op == '*') ? result * factor : result / factor;
        }
        return result;
    }

    // factor → number | '(' expr ')'
    int parseFactor() {
        skipSpaces();
        if (pos < (int)s.length() && s[pos] == '(') {
            pos++; // consume '('
            int val = parseExpr();
            pos++; // consume ')'
            return val;
        }
        return parseNumber();
    }

    int parseNumber() {
        skipSpaces();
        int sign = 1;
        if (pos < (int)s.length() && s[pos] == '-') { sign = -1; pos++; }
        int num = 0;
        while (pos < (int)s.length() && isdigit(s[pos])) {
            num = num * 10 + (s[pos++] - '0');
        }
        skipSpaces();
        return sign * num;
    }

    void skipSpaces() {
        while (pos < (int)s.length() && s[pos] == ' ') pos++;
    }

public:
    int calculate(string s) {
        this->s = s;
        this->pos = 0;
        return parseExpr();
    }
};
```

---

## Basic Calculator (LC 224) — Only +, -, parentheses

Simpler: only addition/subtraction, no precedence levels. Use a stack to handle parentheses.

```cpp
#include <bits/stdc++.h>
using namespace std;

int calculate(string s) {
    stack<int> stk;
    int result = 0, num = 0, sign = 1;

    for (char c : s) {
        if (isdigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num; num = 0; sign = 1;
        } else if (c == '-') {
            result += sign * num; num = 0; sign = -1;
        } else if (c == '(') {
            // Save current result and sign; start fresh
            stk.push(result);
            stk.push(sign);
            result = 0; sign = 1;
        } else if (c == ')') {
            result += sign * num; num = 0;
            int topSign = stk.top(); stk.pop();
            result *= topSign;   // multiply by sign before '('
            int prevResult = stk.top(); stk.pop();
            result += prevResult;   // add result before '('
        }
    }
    return result + sign * num;
}
```

---

## Basic Calculator II (LC 227) — +, -, *, / (no parentheses)

Use a stack: push positive/negative for `+`/`-`, and apply `*`/`/` immediately.

```cpp
#include <bits/stdc++.h>
using namespace std;

int calculate(string s) {
    stack<int> stk;
    int num = 0;
    char op = '+';

    for (int i = 0; i < (int)s.length(); i++) {
        char c = s[i];
        if (isdigit(c)) num = num * 10 + (c - '0');
        if ((!isdigit(c) && c != ' ') || i == (int)s.length() - 1) {
            if (op == '+') stk.push(num);
            else if (op == '-') stk.push(-num);
            else if (op == '*') {
                int top = stk.top(); stk.pop();
                stk.push(top * num);
            } else if (op == '/') {
                int top = stk.top(); stk.pop();
                stk.push(top / num);
            }
            op = c;
            num = 0;
        }
    }

    int result = 0;
    while (!stk.empty()) { result += stk.top(); stk.pop(); }
    return result;
}
```

---

## Design Insight: Why Recursive Descent?

| Approach | Handles Precedence | Handles Parentheses | Code Complexity |
|----------|-------------------|--------------------|----|
| Stack-based | Yes (with discipline) | Yes | Medium |
| Recursive descent | ✅ Naturally via depth | ✅ Naturally via recursion | Low |
| Shunting-yard | Yes | Yes | High (two stacks) |

Recursive descent maps **grammar → code** 1:1. Adding a new operator means adding a new level of recursion.

---

## Related Files

- [Expression Evaluator](./Expression%20Evaluator.md)
- [Backtracking](../Patterns/Backtracking.md)

---

**Back:** [Recursion README](../README.md)

> **Last Updated:** 2026-06-26
