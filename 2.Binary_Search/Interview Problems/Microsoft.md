# Microsoft — Binary Search Interview Questions

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Problems
> **Tags:** `microsoft` `interview` `binary-search`

---

## Frequently Asked Questions

### 1. Search in Rotated Sorted Array

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, R1 |
| **Pattern** | [Classic BS](../Patterns/Classic%20Binary%20Search.md) |
| **LeetCode** | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| **Follow-ups** | Duplicates? (LC 81) · Find min? (LC 153) |

---

### 2. First and Last Position of Element

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1 |
| **Pattern** | [Lower/Upper Bound](../Patterns/Lower%20and%20Upper%20Bound.md) |
| **LeetCode** | [LC 34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| **Follow-ups** | Count of occurrences · What if unsorted? |

```cpp
#include <bits/stdc++.h>
using namespace std;

// Demonstrate clean decomposition
vector<int> searchRange(vector<int>& nums, int target) {
    int first = lowerBound(nums, target);
    if (first == (int)nums.size() || nums[first] != target) return {-1, -1};
    return {first, upperBound(nums, target) - 1};
}
```

---

### 3. Single Element in Sorted Array

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1, R2 |
| **Pattern** | [Classic BS](../Patterns/Classic%20Binary%20Search.md) — XOR index parity |
| **LeetCode** | [LC 540](https://leetcode.com/problems/single-element-in-a-sorted-array/) |
| **Follow-ups** | "Can you explain the even-index invariant?" |

---

### 4. Search a 2D Matrix

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1 |
| **Pattern** | [BS on 2D](../Patterns/Binary%20Search%20on%202D.md) |
| **LeetCode** | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) |
| **Follow-ups** | "What if only rows and columns are sorted, not the whole matrix?" → LC 240 |

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **Correct template** | `lo <= hi` inclusive vs `lo < hi` half-open |
| **Mid overflow** | `lo + (hi-lo)/2` |
| **Boundary verification** | Walk through first and last iteration |
| **Clean helpers** | `lowerBound`/`upperBound` as separate methods |

---

## Related Files

- [Microsoft OA Questions](../OA-Qns/Microsoft.md)
- [Classic Binary Search Pattern](../Patterns/Classic%20Binary%20Search.md)
- [Lower and Upper Bound Pattern](../Patterns/Lower%20and%20Upper%20Bound.md)

> **Last Updated:** 2026-06-26
