# Variable Size Window

> **Topic:** [Sliding Window](../README.md) · **Pattern 2 of 4**
> **Problems:** Min Subarray Sum · LSWORC · Max Cons Ones III · Fruits Into Baskets · Char Replacement · Min Window Substring · All Three Characters · Longest Subarray After Delete

---

## Core Concept

A **variable-size window** expands its right boundary to include new elements and shrinks its left boundary when a constraint is violated. The window is always valid — it represents a range satisfying the problem constraint.

**Two variants:**

| Variant | Shrink When | Typical Goal |
|---------|-------------|--------------|
| **Minimize window** | Constraint is MET (keep shrinking as long as valid) | Minimum length satisfying constraint |
| **Maximize window** | Constraint is VIOLATED (shrink until valid again) | Maximum length satisfying constraint |

---

## Template A: Maximize Window (Shrink on Violation)

```rust
// Find the longest window satisfying the constraint
let mut left = 0usize;
let mut max_len = 0usize;
// data structure to track window state (HashMap, array, counter)

for right in 0..n {
    // 1. Expand: add nums[right] to window
    add(nums[right]);

    // 2. Shrink: while constraint violated, remove from left
    while violated() {
        remove(nums[left]);
        left += 1;
    }

    // 3. Window [left..right] is now valid — update answer
    max_len = max_len.max(right - left + 1);
}
max_len
```

---

## Template B: Minimize Window (Shrink While Valid)

```rust
// Find the smallest window satisfying the constraint
let mut left = 0usize;
let mut min_len = usize::MAX;

for right in 0..n {
    // 1. Expand: add nums[right]
    add(nums[right]);

    // 2. Shrink: while constraint is satisfied, try to make window smaller
    while satisfied() {
        min_len = min_len.min(right - left + 1);
        remove(nums[left]);
        left += 1;
    }
}
if min_len == usize::MAX { 0 } else { min_len }
```

---

## Problem 1: Minimum Size Subarray Sum — LC 209

Find the minimum length subarray with sum ≥ target.

```rust
fn min_sub_array_len(target: i32, nums: &[i32]) -> i32 {
    let mut left = 0usize;
    let mut sum = 0i32;
    let mut min_len = i32::MAX;

    for right in 0..nums.len() {
        sum += nums[right];
        while sum >= target {            // constraint MET → shrink
            min_len = min_len.min((right - left + 1) as i32);
            sum -= nums[left];
            left += 1;
        }
    }
    if min_len == i32::MAX { 0 } else { min_len }
}
```

**Why `while` and not `if`?** After removing `nums[left]` from the window, the sum might still be ≥ target. We keep shrinking as long as the window remains valid (shrinking finds a shorter valid window).

**Complexity:** O(n) time — each element enters and exits the window at most once.

---

## Problem 2: Longest Substring Without Repeating Characters — LC 3

```rust
fn length_of_longest_substring(s: &str) -> i32 {
    let s: Vec<u8> = s.bytes().collect();
    let mut freq = [0i32; 128];  // ASCII
    let mut left = 0usize;
    let mut max_len = 0i32;

    for right in 0..s.len() {
        freq[s[right] as usize] += 1;

        while freq[s[right] as usize] > 1 {  // duplicate found → shrink
            freq[s[left] as usize] -= 1;
            left += 1;
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Alternative: HashMap with last-seen index — jump left directly**

```rust
use std::collections::HashMap;

