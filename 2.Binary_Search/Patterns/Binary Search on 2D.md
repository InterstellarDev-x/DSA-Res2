# Binary Search on 2D Arrays

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Medium
> **Tags:** `binary-search` `2D-matrix` `row-wise-sorted` `staircase-search`

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
- Kth smallest in row-wise sorted matrix

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

## Rust Templates

### 1. Search in 2D Matrix — Type 1 (Flatten)

```rust
fn search_matrix(matrix: &Vec<Vec<i32>>, target: i32) -> bool {
    let m = matrix.len();
    let n = matrix[0].len();
    let mut lo = 0i32;
    let mut hi = (m * n) as i32 - 1;

    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        let val = matrix[(mid as usize) / n][(mid as usize) % n]; // map 1D index to 2D

        if val == target { return true; }
        else if val < target { lo = mid + 1; }
        else { hi = mid - 1; }
    }
    false
}
// Time: O(log(m*n)) | Space: O(1)
```

### 2. Search in 2D Matrix II — Type 2 (Staircase)

```rust
fn search_matrix(matrix: &Vec<Vec<i32>>, target: i32) -> bool {
    let mut row = 0i32;
    let mut col = matrix[0].len() as i32 - 1; // start top-right

    while row < matrix.len() as i32 && col >= 0 {
        if matrix[row as usize][col as usize] == target { return true; }
        else if matrix[row as usize][col as usize] > target { col -= 1; } // too big: move left
        else { row += 1; }                                                  // too small: move down
    }
    false
}
// Time: O(m+n) | Space: O(1)
// Eliminates one row or one column per step.
```

### 3. Median in Row-wise Sorted Matrix

```rust
fn count_less_equal(matrix: &Vec<Vec<i32>>, val: i32) -> i32 {
    let mut count = 0i32;
    for row in matrix {
        // Upper bound of val in this row
        let mut lo = 0usize;
        let mut hi = row.len();
        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if row[mid] <= val { lo = mid + 1; }
            else { hi = mid; }
        }
        count += lo as i32;
    }
    count
}

fn matrix_median(matrix: &Vec<Vec<i32>>) -> i32 {
    let m = matrix.len();
    let n = matrix[0].len();
    let mut lo = 1i32;
    let mut hi = 1_000_000_000i32;
    let desired = ((m * n + 1) / 2) as i32; // position of median

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        let count = count_less_equal(matrix, mid);
        if count < desired { lo = mid + 1; }
        else { hi = mid; }
    }
    lo
}
// Time: O(m * log(n) * log(max)) | Space: O(1)
```

### 4. Kth Smallest Element in Sorted Matrix

```rust
fn count_less_equal(matrix: &Vec<Vec<i32>>, mid: i32, n: usize) -> i32 {
    let mut count = 0i32;
    let mut row = n as i32 - 1;
    let mut col = 0i32;
    while row >= 0 && col < n as i32 {
        if matrix[row as usize][col as usize] <= mid {
            count += row + 1;
            col += 1;
        } else {
            row -= 1;
        }
    }
    count
}

fn kth_smallest(matrix: &Vec<Vec<i32>>, k: i32) -> i32 {
    let n = matrix.len();
    let mut lo = matrix[0][0];
    let mut hi = matrix[n - 1][n - 1];

    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        let count = count_less_equal(matrix, mid, n);
        if count < k { lo = mid + 1; }
        else { hi = mid; }
    }
    lo
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
