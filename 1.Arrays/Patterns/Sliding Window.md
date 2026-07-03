# Sliding Window Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `sliding-window` `subarray` `two-pointers` `frequency-map`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Rust Templates](#rust-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Sliding Window maintains a **dynamic subarray** `[l, r]` that expands and contracts according to a validity condition. Instead of recomputing properties from scratch for each window, it incrementally updates as the window slides.

**Two flavors:**

| Type | Window Size | Use When |
|------|-------------|----------|
| Fixed Window | Constant `k` | "subarray of size k with max sum" |
| Variable Window | Grows/shrinks | "longest subarray with property X" |

**Core loop structure (variable window):**
```
expand r → include nums[r]
while (window invalid):
    shrink l → exclude nums[l], l++
update answer
```

---

## When to Use

- Subarray / substring problems with a **contiguous** constraint
- Max/min length subarray satisfying a condition
- Count of distinct elements in window
- Fixed-size window maximum/minimum/sum
- All values non-negative (for sum-based variable window)

---

## Recognition Cues

| Cue in Problem | Style |
|----------------|-------|
| "subarray of size k" | Fixed window |
| "longest subarray / substring with ..." | Variable window (maximize) |
| "smallest subarray with sum ≥ k" | Variable window (minimize) |
| "at most k distinct characters / elements" | Variable window + frequency map |
| "no repeating characters" | Variable window + `HashSet` |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Fixed window (sum) | O(n) | O(1) |
| Variable window (`HashSet`) | O(n) | O(n) |
| Variable window (freq map) | O(n) | O(26) / O(n) |

---

## Rust Templates

### 1. Fixed Window — Maximum Sum of Size K

```rust
fn max_sum_subarray(arr: &[i32], k: usize) -> i32 {
    let mut window_sum: i32 = arr[..k].iter().sum(); // first window
    let mut max_sum = window_sum;
    for i in k..arr.len() {
        window_sum += arr[i] - arr[i - k]; // slide: add new, remove old
        max_sum = max_sum.max(window_sum);
    }
    max_sum
}
// Time: O(n) | Space: O(1)
```

### 2. Variable Window — Longest Subarray with Sum ≤ K (Non-negative values)

```rust
fn longest_subarray_with_sum_k(arr: &[i32], k: i32) -> usize {
    let mut l = 0;
    let mut sum = 0;
    let mut max_len = 0;
    for r in 0..arr.len() {
        sum += arr[r];
        while sum > k {
            sum -= arr[l];
            l += 1; // shrink from left
        }
        max_len = max_len.max(r - l + 1);
    }
    max_len
}
// Time: O(n) | Space: O(1)
// Note: while loop is amortized O(n) total — l never decreases
```

### 3. Variable Window — Longest Substring Without Repeating Characters

```rust
use std::collections::HashMap;

fn length_of_longest_substring(s: &str) -> usize {
    let chars: Vec<char> = s.chars().collect();
    let mut last_seen: HashMap<char, usize> = HashMap::new();
    let mut l = 0;
    let mut max_len = 0;
    for r in 0..chars.len() {
        let c = chars[r];
        if let Some(&prev) = last_seen.get(&c) {
            if prev >= l {
                l = prev + 1; // jump l past the duplicate
            }
        }
        last_seen.insert(c, r);
        max_len = max_len.max(r - l + 1);
    }
    max_len
}
// Time: O(n) | Space: O(min(n, 26))
```

### 4. Variable Window — At Most K Distinct Characters

```rust
use std::collections::HashMap;

fn longest_substring_k_distinct(s: &str, k: usize) -> usize {
    let chars: Vec<char> = s.chars().collect();
    let mut freq: HashMap<char, usize> = HashMap::new();
    let mut l = 0;
    let mut max_len = 0;
    for r in 0..chars.len() {
        *freq.entry(chars[r]).or_insert(0) += 1;
        while freq.len() > k {
            let left = chars[l];
            l += 1;
            *freq.get_mut(&left).unwrap() -= 1;
            if freq[&left] == 0 {
                freq.remove(&left);
            }
        }
        max_len = max_len.max(r - l + 1);
    }
    max_len
}
// Time: O(n) | Space: O(k)
```

### 5. Minimum Window Substring (Hard)

```rust
use std::collections::HashMap;

fn min_window(s: &str, t: &str) -> String {
    if s.is_empty() || t.is_empty() {
        return String::new();
    }
    let s_chars: Vec<char> = s.chars().collect();
    let mut need: HashMap<char, i32> = HashMap::new();
    for c in t.chars() {
        *need.entry(c).or_insert(0) += 1;
    }

    let mut l = 0;
    let mut formed = 0;
    let required = need.len();
    let mut window: HashMap<char, i32> = HashMap::new();
    let mut ans: Option<(usize, usize, usize)> = None; // (length, l, r)

    for r in 0..s_chars.len() {
        let c = s_chars[r];
        *window.entry(c).or_insert(0) += 1;
        if need.contains_key(&c) && window[&c] == need[&c] {
            formed += 1;
        }

        while formed == required {
            let len = r - l + 1;
            if ans.map_or(true, |(prev_len, _, _)| len < prev_len) {
                ans = Some((len, l, r));
            }
            let lc = s_chars[l];
            l += 1;
            *window.get_mut(&lc).unwrap() -= 1;
            if need.contains_key(&lc) && window[&lc] < need[&lc] {
                formed -= 1;
            }
        }
    }

    match ans {
        None => String::new(),
        Some((_, al, ar)) => s_chars[al..=ar].iter().collect(),
    }
}
// Time: O(|s| + |t|) | Space: O(|s| + |t|)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using sliding window when values can be negative | Switch to Prefix Sum + HashMap |
| Not removing key from map when frequency → 0 | Always `map.remove(&key)` when count hits 0 |
| Fixed window: off-by-one when removing old element | Remove `arr[i - k]`, not `arr[i - k + 1]` |
| Variable window: over-shrinking | `while` condition, not `if` |
| Counting `formed` with `>=` instead of `==` | Only increment `formed` when count matches exactly |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Sliding Window Maximum | Monotonic Deque inside window (LC 239) |
| Exactly K distinct = atMost(K) - atMost(K-1) | Count subarrays |
| Permutation in String | Fixed window with char frequency |
| Find All Anagrams | Fixed window + frequency comparison |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Easy | LC 643 |
| [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | LC 3 |
| [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | LC 209 |
| [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | Medium | LC 904 |
| [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | LC 567 |
| [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | LC 438 |
| [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | LC 424 |
| [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | LC 76 |
| [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | LC 239 |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — Sliding Window is Two Pointers with extra bookkeeping
- [Prefix Sum](./Prefix%20Sum.md) — when values can be negative (Sliding Window breaks)
- [Monotonic Stack/Deque](../../7.Stacks_and_Queues/Patterns/Monotonic%20Stack.md) — for window max/min in O(n)

---

> **Interview Tip:** The most important question to ask yourself: "Are all values non-negative?" If yes, sliding window works for sum constraints. If values can be negative, switch to Prefix Sum + HashMap.

> **Last Updated:** 2026-06-26
