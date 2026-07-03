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

```rust
struct Solution {
    s: Vec<char>,
    pos: usize,
}

impl Solution {
    fn new(s: &str) -> Self {
        Solution {
            s: s.chars().collect(),
            pos: 0,
        }
    }

    // expr → term (('+' | '-') term)*
    fn parse_expr(&mut self) -> i32 {
        let mut result = self.parse_term();
        while self.pos < self.s.len() && (self.s[self.pos] == '+' || self.s[self.pos] == '-') {
            let op = self.s[self.pos];
            self.pos += 1;
            let term = self.parse_term();
            result = if op == '+' { result + term } else { result - term };
        }
        result
    }

    // term → factor (('*' | '/') factor)*
    fn parse_term(&mut self) -> i32 {
        let mut result = self.parse_factor();
        while self.pos < self.s.len() && (self.s[self.pos] == '*' || self.s[self.pos] == '/') {
            let op = self.s[self.pos];
            self.pos += 1;
            let factor = self.parse_factor();
            result = if op == '*' { result * factor } else { result / factor };
        }
        result
    }

    // factor → number | '(' expr ')'
    fn parse_factor(&mut self) -> i32 {
        self.skip_spaces();
        if self.pos < self.s.len() && self.s[self.pos] == '(' {
            self.pos += 1; // consume '('
            let val = self.parse_expr();
            self.pos += 1; // consume ')'
            return val;
        }
        self.parse_number()
    }

    fn parse_number(&mut self) -> i32 {
        self.skip_spaces();
        let mut sign = 1;
        if self.pos < self.s.len() && self.s[self.pos] == '-' {
            sign = -1;
            self.pos += 1;
        }
        let mut num = 0;
        while self.pos < self.s.len() && self.s[self.pos].is_ascii_digit() {
            num = num * 10 + (self.s[self.pos] as i32 - '0' as i32);
            self.pos += 1;
        }
        self.skip_spaces();
        sign * num
    }

    fn skip_spaces(&mut self) {
        while self.pos < self.s.len() && self.s[self.pos] == ' ' {
            self.pos += 1;
        }
    }

    pub fn calculate(s: String) -> i32 {
        let mut parser = Solution::new(&s);
        parser.parse_expr()
    }
}
```

---

## Basic Calculator (LC 224) — Only +, -, parentheses

Simpler: only addition/subtraction, no precedence levels. Use a stack to handle parentheses.

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut result = 0;
    let mut num = 0;
    let mut sign = 1;

    for c in s.chars() {
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        } else if c == '+' {
            result += sign * num; num = 0; sign = 1;
        } else if c == '-' {
            result += sign * num; num = 0; sign = -1;
        } else if c == '(' {
            // Save current result and sign; start fresh
            stk.push(result);
            stk.push(sign);
            result = 0; sign = 1;
        } else if c == ')' {
            result += sign * num; num = 0;
            let top_sign = stk.pop().unwrap();
            result *= top_sign;   // multiply by sign before '('
            let prev_result = stk.pop().unwrap();
            result += prev_result;   // add result before '('
        }
    }
    result + sign * num
}
```

---

## Basic Calculator II (LC 227) — +, -, *, / (no parentheses)

Use a stack: push positive/negative for `+`/`-`, and apply `*`/`/` immediately.

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut num = 0i32;
    let mut op = '+';
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();

    for i in 0..n {
        let c = chars[i];
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        }
        if (!c.is_ascii_digit() && c != ' ') || i == n - 1 {
            if op == '+' { stk.push(num); }
            else if op == '-' { stk.push(-num); }
            else if op == '*' {
                let top = stk.pop().unwrap();
                stk.push(top * num);
            } else if op == '/' {
                let top = stk.pop().unwrap();
                stk.push(top / num);
            }
            op = c;
            num = 0;
        }
    }

    let mut result = 0;
    while let Some(top) = stk.pop() { result += top; }
    result
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
