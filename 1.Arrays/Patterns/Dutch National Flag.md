# Dutch National Flag Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `three-pointers` `in-place` `partition` `sort`

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

The Dutch National Flag algorithm (Dijkstra, 1976) sorts an array of three distinct values in a single pass using **three pointers**:

```
[0..lo-1]  = section 1 (e.g., 0s)
[lo..mid-1]= section 2 (e.g., 1s)  ← current element
[mid..hi]  = unexplored
[hi+1..n-1]= section 3 (e.g., 2s)
```

**Pointer roles:**
- `lo` — boundary of section 1 (exclusive end)
- `mid` — current index scanning forward
- `hi` — boundary of section 3 (exclusive start)

---

## When to Use

- Sort array with exactly 3 distinct values in O(n)
- Partition array into three groups by some predicate
- Segregate negatives, zeros, positives
- Separate even/odd/specific-mod classes

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "sort array of 0s, 1s, 2s" | Classic DNF |
| "move all negatives to left, positives to right" | DNF (2-way partition) or Two Pointers |
| "group elements by three criteria" | DNF |
| "partition around a pivot" | DNF (QuickSort partition) |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Classic DNF sort | O(n) | O(1) |
| vs Counting Sort | O(n) | O(1) |
| vs Two-pass approach | O(n) | O(1) |

---

## C++ Templates

### 1. Classic Dutch National Flag (Sort Colors)

```cpp
#include <bits/stdc++.h>
using namespace std;

void sortColors(vector<int>& nums) {
    int lo = 0, mid = 0, hi = (int)nums.size() - 1;

    while (mid <= hi) {
        if (nums[mid] == 0) {
            swap(nums[lo++], nums[mid++]); // 0: goes to front, both advance
        } else if (nums[mid] == 1) {
            mid++;                         // 1: already in place
        } else {                           // nums[mid] == 2
            swap(nums[mid], nums[hi--]);   // 2: goes to back, DON'T advance mid
        }
    }
}
// Time: O(n) | Space: O(1) | Single pass
```

> **Why not advance `mid` when swapping with `hi`?**
> The element swapped from `hi` is unknown — it could be 0, 1, or 2. We must re-examine it.

### 2. Segregate Negatives and Positives (2-way)

```cpp
#include <bits/stdc++.h>
using namespace std;

void segregate(vector<int>& arr) {
    int lo = 0, hi = (int)arr.size() - 1;
    while (lo < hi) {
        while (lo < hi && arr[lo] < 0) lo++;
        while (lo < hi && arr[hi] >= 0) hi--;
        if (lo < hi) swap(arr[lo], arr[hi]);
    }
}
// Time: O(n) | Space: O(1)
```

### 3. Three-way Partition around Pivot (QuickSort)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Partition arr[l..r] around pivot value
pair<int,int> threeWayPartition(vector<int>& arr, int l, int r, int pivot) {
    int lo = l, mid = l, hi = r;
    while (mid <= hi) {
        if (arr[mid] < pivot)       swap(arr[lo++], arr[mid++]);
        else if (arr[mid] == pivot) mid++;
        else                        swap(arr[mid], arr[hi--]);
    }
    return {lo, hi}; // [lo, hi] is the range of pivot elements
}
```

### 4. Rearrange Positives and Negatives (Preserve Relative Order)

```cpp
#include <bits/stdc++.h>
using namespace std;

// This needs O(n) extra space to preserve relative order
void rearrange(vector<int>& arr) {
    int n = arr.size();
    vector<int> result(n);
    int pos = 0, neg = n / 2;
    // Assumes equal counts of positives and negatives
    for (auto& x : arr) {
        if (x >= 0) result[pos++] = x;
        else result[neg++] = x;
    }
    arr = result;
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Advancing `mid` after swap with `hi` | Do NOT increment `mid` — re-inspect the swapped element |
| Using counting sort when O(1) space required | DNF is always O(1) space |
| Not handling equal pivot correctly | `mid` advances only for equal elements |
| Confusing `lo++, mid++` vs `lo++` only | Both advance when element belongs to leftmost group |
| Off-by-one: `while (mid <= hi)` vs `mid < hi` | Must be `mid <= hi` to process element at `hi` |

---

## Variations

| Variation | Description |
|-----------|-------------|
| 4-way partition | Add a 4th category; use 4 pointers |
| Weighted partition | Sort by frequency within groups |
| Sort by parity | Even before odd (LC 905) |
| Sort array by sign alternating | Two separate DNF passes |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium | LC 75 |
| [Sort Array By Parity](https://leetcode.com/problems/sort-array-by-parity/) | Easy | LC 905 |
| [Sort Array By Parity II](https://leetcode.com/problems/sort-array-by-parity-ii/) | Easy | LC 922 |
| [Partition Array into Three Parts with Equal Sum](https://leetcode.com/problems/partition-array-into-three-parts-with-equal-sum/) | Easy | LC 1013 |
| [Wiggle Sort II](https://leetcode.com/problems/wiggle-sort-ii/) | Medium | LC 324 |
| Segregate 0s and 1s (Array) | Easy | GFG |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — DNF is a 3-pointer extension of Two Pointers
- [Sliding Window](./Sliding%20Window.md) — for contiguous subarray problems
- [Cyclic Sort](./Cyclic%20Sort.md) — for placing elements at exact correct indices

---

> **Interview Tip:** If the interviewer says "in one pass, O(1) space", and the array has three distinct categories → Dutch National Flag. Always walk through why `mid` doesn't advance when swapping with `hi`.

> **Last Updated:** 2026-06-26
