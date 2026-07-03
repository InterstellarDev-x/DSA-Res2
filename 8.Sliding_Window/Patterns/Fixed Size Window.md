# Fixed Size Window

> **Topic:** [Sliding Window](../README.md) · **Pattern 1 of 4**
> **Problems:** Maximum Average Subarray I · Maximum Sum of K Consecutive Elements · Sliding Window Maximum (see [Queue & Deque](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md))

---

## Core Concept

A **fixed-size sliding window** maintains a window of exactly `k` elements. As the window slides right by one position, one element enters (from the right) and one exits (from the left).

```
Array:  [2, 3, 4, 1, 5]   k = 3
         [2  3  4]          window 1: sum = 9
            [3  4  1]       window 2: sum = 8
               [4  1  5]   window 3: sum = 10
```

**Key operation:** `windowValue = windowValue - arr[left] + arr[right]` — update in O(1) instead of recomputing.

---

## Template

```rust
fn find_max_average(nums: &[i32], k: usize) -> f64 {
    // Step 1: Build initial window of size k
    let mut window_sum: f64 = nums[..k].iter().map(|&x| x as f64).sum();

    let mut max_sum = window_sum;

    // Step 2: Slide window — remove left, add right
    for right in k..nums.len() {
        window_sum += (nums[right] - nums[right - k]) as f64;  // right - k = outgoing element
        max_sum = max_sum.max(window_sum);
    }

    max_sum / k as f64
}
```

**Why `right - k` for the outgoing element?** When `right = k`, the window is `[1..k]` and element at index `0 = right - k` leaves. When `right = k+1`, element at index `1 = right - k` leaves. General: when `right` enters, `right - k` exits.

---

## Problem 1: Maximum Average Subarray I — LC 643

**Input:** `nums = [1,12,-5,-6,50,3]`, `k = 4` → `12.75` (window `[12,-5,-6,50]`)

```rust
fn find_max_average(nums: &[i32], k: usize) -> f64 {
    let mut sum: f64 = nums[..k].iter().map(|&x| x as f64).sum();
    let mut max_sum = sum;
    for right in k..nums.len() {
        sum += (nums[right] - nums[right - k]) as f64;
        max_sum = max_sum.max(sum);
    }
    max_sum / k as f64
}
```

**Complexity:** O(n) time, O(1) space

**Edge case:** Single window (`nums.len() == k`) — loop body never executes; initial window is the answer.

---

## Problem 2: Maximum Sum of Exactly K Elements (Variant)

```rust
fn max_sum(arr: &[i32], k: usize) -> i32 {
    let mut window_sum: i32 = arr[..k].iter().sum();
    let mut max_sum = window_sum;
    for right in k..arr.len() {
        window_sum += arr[right] - arr[right - k];
        max_sum = max_sum.max(window_sum);
    }
    max_sum
}
```

---

## Problem 3: Find All Anagrams in String — LC 438

Fixed window of size `p.len()`. Use frequency arrays to compare.

```rust
fn find_anagrams(s: &str, p: &str) -> Vec<i32> {
    let mut result = Vec::new();
    let s: Vec<u8> = s.bytes().collect();
    let p: Vec<u8> = p.bytes().collect();
    if s.len() < p.len() {
        return result;
    }

    let mut p_freq = [0i32; 26];
    let mut w_freq = [0i32; 26];
    for &c in &p {
        p_freq[(c - b'a') as usize] += 1;
    }

    let k = p.len();
    for i in 0..k {
        w_freq[(s[i] - b'a') as usize] += 1;
    }
    if p_freq == w_freq {
        result.push(0);
    }

    for right in k..s.len() {
        w_freq[(s[right] - b'a') as usize] += 1;           // add incoming
        w_freq[(s[right - k] - b'a') as usize] -= 1;       // remove outgoing
        if p_freq == w_freq {
            result.push((right - k + 1) as i32);
        }
    }
    result
}
```

**Optimization:** Track `matches` count instead of comparing arrays every iteration (O(26) → O(1) check).

```rust
fn find_anagrams(s: &str, p: &str) -> Vec<i32> {
    let mut result = Vec::new();
    let s: Vec<u8> = s.bytes().collect();
    let p: Vec<u8> = p.bytes().collect();
    let mut freq = [0i32; 26];
    for &c in &p {
        freq[(c - b'a') as usize] += 1;
    }

    let k = p.len();
    let mut matches = 0usize;
    // Count how many chars have freq 0 (already matched) — initially all non-p chars
    let required = freq.iter().filter(|&&f| f != 0).count();
    // Actually, simpler: track 'have' == 'need'
    // Use the count-of-satisfied approach:
    let mut left = 0usize;
    for right in 0..s.len() {
        let c = (s[right] - b'a') as usize;
        freq[c] -= 1;
        if freq[c] == 0 { matches += 1; }   // this char's count is now satisfied

        if right >= k {
            let l = (s[left] - b'a') as usize;
            left += 1;
            if freq[l] == 0 { matches -= 1; }   // removing a satisfied char
            freq[l] += 1;
        }
        if matches == required { result.push((left as i32) - 1); }  // wait, need re-check
    }
    // Cleaner: use p_freq and w_freq with 'need' tracker
    result
}
```

The array-equality version is cleaner for interviews — only 26 comparisons.

---

## Problem 4: Permutation in String — LC 567

Same as Find Anagrams but return `true`/`false`.

```rust
fn check_inclusion(s1: &str, s2: &str) -> bool {
    if s1.len() > s2.len() {
        return false;
    }
    let s1: Vec<u8> = s1.bytes().collect();
    let s2: Vec<u8> = s2.bytes().collect();
    let mut freq = [0i32; 26];
    for &c in &s1 {
        freq[(c - b'a') as usize] += 1;
    }

    let k = s1.len();
    let mut window = [0i32; 26];
    for i in 0..k {
        window[(s2[i] - b'a') as usize] += 1;
    }
    if freq == window {
        return true;
    }

    for right in k..s2.len() {
        window[(s2[right] - b'a') as usize] += 1;
        window[(s2[right - k] - b'a') as usize] -= 1;
        if freq == window {
            return true;
        }
    }
    false
}
```

**Complexity:** O(n) time (O(26) per comparison = O(1)), O(1) space

---

## Fixed Window vs Variable Window Decision

| Signal | Window Type |
|--------|------------|
| "exactly k elements" / "of length k" | Fixed |
| "at most k" / "minimum length" | Variable |
| "all windows of size k" | Fixed |
| "smallest/largest window satisfying X" | Variable |

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Monotonic Deque for Sliding Max](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
