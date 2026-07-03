# Common Mistakes — Sliding Window & Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## Mistake 1: Using `while` Instead of `if` for Max-Length Windows

```rust
// BAD — for CHARACTER REPLACEMENT (max window problem)
while (right - left + 1) - max_freq > k {
    freq[s[left] as usize - b'A' as usize] -= 1;
    left += 1;
}

// GOOD — use if; window size never needs to decrease below current max
if (right - left + 1) - max_freq > k {
    freq[s[left] as usize - b'A' as usize] -= 1;
    left += 1;
}
```

For min-length problems (Minimum Window Substring, Minimum Subarray Sum), use `while`. For max-length problems (Char Replacement, LSWORC), use `if` to avoid shrinking below the already-recorded maximum.

---

## Mistake 2: Missing `if (goal < 0) return 0` in atMost

```rust
// BAD
fn at_most(nums: &[i32], goal: i32) -> i32 {
    let mut left = 0usize;
    let mut sum = 0i32;
    let mut result = 0i32;
    for right in 0..nums.len() {
        sum += nums[right];
        while sum > goal { sum -= nums[left]; left += 1; }  // if goal=-1 and sum=0: 0 > -1 → loop forever
        result += (right - left + 1) as i32;
    }
    result
}

// GOOD
fn at_most(nums: &[i32], goal: i32) -> i32 {
    if goal < 0 { return 0; }  // ← guard
    // ...
    0
}
```

When `exactly(k) = atMost(k) - atMost(k-1)` and `k=0`, we call `atMost(-1)`. The guard prevents infinite loop.

---

## Mistake 3: Forgetting to Advance Both Pointers After Triplet in 3Sum

```rust
// BAD — infinite loop on [0,0,0] since left/right don't advance
while left < right {
    if sum == 0 {
        result.push(vec![...]);
        while left < right && nums[left] == nums[left + 1] { left += 1; }
        while left < right && nums[right] == nums[right - 1] { right -= 1; }
        // MISSING: left += 1; right -= 1;
    }
}

// GOOD
if sum == 0 {
    result.push(vec![...]);
    while left < right && nums[left] == nums[left + 1] { left += 1; }
    while left < right && nums[right] == nums[right - 1] { right -= 1; }
    left += 1; right -= 1;  // ← always advance after recording
}
```

---

## Mistake 4: Wrong Duplicate Skip Condition in kSum

```rust
// BAD — i=0 should not be skipped (first element is never a duplicate of previous)
if nums[i] == nums[i - 1] { continue; }  // index out of bounds when i=0!

// GOOD
if i > 0 && nums[i] == nums[i - 1] { continue; }
```

Also, for inner loops in 4Sum:
```rust
// BAD — skips valid j=i+1 if nums[i+1]==nums[i]
if j > 0 && nums[j] == nums[j - 1] { continue; }

// GOOD — j must be > i+1 (not > 0) to allow the first j after i
if j > i + 1 && nums[j] == nums[j - 1] { continue; }
```

---

## Mistake 5: Not Handling `k <= 1` in Product < K

```rust
// BAD — infinite loop for k=1 since product=1 ≥ 1 forever
fn num_subarray_product_less_than_k(nums: &[i32], k: i32) -> i32 {
    let mut left = 0usize;
    let mut product = 1i32;
    let mut result = 0i32;
    for right in 0..nums.len() {
        product *= nums[right];
        while product >= k { product /= nums[left]; left += 1; }  // never exits if k=1
        result += (right - left + 1) as i32;
    }
    result
}

// GOOD
if k <= 1 { return 0; }
```

---

## Mistake 6: Off-By-One in Fixed Window Initialization

```rust
// BAD — processes first k+1 elements in initial window instead of k
let mut sum = 0.0f64;
for i in 0..=k { sum += nums[i] as f64; }  // ← ..= should be ..

// GOOD
for i in 0..k { sum += nums[i] as f64; }
```

---

## Mistake 7: Min Window — Returning Before Checking Last State

```rust
// BAD — misses updating min_len if window is valid at end
fn min_window(s: &str, t: &str) -> String {
    // ... loop ends ...
    s[min_left..min_left + min_len].to_string()  // min_len might still be usize::MAX
}

// GOOD
if min_len == usize::MAX {
    String::new()
} else {
    s[min_left..min_left + min_len].to_string()
}
```

If `t = "abc"` and `s = "abc"`, the window becomes valid at the end but the `while` shrink block updates `min_len` correctly (it fires during the loop when `have == required`). Still, the `== usize::MAX` guard is needed for cases where `t` has characters not in `s`.

---

## Mistake 8: Longest Subarray After Delete — Wrong Length Formula

```rust
// BAD — returns window size instead of window size - 1
max_len = max_len.max(right - left + 1);

// GOOD — must delete exactly one element
max_len = max_len.max(right - left);  // window_size - 1
```

LC 1493 requires **exactly** one deletion (not "at most one"). The answer is always `windowSize - 1`.

---

## Mistake 9: Two Pointers on Unsorted Array for Sum Problems

```rust
// BAD — trying two-pointer approach without sorting
let mut left = 0usize;
let mut right = nums.len() - 1;
while left < right {
    if nums[left] + nums[right] == target { /* ... */ }
    else if nums[left] + nums[right] < target { left += 1; }
    else { right -= 1; }
}
// WRONG for unsorted arrays — can't make greedy pointer decisions
```

Two pointers for sum problems **require a sorted array**. Sorting is O(n log n) and must be done first.

---

## Mistake 10: Not Removing Entry from Map When Count Reaches Zero

```rust
use std::collections::HashMap;

// BAD — map still has 0-count entries, affecting map.len() check
*basket.entry(fruits[left]).or_insert(0) -= 1;
left += 1;

// GOOD — remove 0-count entries to keep map size accurate
*basket.entry(fruits[left]).or_insert(0) -= 1;
if basket[&fruits[left]] == 0 { basket.remove(&fruits[left]); }
left += 1;
```

For problems that track `map.len() > k` (distinct element count), stale 0-count entries corrupt the size check.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Window Template](./Window%20Template.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
