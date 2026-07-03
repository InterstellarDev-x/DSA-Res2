# Binary Search on 2D Arrays

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Medium
> **Tags:** `binary-search` `2D-matrix` `row-wise-sorted` `staircase-search`

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

Two types of sorted 2D matrices require different strategies:

**Type 1 — Fully Sorted (LC 74):**
- Each row is sorted left to right
- First element of each row > last element of previous row
- Can treat as 1D sorted array of size `m×n`
- Use standard binary search mapping `mid → (mid/n, mid%n)`

**Type 2 — Row+Column Sorted (LC 240):**
- Each row sorted left to right
- Each column sorted top to bottom
- **Cannot** flatten to 1D — overlapping row/col structure
- Use **Staircase Search**: start from top-right corner

```
Type 1 (fully sorted):        Type 2 (row+col sorted):
1  3  5  7                    1  4  7  11
10 11 16 20                   2  5  8  12
23 30 34 60                   3  6  9  16
→ Treat as [1,3,5,7,10,...]   → Staircase from top-right
```

---

## When to Use

- Matrix where rows/columns are sorted
- Finding a target in O(log(m×n)) or O(m+n)
- Median finding in row-wise sorted matrix
- Kth smallest in sorted matrix

---

## Recognition Cues

| Cue | Strategy |
|-----|----------|
| "m×n matrix, rows sorted, first of row > last of prev" | Flatten + binary search |
| "each row and column sorted" | Staircase search (top-right) |
| "median in row-wise sorted matrix" | Binary search on value range |
| "Kth smallest in sorted matrix" | Binary search on value range |

---

## Complexity

| Algorithm | Time | Space |
|-----------|------|-------|
| Flatten + binary search (Type 1) | O(log(m×n)) | O(1) |
| Staircase search (Type 2) | O(m+n) | O(1) |
| Median in row-wise sorted | O(m × log n × log(max-min)) | O(1) |
| Kth smallest (binary search) | O((m+n) × log(max-min)) | O(1) |

---

## C++ Templates

### 1. Search in 2D Matrix — Type 1 (Flatten)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int m = matrix.size(), n = matrix[0].size();
    int lo = 0, hi = m * n - 1;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int val = matrix[mid / n][mid % n]; // map 1D index to 2D

        if (val == target) return true;
        else if (val < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return false;
}
// Time: O(log(m*n)) | Space: O(1)
```

### 2. Search in 2D Matrix II — Type 2 (Staircase)

```cpp
#include <bits/stdc++.h>
using namespace std;

bool searchMatrix(vector<vector<int>>& matrix, int target) {
    int row = 0, col = matrix[0].size() - 1; // start top-right

    while (row < (int)matrix.size() && col >= 0) {
        if (matrix[row][col] == target) return true;
        else if (matrix[row][col] > target) col--; // too big: move left
        else row++;                                  // too small: move down
    }
    return false;
}
// Time: O(m+n) | Space: O(1)
// Eliminates one row or one column per step.
```

### 3. Median in Row-wise Sorted Matrix

```cpp
#include <bits/stdc++.h>
using namespace std;

int countLessEqual(vector<vector<int>>& matrix, int val) {
    int count = 0;
    for (auto& row : matrix) {
        // Upper bound of val in this row
        int lo = 0, hi = row.size();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (row[mid] <= val) lo = mid + 1;
            else hi = mid;
        }
        count += lo;
    }
    return count;
}

int matrixMedian(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    int lo = 1, hi = (int) 1e9;
    int desired = (m * n + 1) / 2; // position of median

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessEqual(matrix, mid);
        if (count < desired) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
// Time: O(m * log(n) * log(max)) | Space: O(1)
```

### 4. Kth Smallest Element in Sorted Matrix

```cpp
#include <bits/stdc++.h>
using namespace std;

int countLessEqual(vector<vector<int>>& matrix, int mid, int n) {
    int count = 0, row = n - 1, col = 0;
    while (row >= 0 && col < n) {
        if (matrix[row][col] <= mid) { count += row + 1; col++; }
        else row--;
    }
    return count;
}

int kthSmallest(vector<vector<int>>& matrix, int k) {
    int n = matrix.size();
    int lo = matrix[0][0], hi = matrix[n-1][n-1];

    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        int count = countLessEqual(matrix, mid, n);
        if (count < k) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}
// Time: O((m+n) * log(max-min)) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Type 1: `mid % cols` using rows instead of cols | Map: `row = mid / n`, `col = mid % n` where `n = cols` |
| Type 2: starting at bottom-left instead of top-right | Either works: top-right eliminates col (left) or row (down) |
| Staircase: going out of bounds | Guard: `row < m && col >= 0` |
| Median: treating all values as present in array | Binary search on value range, not on indices |
| Kth smallest: using a heap when O(1) space needed | Binary search on value + count approach |

---

## Variations

| Variation | Strategy |
|-----------|----------|
| Count negatives in sorted matrix | Staircase from bottom-left, O(m+n) |
| Row with max 1s (binary 0/1 matrix) | Binary search in each row for first 1, O(m log n) |
| Lucky numbers in matrix | Row min + col max check |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium | LC 74 |
| [Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) | Medium | LC 240 |
| [Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | LC 378 |
| [Find Peak Element II](https://leetcode.com/problems/find-a-peak-element-ii/) | Hard | LC 1901 |
| Median in Row-wise Sorted Matrix | Hard | GFG |
| Count Negatives in Sorted Matrix | Easy | LC 1351 |

---

## Related Patterns

- [Classic Binary Search](./Classic%20Binary%20Search.md) — Type 1 flattens to 1D
- [Binary Search on Answer](./Binary%20Search%20on%20Answer.md) — Median/Kth search on value range
- [Two Pointers](../../1.Arrays/Patterns/Two%20Pointers.md) — Staircase search is two-pointer variant

---

> **Interview Tip:** Always ask: "Is the first element of each row > last element of previous row?" If yes → flatten (O(log mn)). If only rows and columns are individually sorted → staircase (O(m+n)). These are two completely different algorithms.

> **Last Updated:** 2026-06-26
