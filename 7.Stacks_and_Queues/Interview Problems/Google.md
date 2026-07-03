# Google — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Google
> [← Google OA](../OA-Qns/Google.md)

---

## Problem 1: Maximum Frequency Stack — Deep Dive

**LC 895** · Hard · O(1) push, O(1) pop

### Full Solution

```rust
use std::collections::HashMap;

struct FreqStack {
    freq: HashMap<i32, i32>,
    group: HashMap<i32, Vec<i32>>,
    max_freq: i32,
}

impl FreqStack {
    fn new() -> Self {
        FreqStack {
            freq: HashMap::new(),
            group: HashMap::new(),
            max_freq: 0,
        }
    }

    fn push(&mut self, val: i32) {
        let f = {
            let entry = self.freq.entry(val).or_insert(0);
            *entry += 1;
            *entry
        };
        self.max_freq = self.max_freq.max(f);
        self.group.entry(f).or_insert_with(Vec::new).push(val);
    }

    fn pop(&mut self) -> i32 {
        let stk = self.group.get_mut(&self.max_freq).unwrap();
        let val = stk.pop().unwrap();
        *self.freq.get_mut(&val).unwrap() -= 1;  // freq[val]--
        if stk.is_empty() {
            self.group.remove(&self.max_freq);
            self.max_freq -= 1;
        }
        val
    }
}
```

**Q: Why does `maxFreq` always decrement by exactly 1 (never skip)?**
A: When `group[maxFreq]` becomes empty, `maxFreq - 1` is guaranteed non-empty. Here's why: the element we just popped was in `group[maxFreq]`. Before it reached frequency `maxFreq`, it was in `group[maxFreq - 1]`. Other elements that were in `group[maxFreq - 1]` haven't been popped yet (we always pop from `maxFreq`), so `group[maxFreq - 1]` still has elements.

**Q: What if we need to also support `top()` (peek without pop)?**
A: `return group[maxFreq].top()` — O(1).

**Q: Design challenge: What if capacity limit N is added?**
A: On `push`, if `freq.size() == N` and `val` is not in `freq`: evict the element at `group[1]` (lowest frequency, and if tie, oldest = bottom of that group's stack). Then insert new element. This turns it into an LFU cache with stack-based tie-breaking.

---

## Problem 2: Basic Calculator (All Variants)

Google often asks you to solve all three variants in sequence within a single interview.

### LC 227: Basic Calculator II (`+`, `-`, `*`, `/`, no parens)

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut num = 0i32;
    let mut sign = '+';
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    for i in 0..n {
        let c = chars[i];
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        }
        if (!c.is_ascii_digit() && c != ' ') || i == n - 1 {
            if sign == '+' { stk.push(num); }
            else if sign == '-' { stk.push(-num); }
            else if sign == '*' { let t = stk.pop().unwrap(); stk.push(t * num); }
            else if sign == '/' { let t = stk.pop().unwrap(); stk.push(t / num); }
            sign = c;
            num = 0;
        }
    }
    stk.iter().sum()
}
```

### LC 224: Basic Calculator (`+`, `-`, parens, no `*`/`/`)

```rust
fn calculate(s: String) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut result = 0i32;
    let mut num = 0i32;
    let mut sign = 1i32;
    for c in s.chars() {
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        } else if c == '+' {
            result += sign * num; num = 0; sign = 1;
        } else if c == '-' {
            result += sign * num; num = 0; sign = -1;
        } else if c == '(' {
            stk.push(result); stk.push(sign); result = 0; sign = 1;
        } else if c == ')' {
            result += sign * num; num = 0;
            let s1 = stk.pop().unwrap();
            let s2 = stk.pop().unwrap();
            result = result * s1 + s2;
        }
    }
    result + sign * num
}
```

### LC 772: Basic Calculator III (all operators + parens)

Combine both: when encountering `(`, push current accumulated result and sign; apply `*/` immediately via stack; on `)`, flush and restore.

```rust
fn parse(s: &[char], idx: &mut usize) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut num = 0i32;
    let mut sign = '+';
    while *idx < s.len() {
        let c = s[*idx];
        *idx += 1;
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        } else if c == '(' {
            num = parse(s, idx); // recurse for subexpr
        }
        if (!c.is_ascii_digit() && c != ' ') || *idx == s.len() {
            if sign == '+' { stk.push(num); }
            else if sign == '-' { stk.push(-num); }
            else if sign == '*' { let t = stk.pop().unwrap(); stk.push(t * num); }
            else if sign == '/' { let t = stk.pop().unwrap(); stk.push(t / num); }
            sign = c;
            num = 0;
            if c == ')' { break; } // return from recursion
        }
    }
    stk.iter().sum()
}

fn calculate(s: String) -> i32 {
    // Recursive descent approach (Google prefers this at L5+)
    let chars: Vec<char> = s.chars().collect();
    let mut idx = 0;
    parse(&chars, &mut idx)
}
```

**Q: How does the recursive approach handle nested parentheses?**
A: On `(`, we recurse with `idx` as a shared mutable reference so both levels advance through the same string. On `)`, the inner call breaks out of its loop and returns. The outer call receives the inner expression's value as `num` and continues.

---

## Problem 3: Number of Visible People in Queue — Google L4

**LC 1944** · Hard · O(n) time

`heights[i]` = height of person i. Person `i` can see person `j` (i < j) if everyone between them is shorter than both `heights[i]` and `heights[j]`.

```rust
fn can_see_persons_count(heights: Vec<i32>) -> Vec<i32> {
    let n = heights.len();
    let mut result = vec![0i32; n];
    let mut stk: Vec<i32> = Vec::new(); // decreasing stack of heights

    for i in (0..n).rev() {
        let mut count = 0;
        // Count how many people to the right are visible
        while !stk.is_empty() && *stk.last().unwrap() < heights[i] {
            stk.pop();
            count += 1;
        }
        // The first person not popped (taller or equal) is also visible if it exists
        if !stk.is_empty() { count += 1; }
        result[i] = count;
        stk.push(heights[i]);
    }
    result
}
```

**Q: Why scan right-to-left?**
A: We want to know what person `i` can see to their right. Scanning right-to-left, the stack represents people seen so far (to the right). People shorter than `heights[i]` are visible (and are popped). The first person taller than `heights[i]` is also visible (but blocks everyone behind it).

**Q: Why are popped people visible to person i?**
A: Each popped person is shorter than `heights[i]`. Since they were on the decreasing stack, no one between them and `i` is taller than them — so `i` can see over all intervening people to reach them.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | Valid Parens, Min Stack, Daily Temperatures |
| L4 | Largest Rectangle, Basic Calc, Sliding Window Max |
| L5+ | Max Freq Stack, Number of Visible People, Basic Calc III (recursive) |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [Maximum Frequency Stack Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
