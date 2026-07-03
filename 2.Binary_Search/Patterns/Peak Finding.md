# Peak Finding Pattern

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Medium
> **Tags:** `peak-element` `mountain-array` `binary-search` `unimodal`

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

A **peak element** is one that is strictly greater than its neighbours. Peak Finding uses binary search on the **slope direction** — if the element to the right is larger, the peak must be to the right (we're on an upslope). If smaller, the peak is to the left (or at mid).

**Core Invariant:**
> If `nums[mid] < nums[mid+1]`, there is always a peak in `[mid+1, hi]`.
> If `nums[mid] > nums[mid+1]`, there is always a peak in `[lo, mid]`.

This works even for arrays with multiple peaks — we find **any one peak**.

```
Array: [1, 3, 5, 4, 2]
            ↑ peak at index 2 (5 > 4 and 5 > 3)

Array: [1, 2, 1, 3, 5, 6, 4]
                        ↑ peak at index 5 (6 > 5 and 6 > 4)
```

---

## When to Use

- Find any peak element (local maximum)
- Mountain array: find the peak to split into two sorted halves
- Find maximum in unimodal function (bitonic)
- Find summit / transition point

---

## Recognition Cues

| Cue | Variant |
|-----|---------|
| "find a peak element, O(log n)" | Classic peak finding |
| "mountain array: first increases then decreases" | Mountain peak — same algorithm |
| "find index of maximum in unimodal array" | Same as peak |
| "search in mountain array" | Find peak, then binary search each half |
| "2D peak element" | Binary search on rows, linear scan on col |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| 1D peak finding | O(log n) | O(1) |
| 2D peak finding | O(m log n) | O(1) |
| Search in mountain array | O(log n) | O(1) |

---

## C++ Templates

### 1. Find Peak Element (any peak)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findPeakElement(vector<int>& nums) {
    int lo = 0, hi = nums.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] < nums[mid + 1])
            lo = mid + 1; // ascending slope: peak to the right
        else
            hi = mid;     // descending or at peak: peak at mid or left
    }
    return lo; // lo == hi == peak index
}
// Time: O(log n) | Space: O(1)
// Works for any array, guaranteed ≥1 peak exists
// (assumes nums[-1] = nums[n] = -∞ at boundaries)
```

### 2. Mountain Array Peak Index

```cpp
#include <bits/stdc++.h>
using namespace std;

// Array strictly increases then strictly decreases
int peakIndexInMountainArray(vector<int>& arr) {
    int lo = 0, hi = arr.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < arr[mid + 1])
            lo = mid + 1; // still on upslope
        else
            hi = mid;     // on downslope or at peak
    }
    return lo;
}
// Time: O(log n) | Space: O(1)
```

### 3. Search in Mountain Array (search both halves)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findPeak(vector<int>& arr) {
    int lo = 0, hi = arr.size() - 1;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < arr[mid + 1]) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

int binarySearch(vector<int>& arr, int lo, int hi, int target, bool ascending) {
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        if (ascending) {
            if (arr[mid] < target) lo = mid + 1; else hi = mid - 1;
        } else {
            if (arr[mid] > target) lo = mid + 1; else hi = mid - 1;
        }
    }
    return -1;
}

int findInMountainArray(int target, vector<int>& mountainArr) {
    int peak = findPeak(mountainArr);

    // Search in ascending half [0, peak]
    int idx = binarySearch(mountainArr, 0, peak, target, true);
    if (idx != -1) return idx;

    // Search in descending half [peak+1, n-1]
    return binarySearch(mountainArr, peak + 1, mountainArr.size() - 1, target, false);
}
// Time: O(log n) | Space: O(1)
```

### 4. Find Peak Element II (2D Matrix)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Each row is a separate 1D problem; binary search on columns
vector<int> findPeakGrid(vector<vector<int>>& mat) {
    int lo = 0, hi = mat[0].size() - 1;
    while (lo <= hi) {
        int midCol = lo + (hi - lo) / 2;
        int maxRow = 0;
        // Find row with max element in midCol
        for (int r = 0; r < (int)mat.size(); r++)
            if (mat[r][midCol] > mat[maxRow][midCol]) maxRow = r;

        bool leftBigger  = midCol > 0 && mat[maxRow][midCol - 1] > mat[maxRow][midCol];
        bool rightBigger = midCol < (int)mat[0].size() - 1 && mat[maxRow][midCol + 1] > mat[maxRow][midCol];

        if (!leftBigger && !rightBigger) return vector<int>{maxRow, midCol};
        else if (rightBigger) lo = midCol + 1;
        else hi = midCol - 1;
    }
    return vector<int>{-1, -1};
}
// Time: O(m log n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Loop condition `lo <= hi` (inclusive) | Use `lo < hi` — loop ends when `lo == hi` (single element = peak) |
| Checking `nums[mid] > nums[mid-1]` instead of `nums[mid+1]` | Always compare `mid` with `mid+1` — accessing `mid-1` risks out-of-bounds |
| Not handling single-element array | Works naturally — `lo = hi = 0`, no loop iterations |
| Mountain array: confused with "find min in rotated" | Both are O(log n) but different invariants — mountain goes up then down, rotated goes up then down then up |
| Thinking there's a unique peak | Problem only asks for **any** peak — return first one found |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Local minimum | Flip comparison: go where `nums[mid] > nums[mid+1]` |
| Strictly unimodal continuous function | Ternary search (not binary) |
| Peak in circular array | Modify boundary conditions |
| All same elements | No true peak — problem usually excludes this case |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Find Peak Element](https://leetcode.com/problems/find-peak-element/) | Medium | LC 162 |
| [Peak Index in a Mountain Array](https://leetcode.com/problems/peak-index-in-a-mountain-array/) | Medium | LC 852 |
| [Find in Mountain Array](https://leetcode.com/problems/find-in-mountain-array/) | Hard | LC 1095 |
| [Valid Mountain Array](https://leetcode.com/problems/valid-mountain-array/) | Easy | LC 941 |
| [Find a Peak Element II](https://leetcode.com/problems/find-a-peak-element-ii/) | Hard | LC 1901 |
| [Longest Mountain in Array](https://leetcode.com/problems/longest-mountain-in-array/) | Medium | LC 845 |

---

## Related Patterns

- [Classic Binary Search](./Classic%20Binary%20Search.md) — same template, different condition
- [Binary Search on 2D](./Binary%20Search%20on%202D.md) — 2D peak extends this
- [Binary Search on Answer](./Binary%20Search%20on%20Answer.md) — searching on value space

---

> **Interview Tip:** The question "why does binary search work here even if there are multiple peaks?" is a common follow-up. Answer: the invariant guarantees a peak always exists in the remaining range. If `nums[mid] < nums[mid+1]`, the right side has an upslope — even if there are valleys, it must eventually peak (since the array ends at `-∞`).

> **Last Updated:** 2026-06-26
