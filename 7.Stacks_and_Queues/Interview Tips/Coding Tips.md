# Coding Tips — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Always Use `VecDeque`, Not `Vec` Alone

```rust
use std::collections::VecDeque;

// Simple LIFO — Vec<i32> as stack (limited to top/end access only)
let mut stk: Vec<i32> = Vec::new();
stk.push(x);      // push to top (end)
stk.pop();        // remove from top (end)
stk.last();       // view top (returns Option<&i32>)

// FLEXIBLE — VecDeque<i32> supports both ends
let mut dq: VecDeque<i32> = VecDeque::new();
dq.push_front(x);  // push to front (top)
dq.pop_front();    // remove from front (top)
dq.front();        // view front (top)
```

`Vec<T>` used as a stack has only top (end) access. `VecDeque<T>` is more flexible and supports bidirectional access, making it the preferred choice when you need explicit front/back control.

---

## 2. Monotonic Stack — Store Indices, Not Values

```rust
// BAD — can't compute distances or write back to result array
let mut stk: Vec<i32> = Vec::new();
stk.push(nums[i]);   // pushing value

// GOOD
let mut stk: Vec<usize> = Vec::new();
stk.push(i);         // pushing index
// Then: nums[*stk.last().unwrap()] gives value, stk.last() gives position
```

Reason: Distance-based answers (Daily Temperatures: `i - prevIdx`) and writing answers back to result arrays both require the index.

---

## 3. Monotonic Deque — Use `push_back/pop_back` and `push_front/pop_front`

```rust
use std::collections::VecDeque;

// For sliding window maximum (decreasing deque):
let mut dq: VecDeque<usize> = VecDeque::new();
// Add to back (maintain decreasing order from front to back)
while !dq.is_empty() && nums[*dq.back().unwrap()] <= nums[i] { dq.pop_back(); }
dq.push_back(i);
// Get max from front
let max_val = nums[*dq.front().unwrap()];
// Expire old window from front
if *dq.front().unwrap() <= i - k { dq.pop_front(); }
```

Never mix `push_front/pop_front` (stack style) with `push_back/pop_front` (queue style) haphazardly. Be explicit about which end you're accessing.

---

## 4. Histogram Width Formula

```rust
while !stk.is_empty() && heights[*stk.last().unwrap()] > h {
    let height = heights[stk.pop().unwrap()];
    // LEFT boundary is exclusive: stk.last() + 1
    // RIGHT boundary is exclusive: i (current index)
    let width = if stk.is_empty() { i } else { i - stk.last().unwrap() - 1 };
    max_area = max_area.max(height * width);
}
```

Mnemonic: "Width is the gap between the two shorter walls." `i` is the right wall, `stk.last()` is the left wall. Width = `right - left - 1` (excluding both walls).

---

## 5. Basic Calculator — The `sign` Variable Pattern

```rust
let mut num: i32 = 0;
let mut sign = '+';   // sign of the UPCOMING number, not current

// When we see an operator or reach end of string:
// 1. Apply 'sign' to 'num' (process the number we just built)
// 2. Update 'sign' = current operator
// 3. Reset num = 0
```

This lookahead trick works because in `"3+5*2"`, when we see `+` we're done reading `3`. The sign `+` applies to the next operand `5` (we don't know yet).

---

## 6. Decode String — Multi-Digit Counts

```rust
if c.is_ascii_digit() {
    k = k * 10 + (c as i32 - '0' as i32);  // accumulate: "12[" → k = 1*10+2 = 12
}
```

Easy to forget when writing the solution quickly. `"12[a]"` → `"aaaaaaaaaaaa"` (12 a's). Without the `k * 10`, you'd get `k = 2`.

---

## 7. Queue via Two Stacks — Transfer Only When Needed

```rust
fn transfer(in_stack: &mut Vec<i32>, out_stack: &mut Vec<i32>) {
    if out_stack.is_empty() {    // only transfer when out_stack is EMPTY
        while let Some(val) = in_stack.pop() {
            out_stack.push(val);
        }
    }
}
```

Partial transfer (when outStack has some items) would break FIFO ordering. Transfer only when `out_stack` is completely drained.

---

## 8. Max Frequency Stack — Entry API Pattern

```rust
group.entry(f).or_insert_with(Vec::new).push(val);  // entry() default-inserts if key absent
```

Equivalent to:
```rust
if !group.contains_key(&f) { group.insert(f, Vec::new()); }
if let Some(stack) = group.get_mut(&f) { stack.push(val); }
```

The `group.entry(f).or_insert_with(Vec::new).push(val)` version is idiomatic Rust — `entry()` inserts a default value for missing keys automatically, avoiding the double-lookup.

---

## 9. Circular Queue — `size` Variable vs One Slot Wasted

```rust
// Option A (recommended): Track size separately
if self.is_full() { return false; }  // size == capacity
self.tail = (self.tail + 1) % self.capacity;
self.data[self.tail] = value;
self.size += 1;

// Option B: One slot wasted (capacity+1 slots for N-element queue)
// is_full = (self.tail + 1) % self.capacity == self.head
// This wastes one slot but avoids a separate size counter
```

Option A with `size` is cleaner and avoids the off-by-one confusion inherent in Option B.

---

## 10. Sentinel in Histogram

```rust
for i in 0..=n {
    let h = if i == n { 0 } else { heights[i] };   // sentinel height = 0
    // ...
}
```

The sentinel forces all remaining stack entries to flush after the loop. Forgetting this is the most common bug in histogram problems — elements that are never smaller than their right neighbor stay in the stack forever.

---

## Quick Reference: Method Names

| Operation | Vec\<i32\> (stack) | VecDeque\<i32\> (queue) | VecDeque\<i32\> (explicit) |
|-----------|-------------|-------------|------------------------|
| Add top/front | `push(x)` | `push_back(x)` | `push_front(x)` |
| Add bottom/back | — | — | `push_back(x)` |
| Remove top/front | `pop()` | `pop_front()` | `pop_front()` |
| Remove back | — | — | `pop_back()` |
| View top/front | `last()` | `front()` | `front()` |
| View back | — | — | `back()` |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)
