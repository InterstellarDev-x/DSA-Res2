# Binary Search on Answer (Search Space Reduction)

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `binary-search-on-answer` `search-space` `monotone-predicate` `min-max`

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

Binary Search on Answer doesn't search **in** an array — it searches **for** an answer in a value range `[lo, hi]`. The search space is the set of all possible answer values, and a **predicate function** tells us whether a candidate answer is feasible.

**Core Insight:**
> If `is_feasible(x)` is monotone (false for small x, true for large x), binary search finds the minimum x where `is_feasible(x)` is true.

```
Answer space:  [lo ..... threshold ..... hi]
isFeasible:     F  F  F  F  T  T  T  T  T
                            ↑ this is what we find
```

**The two canonical problem types:**

| Type | What to binary search | Predicate |
|------|-----------------------|-----------|
| **Minimize maximum** | Binary search on the max value | `can_achieve_with_max(m)` |
| **Maximize minimum** | Binary search on the min value | `can_achieve_with_min(m)` |

---

## When to Use

- Problem asks for "minimum of maximum" or "maximum of minimum"
- Greedy check exists: given a fixed value, can you verify feasibility fast (O(n) or O(n log n))?
- Problem involves allocation: books to students, packages to days, workers to jobs
- Integer answer in a known range

---

## Recognition Cues

| Phrase in Problem | Binary Search on Answer |
|------------------|------------------------|
| "minimum number of days / operations" | Search on days/ops range |
| "minimum speed / maximum capacity" | Search on that metric |
| "allocate to minimize maximum" | Minimize-max template |
| "split array to minimize largest sum" | Minimize-max template |
| "aggressive cows / maximum minimum distance" | Maximize-min template |
| "k workers, minimize time" | Minimize-max template |

---

## Complexity

| Component | Complexity |
|-----------|-----------|
| Binary search outer loop | O(log(hi - lo)) |
| Feasibility check (linear) | O(n) per call |
| **Total** | **O(n log(hi - lo))** |

---

## Rust Templates

### Master Template

```rust
// Find minimum answer m such that is_feasible(m) is true
fn binary_search_on_answer(/* problem params */) -> i32 {
    let mut lo: i32 = /* minimum possible answer */;
    let mut hi: i32 = /* maximum possible answer */;

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if is_feasible(mid /*, other params */) {
            hi = mid;        // mid works; try smaller
        } else {
            lo = mid + 1;    // mid doesn't work; need larger
        }
    }
    lo // minimum feasible answer
}
```

---

### 1. Koko Eating Bananas

```rust
// Koko eats at speed k bananas/hr. Min k to finish all piles in h hours.
fn can_finish(piles: &[i32], h: i32, speed: i32) -> bool {
    let mut hours = 0i32;
    for &pile in piles {
        hours += (pile + speed - 1) / speed; // ceil division
    }
    hours <= h
}

fn min_eating_speed(piles: Vec<i32>, h: i32) -> i32 {
    let mut lo = 1;
    let mut hi = *piles.iter().max().unwrap();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if can_finish(&piles, h, mid) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    lo
}
// Time: O(n log maxPile) | Space: O(1)
```

### 2. Capacity to Ship Packages in D Days

```rust
fn can_ship(weights: &[i32], days: i32, capacity: i32) -> bool {
    let mut days_needed = 1i32;
    let mut current = 0i32;
    for &w in weights {
        if current + w > capacity {
            days_needed += 1;
            current = 0;
        }
        current += w;
    }
    days_needed <= days
}

fn ship_within_days(weights: Vec<i32>, days: i32) -> i32 {
    let mut lo = *weights.iter().max().unwrap(); // must carry heaviest
    let mut hi: i32 = weights.iter().sum();      // carry all in 1 day
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if can_ship(&weights, days, mid) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    lo
}
// Time: O(n log(sum-max)) | Space: O(1)
```

### 3. Split Array Largest Sum (= Book Allocation = Painter's Partition)

```rust
fn can_split(nums: &[i32], k: i32, max_sum: i32) -> bool {
    let mut parts = 1i32;
    let mut current = 0i32;
    for &num in nums {
        if current + num > max_sum {
            parts += 1;
            current = 0;
        }
        current += num;
    }
    parts <= k
}

fn split_array(nums: Vec<i32>, k: i32) -> i32 {
    let mut lo = *nums.iter().max().unwrap();
    let mut hi: i32 = nums.iter().sum();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if can_split(&nums, k, mid) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    lo
}
// Time: O(n log(sum)) | Space: O(1)
```

### 4. Aggressive Cows — Maximize Minimum Distance

```rust
fn can_place(stalls: &[i32], k: i32, min_dist: i32) -> bool {
    let mut count = 1i32;
    let mut last = stalls[0];
    for i in 1..stalls.len() {
        if stalls[i] - last >= min_dist {
            count += 1;
            last = stalls[i];
        }
    }
    count >= k
}

fn aggressive_cows(mut stalls: Vec<i32>, k: i32) -> i32 {
    stalls.sort();
    let mut lo = 1;
    let mut hi = stalls[stalls.len() - 1] - stalls[0];
    while lo < hi {
        let mid = lo + (hi - lo + 1) / 2;  // upper-mid for maximize
        if can_place(&stalls, k, mid) {
            lo = mid;
        } else {
            hi = mid - 1;
        }
    }
    lo
}
// Time: O(n log n + n log(max-min)) | Space: O(1)
```

