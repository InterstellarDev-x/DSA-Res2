# Prefix Sum Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `prefix-sum` `subarray` `range-query` `hashmap`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Java Template](#java-template)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Prefix Sum precomputes cumulative sums so that any subarray sum `[l, r]` can be answered in **O(1)** after an **O(n)** build phase.

```
prefix[0] = 0
prefix[i] = arr[0] + arr[1] + ... + arr[i-1]

sum(l, r) = prefix[r+1] - prefix[l]
```

The key insight: instead of summing `arr[l..r]` every query, store prefix sums and subtract.

For **subarray sum = K** problems, pair it with a HashMap tracking `count of prefix sums seen so far`. If `prefix[j] - K` was seen, the subarray ending at `j` has sum `K`.

---

## When to Use

- Range sum queries on a static array
- Count subarrays with sum / XOR / product equal to a target
- 2D matrix range sum queries
- Difference arrays for range updates

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "subarray sum equals K" | Prefix Sum + HashMap |
| "number of subarrays with sum ≤ K" | Prefix Sum + Binary Search or sliding window |
| "range sum query, immutable" | Classic Prefix Sum |
| "how many subarrays have even/odd sum" | Prefix Sum parity |
| "minimum operations to make subarray sum 0" | Prefix Sum difference |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix array | O(n) | O(n) |
| Single range query | O(1) | — |
| Count subarrays (with HashMap) | O(n) | O(n) |
| 2D prefix sum build | O(m×n) | O(m×n) |

---

## Java Template

### 1. Classic Prefix Sum (Range Query)

```java
public int[] buildPrefix(int[] arr) {
    int n = arr.length;
    int[] prefix = new int[n + 1]; // prefix[0] = 0
    for (int i = 0; i < n; i++) {
        prefix[i + 1] = prefix[i] + arr[i];
    }
    return prefix;
}

// Sum of arr[l..r] (0-indexed, inclusive)
public int rangeSum(int[] prefix, int l, int r) {
    return prefix[r + 1] - prefix[l];
}
```

### 2. Count Subarrays with Sum = K

```java
public int subarraySum(int[] nums, int k) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1); // empty prefix
    int sum = 0, count = 0;

    for (int num : nums) {
        sum += num;
        // If (sum - k) was seen, those subarrays sum to k
        count += prefixCount.getOrDefault(sum - k, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
// Time: O(n) | Space: O(n)
```

### 3. Largest Subarray with Sum 0

```java
public int largestSubarrayWithZeroSum(int[] arr) {
    Map<Integer, Integer> firstSeen = new HashMap<>();
    firstSeen.put(0, -1);
    int sum = 0, maxLen = 0;

    for (int i = 0; i < arr.length; i++) {
        sum += arr[i];
        if (firstSeen.containsKey(sum)) {
            maxLen = Math.max(maxLen, i - firstSeen.get(sum));
        } else {
            firstSeen.put(sum, i);
        }
    }
    return maxLen;
}
// Time: O(n) | Space: O(n)
```

### 4. 2D Prefix Sum

```java
public int[][] build2DPrefix(int[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[][] p = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            p[i][j] = matrix[i-1][j-1] + p[i-1][j] + p[i][j-1] - p[i-1][j-1];
    return p;
}

// Sum of submatrix (r1,c1) to (r2,c2) — 0-indexed
public int query2D(int[][] p, int r1, int c1, int r2, int c2) {
    return p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1];
}
// Time: O(1) per query after O(m*n) build
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Off-by-one: `prefix[i] = arr[0..i]` vs `arr[0..i-1]` | Use 1-indexed prefix: `prefix[0]=0`, `prefix[i]=prefix[i-1]+arr[i-1]` |
| Forgetting to seed `prefixCount.put(0, 1)` | Always insert the empty prefix before the loop |
| Integer overflow on large arrays | Use `long` for sum accumulation |
| Modifying prefix array while answering queries | Prefix is read-only after build |
| 2D prefix: forgetting the inclusion-exclusion formula | `p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]` |

---

## Variations

| Variation | Description | Key Change |
|-----------|-------------|------------|
| XOR Prefix | `prefix[i] = arr[0] XOR ... XOR arr[i-1]` | Replace `+` with `^` |
| Product Prefix | Prefix products for range product queries | Divide instead of subtract |
| Difference Array | Range increment/decrement in O(1) | Inverse of prefix sum |
| Running Maximum | Track max prefix so far | Use `Math.max` instead of `+` |
| Subarray with sum divisible by K | Prefix mod K + HashMap | Store `sum % k` |

### Subarray Sum Divisible by K

```java
public int subarraysDivByK(int[] nums, int k) {
    Map<Integer, Integer> modCount = new HashMap<>();
    modCount.put(0, 1);
    int sum = 0, count = 0;
    for (int num : nums) {
        sum = ((sum + num) % k + k) % k; // handle negatives
        count += modCount.getOrDefault(sum, 0);
        modCount.merge(sum, 1, Integer::sum);
    }
    return count;
}
```

---

## Practice Problems

| Problem | Difficulty | LeetCode | Pattern |
|---------|-----------|----------|---------|
| [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | Easy | LC 303 | Classic Prefix |
| [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium | LC 560 | Prefix + HashMap |
| [Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Medium | LC 525 | Prefix (0→-1 trick) |
| [Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | Medium | LC 974 | Prefix mod |
| [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | Medium | LC 1248 | Prefix (odd→1) |
| [Maximum Size Subarray Sum Equals k](https://leetcode.com/problems/maximum-size-subarray-sum-equals-k/) | Medium | LC 325 | Prefix + first-seen |
| [Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/) | Medium | LC 304 | 2D Prefix |
| Largest Subarray with 0 Sum | Medium | GFG | Prefix + first-seen |
| Count Subarrays with XOR = K | Medium | InterviewBit | XOR Prefix |

---

## Related Patterns

- [Sliding Window](./Sliding%20Window.md) — for variable-size subarray problems with constraints
- [Kadane's Algorithm](./Kadane's%20Algorithm.md) — for maximum subarray sum (no target K)
- [Two Pointers](./Two%20Pointers.md) — when all values are non-negative (monotone prefix)

---

> **Interview Tip:** When a problem says "count subarrays with property X", immediately think Prefix Sum + HashMap. The map stores seen prefix values; the check is `prefix[j] - target` already in map.

> **Last Updated:** 2026-06-26
