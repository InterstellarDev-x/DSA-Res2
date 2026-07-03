# Common Mistakes — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Mistake 1: Using `Vec` as Stack Instead of `VecDeque`

```rust
use std::collections::VecDeque;

// BAD
let mut stk: Vec<i32> = Vec::new();
stk.push(1);

// GOOD
let mut stk: VecDeque<i32> = VecDeque::new();
stk.push_front(1);
```

`Vec<T>` used as a stack only exposes one end and does not support double-ended access. `VecDeque<T>` provides direct access to both ends and does not expose unintended access patterns. `VecDeque<T>` is the correct choice when you need a double-ended structure.

---

## Mistake 2: Pushing Values Instead of Indices in Monotonic Stack

```rust
// BAD — for Daily Temperatures
while !stk.is_empty() && nums[*stk.last().unwrap()] < nums[i] {
    let prev_idx = stk.pop().unwrap();
    result[prev_idx] = i - prev_idx;  // ERROR: stk.last() returns the VALUE, can't do i - value
}

// GOOD
while !stk.is_empty() && temperatures[*stk.last().unwrap()] < temperatures[i] {
    let prev_idx = stk.pop().unwrap();
    result[prev_idx] = i - prev_idx;  // prev_idx is an INDEX, subtraction gives days
}
stk.push(i);  // push index, not temperatures[i]
```

If you push values, you can't compute distances or write back to the result array by position.

---

## Mistake 3: Histogram Width Formula Off-By-One

```rust
// BAD — wrong width
let width = i - stk.last().unwrap();           // off by 1 (includes left wall)

// GOOD
let width = if stk.is_empty() { i } else { i - stk.last().unwrap() - 1 };

// Explanation:
// - Right boundary (exclusive): i (heights[i] is shorter, so rectangle ends before i)
// - Left boundary (exclusive): stk.last() (this is the next-shorter bar to the left)
// - Width: (i-1) - (stk.last()+1) + 1 = i - stk.last() - 1
```

The `-1` is the most common off-by-one in this problem. Verify with a 1-element array: `heights=[5]`, at sentinel `i=1`, stk.pop()=0, stk is empty, width=1 ✓.

---

## Mistake 4: Forgetting the Histogram Sentinel

```rust
// BAD — bars that are never smaller than their right neighbor stay in stack
for i in 0..n {
    while !stk.is_empty() && heights[*stk.last().unwrap()] > heights[i] {
        // compute area
    }
    stk.push(i);
}
// Stack still has entries! Never flushed for bars like [3, 5, 7]

// GOOD
for i in 0..=n {                                          // ..=n, not ..n
    let h = if i == n { 0 } else { heights[i] };         // height 0 flushes all remaining
    // ...
}
```

For input `[3, 5, 7]` (monotonically increasing), no pops occur in the main loop. The sentinel forces all three to be popped and their areas computed.

---

## Mistake 5: Wrong Transfer Order in Queue via Two Stacks

```rust
// BAD — partial transfer breaks FIFO
fn transfer(in_stack: &mut Vec<i32>, out_stack: &mut Vec<i32>) {
    while let Some(val) = in_stack.pop() { out_stack.push(val); }  // always transfers
}

// GOOD — only transfer when out_stack is completely empty
fn transfer(in_stack: &mut Vec<i32>, out_stack: &mut Vec<i32>) {
    if out_stack.is_empty() {
        while let Some(val) = in_stack.pop() { out_stack.push(val); }
    }
}
```

If `out_stack` has `[3,2,1]` (1 on top = dequeue next) and you transfer `in_stack=[6,5,4]` (4 on top), you'd get `out_stack=[4,5,6,3,2,1]` with 4 as the next to dequeue — wrong order.

---

## Mistake 6: Circular Queue — Forgetting `size` or Using Wrong Condition

```rust
// BAD — ambiguous: head==tail could mean empty OR full
fn is_empty(&self) -> bool { self.head == self.tail }
fn is_full(&self) -> bool  { self.head == self.tail }  // same condition!

// GOOD — track size separately
// In the struct: size: usize
fn is_empty(&self) -> bool { self.size == 0 }
fn is_full(&self) -> bool  { self.size == self.capacity }
```

Without a size counter (or a `is_full` boolean flag), you can't distinguish an empty circular buffer from a full one when `head == tail`.

---

## Mistake 7: Basic Calculator — Forgetting the Last Number

```rust
// BAD — misses the last token (no operator after it)
for (i, c) in s.chars().enumerate() {
    if c.is_ascii_digit() { num = num * 10 + (c as i32 - '0' as i32); }
    if !c.is_ascii_digit() && c != ' ' {  // WRONG: only processes on operator
        // apply sign to num
    }
}
// Last number never gets processed!

// GOOD — process on operator OR at end of string
if (!c.is_ascii_digit() && c != ' ') || i == s.len() - 1 {
    // apply sign to num
}
```

The input `"3+5"` ends with a digit. Without the `i == s.len() - 1` condition, `5` is never pushed to the stack.

---

## Mistake 8: Decode String — Not Resetting `k` After `[`

```rust
// BAD
} else if c == '[' {
    count_stack.push(k);
    str_stack.push(current.clone());
    current = String::new();
    // MISSING: k = 0
}

// GOOD
} else if c == '[' {
    count_stack.push(k);
    str_stack.push(current.clone());
    k = 0;                    // reset for the next number
    current = String::new();
}
```

Without resetting `k`, nested brackets like `"2[3[a]]"` would use `k=3` again for the outer bracket instead of finding `k=2`.

---

## Mistake 9: Max Frequency Stack — `max_freq` Decrement Condition

```rust
use std::collections::HashMap;

// BAD — always decrement max_freq on pop
fn pop(&mut self) -> i32 {
    let val = self.group.get_mut(&self.max_freq).unwrap().pop().unwrap();
    *self.freq.get_mut(&val).unwrap() -= 1;
    self.max_freq -= 1;   // WRONG: only decrement if group[max_freq] is now empty
    val
}

// GOOD
fn pop(&mut self) -> i32 {
    let val = self.group.get_mut(&self.max_freq).unwrap().pop().unwrap();
    *self.freq.get_mut(&val).unwrap() -= 1;
    if self.group.get(&self.max_freq).map_or(true, |s| s.is_empty()) {
        self.group.remove(&self.max_freq);
        self.max_freq -= 1;   // only decrement when bucket is empty
    }
    val
}
```

After popping from `group[3]`, if `group[3]` still has elements, `max_freq` should stay 3. Decrementing unconditionally would skip valid elements.

---

## Mistake 10: 132 Pattern — Confusing Which Pointer Is Which

The pattern requires three pointers: `nums[i] < nums[k] < nums[j]` where `i < j < k`.
- `stk` = candidates for `nums[j]` (the "3" in 132 — the largest)
- `third` = best candidate for `nums[k]` (the "2" in 132 — middle value)
- `nums[i]` = current element being checked as the smallest ("1")

```rust
// BAD — checking wrong condition
if nums[i] > third { return true; }   // WRONG: we want nums[i] < third

// GOOD
if nums[i] < third { return true; }   // nums[i] is "1", must be LESS than "2" (third)
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