> **Note on upper-mid for maximize:** When searching for *maximum* feasible answer, use `mid = lo + (hi - lo + 1) / 2` to avoid infinite loop when `lo + 1 == hi`.

### 5. Minimum Days to Make M Bouquets

```rust
fn can_make(bloom_day: &[i32], m: i32, k: i32, day: i32) -> bool {
    let mut bouquets = 0i32;
    let mut consecutive = 0i32;
    for &d in bloom_day {
        if d <= day {
            consecutive += 1;
            if consecutive == k {
                bouquets += 1;
                consecutive = 0;
            }
        } else {
            consecutive = 0;
        }
    }
    bouquets >= m
}

fn min_days(bloom_day: Vec<i32>, m: i32, k: i32) -> i32 {
    if (m as i64) * (k as i64) > bloom_day.len() as i64 {
        return -1;
    }
    let mut lo = *bloom_day.iter().min().unwrap();
    let mut hi = *bloom_day.iter().max().unwrap();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if can_make(&bloom_day, m, k, mid) {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    lo
}
```

### 6. Median of Two Sorted Arrays (Hard)

```rust
fn find_median_sorted_arrays(nums1: Vec<i32>, nums2: Vec<i32>) -> f64 {
    // Ensure nums1 is the shorter array
    if nums1.len() > nums2.len() {
        return find_median_sorted_arrays(nums2, nums1);
    }
    let m = nums1.len();
    let n = nums2.len();
    let mut lo = 0usize;
    let mut hi = m;

    while lo <= hi {
        let cut1 = lo + (hi - lo) / 2;
        let cut2 = (m + n + 1) / 2 - cut1;

        let l1 = if cut1 == 0 { i32::MIN } else { nums1[cut1 - 1] };
        let l2 = if cut2 == 0 { i32::MIN } else { nums2[cut2 - 1] };
        let r1 = if cut1 == m { i32::MAX } else { nums1[cut1] };
        let r2 = if cut2 == n { i32::MAX } else { nums2[cut2] };

        if l1 <= r2 && l2 <= r1 {
            if (m + n) % 2 == 0 {
                return (l1.max(l2) + r1.min(r2)) as f64 / 2.0;
            } else {
                return l1.max(l2) as f64;
            }
        } else if l1 > r2 {
            if cut1 == 0 { break; }
            hi = cut1 - 1;
        } else {
            lo = cut1 + 1;
        }
    }
    0.0
}
// Time: O(log(min(m,n))) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong `lo`/`hi` initialization | `lo` = min possible, `hi` = max possible. For capacity: `lo = max(arr)`, `hi = sum(arr)` |
| Integer overflow in `hi = sum` | Use `i64` or validate input constraints |
| Minimize: `mid = lo + (hi-lo)/2`, `hi = mid` — but maximize: need `mid = lo + (hi-lo+1)/2`, `lo = mid` | Maximize requires upper-mid to avoid infinite loop at `lo+1==hi` |
| Feasibility check using strict vs non-strict | Off-by-one in `<=` vs `<` inside `can_achieve` |
| Not validating that a solution exists | E.g., `if m*k > n { return -1; }` before binary search |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Kth missing positive | Binary search on index + math |
| Smallest divisor given threshold | `hi = max(arr)`, minimize `ceil` sum |
| Minimize max distance to gas station | Continuous search space — use `f64` lo/hi |
| Nth root of M | `lo=1, hi=m`, check `mid^n` with overflow guard |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium | LC 875 |
| [Capacity to Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Medium | LC 1011 |
| [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Hard | LC 410 |
| [Minimum Number of Days to Make M Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) | Medium | LC 1482 |
| [Find the Smallest Divisor](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/) | Medium | LC 1283 |
| [Kth Missing Positive Number](https://leetcode.com/problems/kth-missing-positive-number/) | Easy | LC 1539 |
| [Median of Two Sorted Arrays](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard | LC 4 |
| Aggressive Cows | Medium | SPOJ / GFG |
| Book Allocation | Medium | GFG |
| Painter's Partition | Hard | GFG |

---

## Related Patterns

- [Classic Binary Search](./Classic%20Binary%20Search.md) — binary search on an actual array
- [Lower and Upper Bound](./Lower%20and%20Upper%20Bound.md) — predicate-based bounds
- [Greedy](../../10.Greedy/README.md) — feasibility check is often a greedy scan

---

> **Interview Tip:** The hardest part is identifying `lo`, `hi`, and the predicate. Once you write `is_feasible(mid)`, the outer binary search structure is always the same. Practice writing feasibility checks for 10+ problems — the pattern becomes mechanical.

> **Last Updated:** 2026-06-26
