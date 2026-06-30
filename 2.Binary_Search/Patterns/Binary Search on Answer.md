# Binary Search on Answer (Search Space Reduction)

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `binary-search-on-answer` `search-space` `monotone-predicate` `min-max`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Java Templates](#java-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Binary Search on Answer doesn't search **in** an array — it searches **for** an answer in a value range `[lo, hi]`. The search space is the set of all possible answer values, and a **predicate function** tells us whether a candidate answer is feasible.

**Core Insight:**
> If `isFeasible(x)` is monotone (false for small x, true for large x), binary search finds the minimum x where `isFeasible(x)` is true.

```
Answer space:  [lo ..... threshold ..... hi]
isFeasible:     F  F  F  F  T  T  T  T  T
                            ↑ this is what we find
```

**The two canonical problem types:**

| Type | What to binary search | Predicate |
|------|-----------------------|-----------|
| **Minimize maximum** | Binary search on the max value | `canAchieveWithMax(m)` |
| **Maximize minimum** | Binary search on the min value | `canAchieveWithMin(m)` |

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

## Java Templates

### Master Template

```java
// Find minimum answer m such that isFeasible(m) is true
public int binarySearchOnAnswer(/* problem params */) {
    int lo = /* minimum possible answer */;
    int hi = /* maximum possible answer */;

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (isFeasible(mid /*, other params */)) {
            hi = mid;        // mid works; try smaller
        } else {
            lo = mid + 1;    // mid doesn't work; need larger
        }
    }
    return lo; // minimum feasible answer
}
```

---

### 1. Koko Eating Bananas

```java
// Koko eats at speed k bananas/hr. Min k to finish all piles in h hours.
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = Arrays.stream(piles).max().getAsInt();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canFinish(piles, h, mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

private boolean canFinish(int[] piles, int h, int speed) {
    int hours = 0;
    for (int pile : piles)
        hours += (pile + speed - 1) / speed; // ceil division
    return hours <= h;
}
// Time: O(n log maxPile) | Space: O(1)
```

### 2. Capacity to Ship Packages in D Days

```java
public int shipWithinDays(int[] weights, int days) {
    int lo = Arrays.stream(weights).max().getAsInt(); // must carry heaviest
    int hi = Arrays.stream(weights).sum();            // carry all in 1 day
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canShip(weights, days, mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

private boolean canShip(int[] weights, int days, int capacity) {
    int daysNeeded = 1, current = 0;
    for (int w : weights) {
        if (current + w > capacity) { daysNeeded++; current = 0; }
        current += w;
    }
    return daysNeeded <= days;
}
// Time: O(n log(sum-max)) | Space: O(1)
```

### 3. Split Array Largest Sum (= Book Allocation = Painter's Partition)

```java
public int splitArray(int[] nums, int k) {
    int lo = Arrays.stream(nums).max().getAsInt();
    int hi = Arrays.stream(nums).sum();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canSplit(nums, k, mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

private boolean canSplit(int[] nums, int k, int maxSum) {
    int parts = 1, current = 0;
    for (int num : nums) {
        if (current + num > maxSum) { parts++; current = 0; }
        current += num;
    }
    return parts <= k;
}
// Time: O(n log(sum)) | Space: O(1)
```

### 4. Aggressive Cows — Maximize Minimum Distance

```java
public int aggressiveCows(int[] stalls, int k) {
    Arrays.sort(stalls);
    int lo = 1, hi = stalls[stalls.length - 1] - stalls[0];
    while (lo < hi) {
        int mid = lo + (hi - lo + 1) / 2;  // upper-mid for maximize
        if (canPlace(stalls, k, mid)) lo = mid;
        else hi = mid - 1;
    }
    return lo;
}

private boolean canPlace(int[] stalls, int k, int minDist) {
    int count = 1, last = stalls[0];
    for (int i = 1; i < stalls.length; i++) {
        if (stalls[i] - last >= minDist) { count++; last = stalls[i]; }
    }
    return count >= k;
}
// Time: O(n log n + n log(max-min)) | Space: O(1)
```

> **Note on upper-mid for maximize:** When searching for *maximum* feasible answer, use `mid = lo + (hi - lo + 1) / 2` to avoid infinite loop when `lo + 1 == hi`.

### 5. Minimum Days to Make M Bouquets

```java
public int minDays(int[] bloomDay, int m, int k) {
    if ((long) m * k > bloomDay.length) return -1;
    int lo = Arrays.stream(bloomDay).min().getAsInt();
    int hi = Arrays.stream(bloomDay).max().getAsInt();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (canMake(bloomDay, m, k, mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}

private boolean canMake(int[] bloomDay, int m, int k, int day) {
    int bouquets = 0, consecutive = 0;
    for (int d : bloomDay) {
        if (d <= day) { consecutive++; if (consecutive == k) { bouquets++; consecutive = 0; } }
        else consecutive = 0;
    }
    return bouquets >= m;
}
```

### 6. Median of Two Sorted Arrays (Hard)

```java
public double findMedianSortedArrays(int[] nums1, int[] nums2) {
    // Ensure nums1 is the shorter array
    if (nums1.length > nums2.length) return findMedianSortedArrays(nums2, nums1);
    int m = nums1.length, n = nums2.length;
    int lo = 0, hi = m;

    while (lo <= hi) {
        int cut1 = lo + (hi - lo) / 2;
        int cut2 = (m + n + 1) / 2 - cut1;

        int l1 = (cut1 == 0) ? Integer.MIN_VALUE : nums1[cut1 - 1];
        int l2 = (cut2 == 0) ? Integer.MIN_VALUE : nums2[cut2 - 1];
        int r1 = (cut1 == m) ? Integer.MAX_VALUE : nums1[cut1];
        int r2 = (cut2 == n) ? Integer.MAX_VALUE : nums2[cut2];

        if (l1 <= r2 && l2 <= r1) {
            if ((m + n) % 2 == 0)
                return (Math.max(l1, l2) + Math.min(r1, r2)) / 2.0;
            else
                return Math.max(l1, l2);
        } else if (l1 > r2) hi = cut1 - 1;
        else lo = cut1 + 1;
    }
    return 0.0;
}
// Time: O(log(min(m,n))) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Wrong `lo`/`hi` initialization | `lo` = min possible, `hi` = max possible. For capacity: `lo = max(arr)`, `hi = sum(arr)` |
| Integer overflow in `hi = sum` | Use `long` or validate input constraints |
| Minimize: `mid = lo + (hi-lo)/2`, `hi = mid` — but maximize: need `mid = lo + (hi-lo+1)/2`, `lo = mid` | Maximize requires upper-mid to avoid infinite loop at `lo+1==hi` |
| Feasibility check using strict vs non-strict | Off-by-one in `<=` vs `<` inside `canAchieve` |
| Not validating that a solution exists | E.g., `if (m*k > n) return -1` before binary search |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Kth missing positive | Binary search on index + math |
| Smallest divisor given threshold | `hi = max(arr)`, minimize `ceil` sum |
| Minimize max distance to gas station | Continuous search space — use `double` lo/hi |
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

> **Interview Tip:** The hardest part is identifying `lo`, `hi`, and the predicate. Once you write `isFeasible(mid)`, the outer binary search structure is always the same. Practice writing feasibility checks for 10+ problems — the pattern becomes mechanical.

> **Last Updated:** 2026-06-26
