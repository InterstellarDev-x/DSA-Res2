# Monotonic Stack

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 3 of 5**
> **Problems:** NGE I & II · Daily Temperatures · Online Stock Span · 132 Pattern · Asteroid Collision · Remove K Digits · Trapping Rain Water · Sum of Subarray Minimums · Remove Duplicate Letters · Maximum Width Ramp · Number of Visible People

---

## Core Concept

A **monotonic stack** maintains elements in a strictly increasing or decreasing order at all times. When a new element violates the invariant, pop until invariant holds. The moment of popping is when answers are computed.

**Invariant types:**
- **Decreasing stack:** each new element pops all smaller elements → popped element's "Next Greater Element" is the new element
- **Increasing stack:** each new element pops all larger elements → popped element's "Next Smaller Element" is the new element

---

## Template 1: Next Greater Element (NGE)

```rust
// For each element, find the first element to its RIGHT that is greater
fn next_greater_elements(nums: &[i32]) -> Vec<i32> {
    let n = nums.len();
    let mut result = vec![-1; n];          // default: no NGE found
    let mut stk: Vec<usize> = Vec::new();  // stores INDICES

    for i in 0..n {
        // Pop all elements smaller than nums[i] — nums[i] is their NGE
        while !stk.is_empty() && nums[*stk.last().unwrap()] < nums[i] {
            let top = stk.pop().unwrap();
            result[top] = nums[i];
        }
        stk.push(i);
    }
    result
    // Remaining indices in stack have no NGE → stay -1
}
```

**Why store indices?** We need to write the answer back to `result[index]`. Values alone are insufficient.

---

## Problem 1: Next Greater Element I — LC 496

Two arrays: find NGE of each element of `nums1` within `nums2`.

```rust
use std::collections::HashMap;

fn next_greater_element(nums1: &[i32], nums2: &[i32]) -> Vec<i32> {
    let mut nge: HashMap<i32, i32> = HashMap::new();
    let mut stk: Vec<i32> = Vec::new();

    for &val in nums2 {
        while !stk.is_empty() && *stk.last().unwrap() < val {
            let top = stk.pop().unwrap();
            nge.insert(top, val);   // store value→NGE mapping
        }
        stk.push(val);
    }

    nums1.iter().map(|&x| *nge.get(&x).unwrap_or(&-1)).collect()
}
```

**Complexity:** O(n + m) time, O(n) space

---

## Problem 2: Daily Temperatures — LC 739

Find, for each day, how many days until a warmer temperature.

```rust
fn daily_temperatures(temperatures: &[i32]) -> Vec<i32> {
    let n = temperatures.len();
    let mut result = vec![0; n];
    let mut stk: Vec<usize> = Vec::new();  // indices

    for i in 0..n {
        while !stk.is_empty() && temperatures[*stk.last().unwrap()] < temperatures[i] {
            let prev_idx = stk.pop().unwrap();
            result[prev_idx] = (i - prev_idx) as i32;   // days to wait = index difference
        }
        stk.push(i);
    }
    result
}
```

**Key insight:** `i - prev_idx` gives the distance (days to wait). This only works because we store indices, not temperatures.

---

## Problem 3: Next Greater Element II (Circular) — LC 503

Array is circular — wrap around once.

```rust
fn next_greater_elements(nums: &[i32]) -> Vec<i32> {
    let n = nums.len();
    let mut result = vec![-1; n];
    let mut stk: Vec<usize> = Vec::new();

    // Loop twice to simulate circular array
    for i in 0..2 * n {
        let idx = i % n;
        while !stk.is_empty() && nums[*stk.last().unwrap()] < nums[idx] {
            let top = stk.pop().unwrap();
            result[top] = nums[idx];
        }
        if i < n { stk.push(idx); }   // only push indices from first pass
    }
    result
}
```

**Why `if i < n` for push?** During the second pass, we only need to resolve remaining stack elements — we don't push new indices again (they're already in result or will be resolved).

---

## Problem 4: Online Stock Span — LC 901

Design: `next(price)` returns the number of consecutive days (including today) for which today's price ≥ the price on those days.

```rust
struct StockSpanner {
    // Stack stores (price, span) pairs
    stk: Vec<(i32, i32)>,
}

impl StockSpanner {
    fn new() -> Self {
        StockSpanner { stk: Vec::new() }
    }

    fn next(&mut self, price: i32) -> i32 {
        let mut span = 1;
        // Absorb all previous days where price was ≤ current price
        while !self.stk.is_empty() && self.stk.last().unwrap().0 <= price {
            span += self.stk.pop().unwrap().1;
        }
        self.stk.push((price, span));
        span
    }
}
```

**Key trick:** Store cumulative span. When absorbing a previous day, we don't need to re-examine what it absorbed — its stored span already accounts for those days. This achieves O(1) amortized.

