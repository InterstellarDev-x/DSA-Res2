# Kadane's Algorithm Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `kadane` `subarray` `dynamic-programming` `greedy`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [C++ Templates](#c-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Kadane's Algorithm finds the **maximum sum contiguous subarray** in O(n) time.

**Core Insight:** At each index `i`, decide whether to extend the existing subarray or start fresh. Start fresh when the current running sum becomes negative — it can only hurt future sums.

```
maxEndingHere = max(arr[i], maxEndingHere + arr[i])
maxSoFar      = max(maxSoFar, maxEndingHere)
```

This is a **greedy-DP hybrid**: greedy in the extension decision, DP in state transition.

---

## When to Use

- Maximum (or minimum) sum subarray
- Maximum product subarray
- Maximum sum circular subarray
- Maximum sum rectangle in a 2D matrix
- Variants where you need indices of the subarray

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "maximum sum of contiguous subarray" | Classic Kadane |
| "maximum product subarray" | Kadane with two states (max, min) |
| "maximum sum subarray in circular array" | Kadane + (totalSum - minKadane) |
| "find subarray with largest sum" | Kadane + index tracking |
| "at most one deletion" | Kadane forward + backward |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Classic Kadane | O(n) | O(1) |
| With indices | O(n) | O(1) |
| Max product | O(n) | O(1) |
| Circular | O(n) | O(1) |
| 2D max rectangle | O(m × n²) | O(m) |

---

## C++ Templates

### 1. Classic Kadane — Maximum Sum

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSubArray(vector<int>& nums) {
    int maxSoFar = nums[0];
    int maxEndingHere = nums[0];

    for (int i = 1; i < nums.size(); i++) {
        maxEndingHere = max(nums[i], maxEndingHere + nums[i]);
        maxSoFar = max(maxSoFar, maxEndingHere);
    }
    return maxSoFar;
}
// Time: O(n) | Space: O(1)
```

### 2. Kadane with Subarray Indices

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> maxSubArrayWithIndices(vector<int>& nums) {
    int maxSum = nums[0], curSum = nums[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < nums.size(); i++) {
        if (curSum + nums[i] < nums[i]) {
            curSum = nums[i];
            tempStart = i;
        } else {
            curSum += nums[i];
        }
        if (curSum > maxSum) {
            maxSum = curSum;
            start = tempStart;
            end = i;
        }
    }
    return vector<int>{maxSum, start, end};
}
// Returns [maxSum, startIndex, endIndex]
```

### 3. Maximum Product Subarray

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxProduct(vector<int>& nums) {
    int maxProd = nums[0];
    int curMax = nums[0], curMin = nums[0]; // track both: negatives flip sign

    for (int i = 1; i < nums.size(); i++) {
        int temp = curMax;
        curMax = max(nums[i], max(curMax * nums[i], curMin * nums[i]));
        curMin = min(nums[i], min(temp * nums[i], curMin * nums[i]));
        maxProd = max(maxProd, curMax);
    }
    return maxProd;
}
// Time: O(n) | Space: O(1)
```

### 4. Maximum Sum Circular Subarray

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSubarraySumCircular(vector<int>& nums) {
    int totalSum = 0;
    int maxSum = nums[0], curMax = 0;
    int minSum = nums[0], curMin = 0;

    for (auto& num : nums) {
        curMax = max(curMax + num, num);
        maxSum = max(maxSum, curMax);

        curMin = min(curMin + num, num);
        minSum = min(minSum, curMin);

        totalSum += num;
    }
    // If all negative, totalSum - minSum wraps entire array → invalid
    return maxSum > 0 ? max(maxSum, totalSum - minSum) : maxSum;
}
// Time: O(n) | Space: O(1)
```

### 5. Minimum Sum Subarray (Kadane Inverted)

```cpp
#include <bits/stdc++.h>
using namespace std;

int minSubArray(vector<int>& nums) {
    int minSoFar = nums[0];
    int minEndingHere = nums[0];

    for (int i = 1; i < nums.size(); i++) {
        minEndingHere = min(nums[i], minEndingHere + nums[i]);
        minSoFar = min(minSoFar, minEndingHere);
    }
    return minSoFar;
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Initializing `maxSoFar = 0` | Init to `nums[0]` — all-negative arrays have negative max |
| Starting loop at index 0 | Start `maxEndingHere = nums[0]`, loop from index 1 |
| Product variant: forgetting `curMin` | Negative × negative = large positive; track both max and min |
| Circular: returning `totalSum - minSum` when all elements negative | Guard: `if maxSum < 0, return maxSum` |
| Index tracking: not resetting `tempStart` correctly | Reset `tempStart = i` only when starting fresh |

---

## Variations

| Variation | Description | Key Change |
|-----------|-------------|------------|
| Count of max-sum subarrays | Count how many subarrays achieve max sum | Track count alongside |
| At most k deletions | DP with state `(index, deletions_used)` | O(n×k) DP |
| Max sum with no two adjacent | House Robber pattern | DP |
| Max sum submatrix | Fix row bounds, Kadane on column sums | O(m²×n) |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | Medium | LC 53 |
| [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Medium | LC 152 |
| [Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Medium | LC 918 |
| [Maximum Subarray Sum with One Deletion](https://leetcode.com/problems/maximum-subarray-sum-with-one-deletion/) | Medium | LC 1186 |
| [Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/) | Hard | LC 363 |
| [K-Concatenation Maximum Sum](https://leetcode.com/problems/k-concatenation-maximum-sum/) | Medium | LC 1191 |

---

## Related Patterns

- [Prefix Sum](./Prefix%20Sum.md) — when you need exact sum = K, not maximum
- [Sliding Window](./Sliding%20Window.md) — when all values are non-negative (can shrink window)
- [Dynamic Programming](../../14.Dynamic_Programming/README.md) — Kadane is DP with O(1) state compression

---

> **Interview Tip:** Interviewers love asking "what if all numbers are negative?" — always initialize from `nums[0]` and start the loop at index 1. Also, follow-up: "print the actual subarray" — track `tempStart` and `start`/`end` indices.

> **Last Updated:** 2026-06-26
