# Microsoft — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Microsoft
> [← Microsoft OA](../OA-Qns/Microsoft.md)

---

## Problem 1: Implement Queue using Stacks — Design Deep Dive

**LC 232** · Easy · Amortized O(1)

```rust
struct MyQueue {
    in_stack: Vec<i32>,
    out_stack: Vec<i32>,
}

impl MyQueue {
    fn new() -> Self {
        MyQueue {
            in_stack: Vec::new(),
            out_stack: Vec::new(),
        }
    }

    fn transfer(&mut self) {
        if self.out_stack.is_empty() {
            while let Some(val) = self.in_stack.pop() {
                self.out_stack.push(val);
            }
        }
    }

    fn push(&mut self, x: i32) {
        self.in_stack.push(x);
    }

    fn pop(&mut self) -> i32 {
        self.transfer();
        self.out_stack.pop().unwrap()
    }

    fn peek(&mut self) -> i32 {
        self.transfer();
        *self.out_stack.last().unwrap()
    }

    fn empty(&self) -> bool {
        self.in_stack.is_empty() && self.out_stack.is_empty()
    }
}
```

**Q: What is the amortized time complexity?**
A: O(1) amortized. Each element moves from `in_stack` to `out_stack` exactly once. So over N operations, total work = O(N). Worst case per single `pop` is O(N) (when transfer occurs), but that can only happen after N pushes, spreading the O(N) cost across N previous operations.

**Q: What if we need peek() and pop() to always be O(1) worst case?**
A: Not possible with only stacks. You'd need to transfer on every push instead — O(N) push, O(1) pop/peek. There's a fundamental tradeoff: you can make either push or pop O(1), but not both worst-case.

**Q: Microsoft follow-up — extend to thread-safe queue.**

```rust
use std::sync::Mutex;

struct ThreadSafeQueue {
    inner: Mutex<(Vec<i32>, Vec<i32>)>, // (in_stack, out_stack)
}

impl ThreadSafeQueue {
    fn new() -> Self {
        ThreadSafeQueue {
            inner: Mutex::new((Vec::new(), Vec::new())),
        }
    }

    fn push(&self, x: i32) {
        let mut guard = self.inner.lock().unwrap();
        guard.0.push(x);
    }

    fn pop(&self) -> i32 {
        let mut guard = self.inner.lock().unwrap();
        if guard.1.is_empty() {
            while let Some(val) = guard.0.pop() {
                guard.1.push(val);
            }
        }
        guard.1.pop().unwrap()
    }
}
```

---

## Problem 2: Basic Calculator II — Microsoft Style

**LC 227** · Medium

Microsoft often asks this in a "live coding" format with follow-ups about edge cases.

```rust
fn calculate(s: &str) -> i32 {
    let mut stk: Vec<i32> = Vec::new();
    let mut num = 0i32;
    let mut sign = '+';
    let bytes = s.as_bytes();
    let n = bytes.len();
    for i in 0..n {
        let c = bytes[i] as char;
        if c.is_ascii_digit() {
            num = num * 10 + (c as i32 - '0' as i32);
        }
        if (!c.is_ascii_digit() && c != ' ') || i == n - 1 {
            match sign {
                '+' => stk.push(num),
                '-' => stk.push(-num),
                '*' => { let top = stk.pop().unwrap(); stk.push(top * num); }
                '/' => { let top = stk.pop().unwrap(); stk.push(top / num); }
                _ => {}
            }
            sign = c;
            num = 0;
        }
    }
    stk.iter().sum()
}
```

**Microsoft Edge Case checklist:**
- Spaces between tokens: `"3 + 2"` — handled by `c != ' '` skip
- Multi-digit numbers: `"14+2"` — handled by `num = num * 10 + digit`
- Negative results: `"1-2"` — handled by pushing `-num` for `-` sign
- Division truncation: Rust integer division truncates toward zero by default ✓
- Single number: `"5"` — final `i == n-1` condition handles last number

---

## Problem 3: Trapping Rain Water — Microsoft Interview Discussion

**LC 42** · Hard

Microsoft interviewers focus on code quality and approach justification.

**Preferred approach for Microsoft interview:**

```rust
fn trap(height: &[i32]) -> i32 {
    let (mut left, mut right) = (0usize, height.len() - 1);
    let (mut max_left, mut max_right, mut water) = (0, 0, 0);
    while left < right {
        if height[left] <= height[right] {
            if height[left] >= max_left {
                max_left = height[left];
            } else {
                water += max_left - height[left];
            }
            left += 1;
        } else {
            if height[right] >= max_right {
                max_right = height[right];
            } else {
                water += max_right - height[right];
            }
            right -= 1;
        }
    }
    water
}
```

**How to walk through this at Microsoft:**
1. "The key insight is that water at position `x` = `min(maxLeft, maxRight) - height[x]`."
2. "If `height[left] ≤ height[right]`, the right side is guaranteed to be the larger boundary, so we can compute water for the left side using just `maxLeft`."
3. "This gives us O(1) space, O(n) time — optimal."

**Q: What if the array contains negative heights?**
A: Problem constraints guarantee non-negative heights. But if it could, negative heights would still work — water calculation uses subtraction, which would be negative (meaning no water trapped there).

---

## Microsoft Interview Style Notes

- Microsoft prioritizes code readability and correctness over cleverness.
- Walk through your approach verbally before coding — they want to see your thought process.
- Common follow-up after any stack problem: "How would you make this thread-safe?"
- For design problems (Min Stack, Circular Queue), mention invariants explicitly: "My invariant is that `out_stack` always has elements in FIFO order."

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Queue & Deque Pattern](../Patterns/Queue%20and%20Deque.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