**Trace:** prices = [100, 80, 60, 70, 60, 75, 85]
```
100: stack=[(100,1)], span=1
80:  stack=[(100,1),(80,1)], span=1
60:  stack=[(100,1),(80,1),(60,1)], span=1
70:  pop (60,1)→span=1+1=2. stack=[(100,1),(80,1),(70,2)], span=2
60:  stack=[(100,1),(80,1),(70,2),(60,1)], span=1
75:  pop (60,1)→3, pop (70,2)→5. stack=[(100,1),(80,1),(75,5)], span=5
85:  pop (75,5)→6, pop (80,1)→7. stack=[(100,1),(85,7)], span=7
```

---

## Problem 5: 132 Pattern — LC 456

Find i < j < k such that `nums[i] < nums[k] < nums[j]`.

**Approach:** Traverse from right to left with a decreasing monotonic stack.
- `third` = the best candidate for `nums[k]` (largest value seen below a "mountain top")
- Stack maintains potential `nums[j]` values (decreasing)

```rust
fn find132pattern(nums: &[i32]) -> bool {
    let n = nums.len();
    let mut third = i32::MIN;            // best candidate for nums[k]
    let mut stk: Vec<i32> = Vec::new();  // candidates for nums[j]

    for i in (0..n).rev() {
        if nums[i] < third { return true; }    // nums[i] is nums[1] (smallest)
        while !stk.is_empty() && *stk.last().unwrap() < nums[i] {
            third = stk.pop().unwrap();   // this is nums[k] — it was below nums[j]=nums[i]
        }
        stk.push(nums[i]);
    }
    false
}
```

**Why right-to-left?** We want to find a smaller element to the left of a "valley above the minimum." Going right-to-left, whenever we see a larger element than the stack top, the popped element becomes the best `k` candidate.

---

## Problem 6: Asteroid Collision — LC 735

Asteroids: positive = moving right, negative = moving left. Collision: right-moving hits left-moving. Larger absolute value survives; equal sizes both explode.

```rust
fn asteroid_collision(asteroids: Vec<i32>) -> Vec<i32> {
    let mut stk: Vec<i32> = Vec::new();
    for a in asteroids {
        let mut exploded = false;
        while !stk.is_empty() && a < 0 && *stk.last().unwrap() > 0 {
            if *stk.last().unwrap() < -a {
                stk.pop();   // right-moving asteroid explodes
                continue;
            } else if *stk.last().unwrap() == -a {
                stk.pop();   // both explode
            }
            // stk.last() > -a: new asteroid explodes
            exploded = true;
            break;
        }
        if !exploded { stk.push(a); }
    }
    stk
}
```

**Collision only when:** current asteroid is negative (`a < 0`) AND stack top is positive (`stk.last() > 0`). Same direction = no collision.

---

## Problem 7: Remove K Digits — LC 402

Remove `k` digits from `num` string to get the smallest possible number.

**Greedy with monotonic stack:** maintain an increasing stack. Whenever we see a digit smaller than the top, pop (that digit is a larger digit we should remove).

```rust
fn remove_kdigits(num: String, mut k: i32) -> String {
    let mut stk: Vec<char> = Vec::new();
    for c in num.chars() {
        while k > 0 && !stk.is_empty() && *stk.last().unwrap() > c {
            stk.pop();
            k -= 1;
        }
        stk.push(c);
    }
    // If k > 0, remove from the end (digits are already in increasing order)
    while k > 0 {
        stk.pop();
        k -= 1;
    }

    // Build result, skipping leading zeros
    let mut result = String::new();
    let mut leading_zero = true;
    // stk is already in insertion order
    for &c in &stk {
        if leading_zero && c == '0' { continue; }
        leading_zero = false;
        result.push(c);
    }
    if result.is_empty() { "0".to_string() } else { result }
}
```

**Complexity:** O(n) time, O(n) space

---

## Problem 8: Trapping Rain Water — LC 42

**Input:** `[0,1,0,2,1,0,1,3,2,1,2,1]` → `6`

### Monotonic Stack Approach — O(n) time, O(n) space

```rust
fn trap(height: &[i32]) -> i32 {
    let mut stk: Vec<usize> = Vec::new();  // decreasing stack of indices
    let mut water = 0i32;

    for i in 0..height.len() {
        while !stk.is_empty() && height[*stk.last().unwrap()] < height[i] {
            let bottom = stk.pop().unwrap();
            if stk.is_empty() { break; }
            let left = *stk.last().unwrap();
            let width = (i - left - 1) as i32;
            let bounded_height = height[left].min(height[i]) - height[bottom];
            water += width * bounded_height;
        }
        stk.push(i);
    }
    water
}
```

**Layer-by-layer concept:** The stack maintains a valley shape. When a right wall (taller element) is found, we compute the water trapped between the left wall (new stack top after pop) and right wall at the height of the just-popped element.

### Two-Pointer Approach — O(n) time, O(1) space

