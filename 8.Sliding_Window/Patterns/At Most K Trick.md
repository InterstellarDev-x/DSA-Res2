# At Most K → Exactly K Trick

> **Topic:** [Sliding Window](../README.md) · **Pattern 3 of 4**
> **Problems:** Binary Subarrays With Sum · Count Number of Nice Subarrays · Subarray Product Less Than K

---

## Core Concept

**Problem:** Count subarrays with **exactly** k of some property (e.g., exactly k ones, exactly k odd numbers).

**Challenge:** Sliding window naturally counts subarrays with **at most** k of a property (because "at most k" means "expand freely, shrink when you exceed k"). "Exactly k" doesn't have a natural shrink condition.

**Solution:** `exactly(k) = atMost(k) - atMost(k-1)`

```
atMost(k):  count all subarrays with ≤ k of property P
atMost(k-1): count all subarrays with ≤ k-1 of property P
Difference: exactly k subarrays have exactly k of P
```

---

## `atMost(k)` Counting Template

```cpp
#include <bits/stdc++.h>
using namespace std;

// How many subarrays have AT MOST k of some property?
int atMost(vector<int>& nums, int k) {
    int left = 0, count = 0, result = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        count += contribution(nums[right]);     // add right element

        while (count > k) {                     // too many → shrink
            count -= contribution(nums[left]);
            left++;
        }

        // All subarrays ending at 'right' and starting in [left..right] are valid
        result += right - left + 1;
    }
    return result;
}
```

**Why `right - left + 1`?** When window `[left..right]` is valid (≤ k of property), all subarrays ending at `right` that start anywhere in `[left, right]` are also valid. There are `right - left + 1` such starting positions.

---

## Problem 1: Binary Subarrays With Sum — LC 930

Count subarrays with sum = goal (array of 0s and 1s).

```cpp
#include <bits/stdc++.h>
using namespace std;

int numSubarraysWithSum(vector<int>& nums, int goal) {
    return atMost(nums, goal) - atMost(nums, goal - 1);
}

int atMost(vector<int>& nums, int goal) {
    if (goal < 0) return 0;    // edge case: goal-1 when goal=0
    int left = 0, sum = 0, result = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        sum += nums[right];
        while (sum > goal) {
            sum -= nums[left++];
        }
        result += right - left + 1;
    }
    return result;
}
```

**Why `if (goal < 0) return 0`?** When `goal = 0`, we call `atMost(nums, -1)`. A sum can never be ≤ -1 (array has non-negative values), so 0 is correct.

**Alternative — Prefix Sum + std::unordered_map:** `prefix[i] - prefix[j] = goal` → `count[prefix[j]] = count[prefix[i] - goal]`. Both O(n) but the sliding window is O(1) space.

---

## Problem 2: Count Number of Nice Subarrays — LC 1248

Count subarrays with exactly k odd numbers.

```cpp
#include <bits/stdc++.h>
using namespace std;

int numberOfSubarrays(vector<int>& nums, int k) {
    return atMost(nums, k) - atMost(nums, k - 1);
}

int atMost(vector<int>& nums, int k) {
    int left = 0, odds = 0, result = 0;
    for (int right = 0; right < (int)nums.size(); right++) {
        if (nums[right] % 2 == 1) odds++;
        while (odds > k) {
            if (nums[left] % 2 == 1) odds--;
            left++;
        }
        result += right - left + 1;
    }
    return result;
}
```

**Reduction insight:** `nums[i] % 2` transforms the problem to Binary Subarrays With Sum — both count binary values summing to k. The `atMost` template works identically.

---

## Problem 3: Subarray Product Less Than K — LC 713

Count subarrays where product is **strictly less than** k.

```cpp
#include <bits/stdc++.h>
using namespace std;

int numSubarrayProductLessThanK(vector<int>& nums, int k) {
    if (k <= 1) return 0;    // product ≥ 1 for positive arrays; nothing < 1 possible
    int left = 0, product = 1, result = 0;

    for (int right = 0; right < (int)nums.size(); right++) {
        product *= nums[right];

        while (product >= k) {     // product too large → shrink
            product /= nums[left++];
        }

        // All subarrays ending at 'right' starting in [left..right] have product < k
        result += right - left + 1;
    }
    return result;
}
```

**This is directly `atMost` with `< k`** — no subtraction needed because the condition is already "less than k" (a valid window condition). The `right - left + 1` counting is the same.

**Edge case:** `k <= 1` — since all `nums[i] >= 1`, any single element has product ≥ 1, which is not less than 1. Return 0.

**Trace for `[10, 5, 2, 6]`, k=100:**
```
right=0: product=10, [10], result += 1 = 1
right=1: product=50, [10,5],[5], result += 2 = 3
right=2: product=100 ≥ 100 → divide 10, left=1, product=10. [5,2],[2], result += 2 = 5
right=3: product=60, [5,2,6],[2,6],[6], result += 3 = 8
```
Output: 8 ✓

---

## When `atMost` Applies Directly (No Subtraction)

Some problems already have "less than k" or "at most k" in the constraint:

| Problem | Condition | Method |
|---------|-----------|--------|
| Subarray Product < K | product < k | Direct `atMost` counting |
| Binary Subarrays Sum = k | sum == k | `atMost(k) - atMost(k-1)` |
| Nice Subarrays exactly k | odds == k | `atMost(k) - atMost(k-1)` |
| Subarrays with at most k distinct | distinct ≤ k | Direct `atMost` counting |

---

## Prefix Sum Alternative for Exact-Count Problems

For binary arrays (0s and 1s), prefix sum + std::unordered_map is equivalent and handles the "exactly k" case directly:

```cpp
#include <bits/stdc++.h>
using namespace std;

// Binary Subarrays With Sum — prefix sum approach
int numSubarraysWithSum(vector<int>& nums, int goal) {
    unordered_map<int, int> prefixCount;
    prefixCount[0] = 1;   // empty prefix
    int sum = 0, result = 0;
    for (auto& num : nums) {
        sum += num;
        result += (prefixCount.count(sum - goal) ? prefixCount[sum - goal] : 0);
        prefixCount[sum]++;
    }
    return result;
}
```

**Trade-off:**
- `atMost` trick: O(n) time, O(1) space — but only works for problems reducible to sliding window
- Prefix sum: O(n) time, O(n) space — more universal (works for negative numbers, complex conditions)

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [Fixed Size Window](./Fixed%20Size%20Window.md)
- [Two Pointers](./Two%20Pointers.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
