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

```java
class Solution {
    private String s;
    private int pos;

    public int calculate(String s) {
        this.s = s;
        this.pos = 0;
        return parseExpr();
    }

    // expr → term (('+' | '-') term)*
    private int parseExpr() {
        int result = parseTerm();
        while (pos < s.length() && (s.charAt(pos) == '+' || s.charAt(pos) == '-')) {
            char op = s.charAt(pos++);
            int term = parseTerm();
            result = (op == '+') ? result + term : result - term;
        }
        return result;
    }

    // term → factor (('*' | '/') factor)*
    private int parseTerm() {
        int result = parseFactor();
        while (pos < s.length() && (s.charAt(pos) == '*' || s.charAt(pos) == '/')) {
            char op = s.charAt(pos++);
            int factor = parseFactor();
            result = (op == '*') ? result * factor : result / factor;
        }
        return result;
    }

    // factor → number | '(' expr ')'
    private int parseFactor() {
        skipSpaces();
        if (pos < s.length() && s.charAt(pos) == '(') {
            pos++; // consume '('
            int val = parseExpr();
            pos++; // consume ')'
            return val;
        }
        return parseNumber();
    }

    private int parseNumber() {
        skipSpaces();
        int sign = 1;
        if (pos < s.length() && s.charAt(pos) == '-') { sign = -1; pos++; }
        int num = 0;
        while (pos < s.length() && Character.isDigit(s.charAt(pos))) {
            num = num * 10 + (s.charAt(pos++) - '0');
        }
        skipSpaces();
        return sign * num;
    }

    private void skipSpaces() {
        while (pos < s.length() && s.charAt(pos) == ' ') pos++;
    }
}
```

---

## Basic Calculator (LC 224) — Only +, -, parentheses

Simpler: only addition/subtraction, no precedence levels. Use a stack to handle parentheses.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int result = 0, num = 0, sign = 1;

    for (char c : s.toCharArray()) {
        if (Character.isDigit(c)) {
            num = num * 10 + (c - '0');
        } else if (c == '+') {
            result += sign * num; num = 0; sign = 1;
        } else if (c == '-') {
            result += sign * num; num = 0; sign = -1;
        } else if (c == '(') {
            // Save current result and sign; start fresh
            stack.push(result);
            stack.push(sign);
            result = 0; sign = 1;
        } else if (c == ')') {
            result += sign * num; num = 0;
            result *= stack.pop();   // multiply by sign before '('
            result += stack.pop();   // add result before '('
        }
    }
    return result + sign * num;
}
```

---

## Basic Calculator II (LC 227) — +, -, *, / (no parentheses)

Use a stack: push positive/negative for `+`/`-`, and apply `*`/`/` immediately.

```java
public int calculate(String s) {
    Deque<Integer> stack = new ArrayDeque<>();
    int num = 0;
    char op = '+';

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (Character.isDigit(c)) num = num * 10 + (c - '0');
        if ((!Character.isDigit(c) && c != ' ') || i == s.length() - 1) {
            if      (op == '+') stack.push(num);
            else if (op == '-') stack.push(-num);
            else if (op == '*') stack.push(stack.pop() * num);
            else if (op == '/') stack.push(stack.pop() / num);
            op = c;
            num = 0;
        }
    }

    int result = 0;
    while (!stack.isEmpty()) result += stack.pop();
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