```rust
fn trap(height: &[i32]) -> i32 {
    let mut left = 0usize;
    let mut right = height.len() - 1;
    let mut max_left = 0i32;
    let mut max_right = 0i32;
    let mut water = 0i32;
    while left < right {
        if height[left] <= height[right] {
            if height[left] >= max_left { max_left = height[left]; }
            else { water += max_left - height[left]; }
            left += 1;
        } else {
            if height[right] >= max_right { max_right = height[right]; }
            else { water += max_right - height[right]; }
            right -= 1;
        }
    }
    water
}
```

**Which to use in interview?** Two-pointer is O(1) space and simpler to code. Monotonic stack approach generalizes more easily to follow-ups.

---

## Problem 9: Sum of Subarray Minimums — LC 907

For each subarray, find its minimum and sum all minimums. Answer modulo 10^9 + 7.

**Key insight:** For each element `A[i]`, count how many subarrays have `A[i]` as their minimum. Use monotonic increasing stack to find:
- `left[i]` = number of subarrays ending at `i` where `A[i]` is minimum (distance to previous smaller element)
- `right[i]` = number of subarrays starting at `i` where `A[i]` is minimum (distance to next smaller or equal element)

```rust
fn sum_subarray_mins(arr: &[i32]) -> i32 {
    const MOD: i64 = 1_000_000_007;
    let n = arr.len();
    let mut left = vec![0i32; n];    // left[i] = # subarrays where arr[i] is min, ending at i
    let mut right = vec![0i32; n];   // right[i] = # subarrays where arr[i] is min, starting at i
    let mut stk: Vec<usize> = Vec::new();

    // Compute left[i]: distance to previous STRICTLY smaller element
    for i in 0..n {
        while !stk.is_empty() && arr[*stk.last().unwrap()] >= arr[i] { stk.pop(); }
        left[i] = if stk.is_empty() { (i + 1) as i32 } else { (i - stk.last().unwrap()) as i32 };
        stk.push(i);
    }
    stk.clear();
    // Compute right[i]: distance to next SMALLER OR EQUAL element
    // Use <= for right to avoid double-counting equal elements
    for i in (0..n).rev() {
        while !stk.is_empty() && arr[*stk.last().unwrap()] > arr[i] { stk.pop(); }
        right[i] = if stk.is_empty() { (n - i) as i32 } else { (*stk.last().unwrap() - i) as i32 };
        stk.push(i);
    }

    let mut ans: i64 = 0;
    for i in 0..n {
        ans = (ans + arr[i] as i64 * left[i] as i64 * right[i] as i64) % MOD;
    }
    ans as i32
}
```

**Why strict `>=` for left but `>` for right?** Prevents double-counting subarrays with equal minimums. The convention: equal elements are counted on the left side.

---

## Problem 10: Remove Duplicate Letters — LC 316

Return the smallest lexicographic subsequence containing all unique characters exactly once.

```rust
fn remove_duplicate_letters(s: &str) -> String {
    let mut count = [0i32; 26];
    let mut in_stack = [false; 26];
    for c in s.chars() { count[(c as u8 - b'a') as usize] += 1; }

    let mut stk: Vec<char> = Vec::new();
    for c in s.chars() {
        count[(c as u8 - b'a') as usize] -= 1;
        if in_stack[(c as u8 - b'a') as usize] { continue; }    // already in result
        // Pop if current char is smaller AND the top char appears later
        while !stk.is_empty()
            && *stk.last().unwrap() > c
            && count[(*stk.last().unwrap() as u8 - b'a') as usize] > 0
        {
            in_stack[(*stk.last().unwrap() as u8 - b'a') as usize] = false;
            stk.pop();
        }
        stk.push(c);
        in_stack[(c as u8 - b'a') as usize] = true;
    }
    stk.iter().collect()
}
```

**Three conditions to pop:** (1) stack not empty, (2) top > current (can improve lexicographic order), (3) top's count > 0 (top will appear again later, so safe to remove now).

---

## Monotonic Stack — Pattern Summary

| Problem Type | Stack Order | Pop When | Answer At |
|-------------|-------------|----------|-----------|
| Next Greater Element | Decreasing | new > top | pop time |
| Next Smaller Element | Increasing | new < top | pop time |
| Stock Span (prev consecutive ≤) | Decreasing | new > top | pop time (accumulate span) |
| 132 Pattern | Decreasing | new > top | maintains `third` |
| Trapping Rain Water | Decreasing | new > top | compute layer on pop |
| Remove K Digits | Increasing | new < top AND k>0 | build from stack |
| Sum of Subarray Mins | Increasing | new < top | left/right distances |
| Remove Duplicate Letters | Stack (lexicographic) | top > new AND will recur | build from stack |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Histogram & Rectangle](./Histogram%20and%20Rectangle.md)
- [Queue & Deque](./Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