fn length_of_longest_substring(s: &str) -> i32 {
    let s: Vec<char> = s.chars().collect();
    let mut last_seen: HashMap<char, usize> = HashMap::new();
    let mut left = 0usize;
    let mut max_len = 0i32;

    for right in 0..s.len() {
        let c = s[right];
        if let Some(&idx) = last_seen.get(&c) {
            if idx >= left {
                left = idx + 1;   // jump left past the duplicate
            }
        }
        last_seen.insert(c, right);
        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Which to use?** Frequency array approach is more general (works for all sliding window problems). Last-seen approach is faster for this specific problem but only applies when the window shrinks to a specific index.

---

## Problem 3: Max Consecutive Ones III — LC 1004

Flip at most `k` zeroes. Find the longest subarray of ones.

```rust
fn longest_ones(nums: &[i32], k: i32) -> i32 {
    let mut left = 0usize;
    let mut zeros = 0i32;
    let mut max_len = 0i32;

    for right in 0..nums.len() {
        if nums[right] == 0 { zeros += 1; }

        while zeros > k {          // too many zeros → shrink
            if nums[left] == 0 { zeros -= 1; }
            left += 1;
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Framing:** "Flip at most k zeroes" = "at most k zeroes in the window." Window shrinks when zero count exceeds k.

---

## Problem 4: Fruits Into Baskets — LC 904

Two baskets (at most 2 distinct fruit types). Find the longest subarray with ≤ 2 distinct values.

```rust
use std::collections::HashMap;

fn total_fruit(fruits: &[i32]) -> i32 {
    let mut basket: HashMap<i32, i32> = HashMap::new();
    let mut left = 0usize;
    let mut max_len = 0i32;

    for right in 0..fruits.len() {
        *basket.entry(fruits[right]).or_insert(0) += 1;  // add fruit

        while basket.len() > 2 {                    // > 2 types → shrink
            let entry = basket.entry(fruits[left]).or_insert(0);
            *entry -= 1;
            if *entry == 0 { basket.remove(&fruits[left]); }
            left += 1;
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**General pattern:** "At most k distinct elements" — replace `2` with `k`. This is a classic category.

---

## Problem 5: Longest Repeating Character Replacement — LC 424

Replace at most `k` characters. Find the longest substring where you can make all characters the same.

**Key insight:** In a valid window, the number of characters to replace = `window_size - max_frequency`. If this exceeds `k`, shrink.

```rust
fn character_replacement(s: &str, k: i32) -> i32 {
    let s: Vec<u8> = s.bytes().collect();
    let mut freq = [0i32; 26];
    let mut left = 0usize;
    let mut max_freq = 0i32;
    let mut max_len = 0i32;

    for right in 0..s.len() {
        freq[(s[right] - b'A') as usize] += 1;
        max_freq = max_freq.max(freq[(s[right] - b'A') as usize]);

        // Window size - max_freq = chars to replace
        if (right - left + 1) as i32 - max_freq > k {   // shrink
            freq[(s[left] - b'A') as usize] -= 1;
            left += 1;
            // Note: max_freq is NOT recomputed — see explanation below
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Why don't we recompute `max_freq` on shrink?** `max_freq` can only increase or stay the same to improve the answer. Even if the current window's actual max frequency is lower, we don't need a shorter valid window than what we already have. We're looking for the maximum length, so `max_freq` serves as a lower bound — any window with a smaller max frequency would be shorter, which isn't useful.

**Formal:** `max_len = right - left + 1` never shrinks (we shrink by exactly 1 when condition violated, so window size is non-decreasing). `max_freq` correspondingly can only increase to allow a longer valid window.

**Complexity:** O(n) time, O(1) space (26-element array)

---

## Problem 6: Minimum Window Substring — LC 76

Find the minimum window in `s` containing all characters of `t`.

```rust
fn min_window(s: &str, t: &str) -> String {
    if s.len() < t.len() { return String::new(); }

    let s: Vec<u8> = s.bytes().collect();
    let mut need = [0i32; 128];
    for c in t.bytes() { need[c as usize] += 1; }

    let mut left = 0usize;
    let mut have = 0i32;
    let required = t.len() as i32;
    let mut min_len = i32::MAX;
    let mut min_left = 0usize;

    for right in 0..s.len() {
        let c = s[right] as usize;
        if need[c] > 0 { have += 1; }    // we needed this char and now have one more
        need[c] -= 1;                     // reduce need (can go negative for excess chars)

        while have == required {   // all chars covered → try to shrink
            if (right - left + 1) as i32 < min_len {
                min_len = (right - left + 1) as i32;
                min_left = left;
            }
            let l = s[left] as usize;
            need[l] += 1;               // return char to need
            if need[l] > 0 { have -= 1; } // now we need this char again
            left += 1;
        }
    }
    if min_len == i32::MAX {
        String::new()
    } else {
        String::from_utf8(s[min_left..min_left + min_len as usize].to_vec()).unwrap()
    }
}
```

**Why `need[c] > 0` for `have += 1`?** `need` can be negative (we have excess of a char). We only increment `have` when we're actually satisfying a need (not just adding excess). Similarly, `need[l] += 1` returns the char; only if it becomes positive does `have -= 1` (we lost a needed char, not just excess).

**Complexity:** O(|s| + |t|) time, O(1) space (128-element array)

---

## Problem 7: Number of Substrings Containing All Three Characters — LC 1358

Count substrings containing at least one each of 'a', 'b', 'c'.

```rust
fn number_of_substrings(s: &str) -> i32 {
    let s: Vec<u8> = s.bytes().collect();
    let mut count = [0i32; 3];
    let mut left = 0usize;
    let mut result = 0i32;

    for right in 0..s.len() {
        count[(s[right] - b'a') as usize] += 1;

        while count[0] > 0 && count[1] > 0 && count[2] > 0 {
            // All three chars present → count all extensions to the right
            result += (s.len() - right) as i32;   // [left..right], [left..right+1], ..., [left..n-1]
            count[(s[left] - b'a') as usize] -= 1;
            left += 1;
        }
    }
    result
}
```

**Key insight:** When `[left..right]` contains all three, then so does `[left..right+1]`, `[left..right+2]`, ..., `[left..n-1]`. All `n - right` extensions are valid from this `left`. After removing `left`, check if still valid.

---

## Problem 8: Longest Subarray of 1s After Deleting One Element — LC 1493

Delete exactly one element. Find longest subarray of 1s.

```rust
fn longest_subarray(nums: &[i32]) -> i32 {
    let mut left = 0usize;
    let mut zeros = 0i32;
    let mut max_len = 0i32;

    for right in 0..nums.len() {
        if nums[right] == 0 { zeros += 1; }

        while zeros > 1 {
            if nums[left] == 0 { zeros -= 1; }
            left += 1;
        }

        // -1 because we must delete exactly one element (the 0 in the window, or any 1)
        max_len = max_len.max((right - left) as i32);  // right - left = window_size - 1
    }
    max_len
}
```

**Why `right - left` not `right - left + 1`?** We must delete exactly one element. The answer length = window size - 1 (we delete the zero in the window, or any element if window is all ones).

---

## Pattern Summary

| Problem | Window State | Expand Condition | Shrink Condition |
|---------|-------------|-----------------|-----------------|
| Min Subarray Sum | running sum | always | sum ≥ target |
| LSWORC | freq array | always | freq > 1 (duplicate) |
| Max Cons Ones III | zero count | always | zeros > k |
| Fruits Into Baskets | fruit type map | always | map.size() > 2 |
| Char Replacement | freq array + max_freq | always | windowSize - max_freq > k |
| Min Window Substring | need array, have count | always | have == required |
| All Three Characters | char counts | always | all 3 counts > 0 |

---

## Related Files

- [Fixed Size Window](./Fixed%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Two Pointers](./Two%20Pointers.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
