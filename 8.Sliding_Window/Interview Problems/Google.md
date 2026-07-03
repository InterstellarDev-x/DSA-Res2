# Google — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Google

---

## Problem 1: Binary Subarrays With Sum — Deep Dive

**LC 930** · Medium

### Both Approaches

```rust
use std::collections::HashMap;

// Approach 1: atMost trick — O(n) time, O(1) space
fn num_subarrays_with_sum(nums: &[i32], goal: i32) -> i32 {
    at_most(nums, goal) - at_most(nums, goal - 1)
}

fn at_most(nums: &[i32], goal: i32) -> i32 {
    if goal < 0 { return 0; }
    let mut left = 0;
    let mut sum = 0;
    let mut result = 0;
    for right in 0..nums.len() {
        sum += nums[right];
        while sum > goal {
            sum -= nums[left];
            left += 1;
        }
        result += (right - left + 1) as i32;
    }
    result
}

// Approach 2: Prefix Sum + HashMap — O(n) time, O(n) space
fn num_subarrays_with_sum_v2(nums: &[i32], goal: i32) -> i32 {
    let mut prefix_count: HashMap<i32, i32> = HashMap::new();
    prefix_count.insert(0, 1);
    let mut sum = 0;
    let mut result = 0;
    for &num in nums {
        sum += num;
        result += prefix_count.get(&(sum - goal)).copied().unwrap_or(0);
        *prefix_count.entry(sum).or_insert(0) += 1;
    }
    result
}
```

**Q: When would you prefer prefix sum over atMost?**
A: Prefix sum works for negative numbers too. `atMost` sliding window assumes monotonic sums — adding an element can only increase the sum. For arrays with negatives, the `while (sum > goal)` shrink loop can overshoot. Prefix sum doesn't have this restriction.

**Q: Why `prefix_count[0] = 1` initialization?**
A: The subarray from index 0 to some index `j` has sum = `prefix[j+1]`. For this to equal `goal`, we need `prefix[j+1] - prefix[0] = goal → prefix[0] = prefix[j+1] - goal`. Since `prefix[0] = 0`, we initialize with `count[0] = 1` to count subarrays starting from index 0.

---

## Problem 2: Max Consecutive Ones III — Google Style

**LC 1004** · Medium

```rust
fn longest_ones(nums: &[i32], k: i32) -> i32 {
    let mut left = 0;
    let mut zeros = 0;
    let mut max_len = 0;
    for right in 0..nums.len() {
        if nums[right] == 0 { zeros += 1; }
        while zeros > k {
            if nums[left] == 0 { zeros -= 1; }
            left += 1;
        }
        max_len = max_len.max(right - left + 1);
    }
    max_len as i32
}
```

**Q: Google follow-up — what if we can also flip at most k ones to zeros (i.e., keep at most k zeros OR ones)?**
A: Two separate calls: `max(longest_ones(nums, k), longest_zeros(nums, k))` where `longest_zeros` flips the values.

**Q: Google follow-up — if you could flip any substring (not individual bits) at most once, find the longest all-ones subarray?**
A: This becomes a different problem — find the longest window with ≤ 1 "block" of zeros (since flipping a substring flips all bits in it). Use segment analysis or dynamic programming.

---

## Problem 3: Number of Substrings Containing All Three Characters

**LC 1358** · Medium

```rust
fn number_of_substrings(s: &str) -> i32 {
    let s: Vec<u8> = s.bytes().collect();
    let mut count = [0i32; 3];
    let mut left = 0;
    let mut result = 0;
    for right in 0..s.len() {
        count[(s[right] - b'a') as usize] += 1;
        while count[0] > 0 && count[1] > 0 && count[2] > 0 {
            result += (s.len() - right) as i32;
            count[(s[left] - b'a') as usize] -= 1;
            left += 1;
        }
    }
    result
}
```

**Q: Explain the `result += s.len() - right` insight.**
A: When `[left..right]` contains all three characters, extending the right boundary to any position from `right` to `n-1` still contains all three. So every substring `[left..right]`, `[left..right+1]`, ..., `[left..n-1]` is valid — that's `n - right` subarrays starting at `left`. Then we shrink `left` and check if the shorter window still has all three.

**Google L4 follow-up:** Generalize to count substrings containing all characters of a given target string `t`. Use the `have/need` framework from Minimum Window Substring.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | LSWORC, Max Consecutive Ones, Minimum Subarray Sum |
| L4 | Min Window Substring, Binary Subarrays, All Three Characters |
| L5+ | Min Window Substring with follow-ups, combined prefix sum + sliding window |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [At Most K Trick](../Patterns/At%20Most%20K%20Trick.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
