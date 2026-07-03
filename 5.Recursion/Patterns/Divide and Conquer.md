# Divide & Conquer

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** Merge Sort, Quick Sort, Binary Search, Pow(x,n), K-th Grammar, Regex Matching

---

## Core Idea

1. **Divide:** Split the problem into 2+ non-overlapping subproblems of the same type.
2. **Conquer:** Recursively solve each subproblem (base case = trivially solvable).
3. **Combine:** Merge results of subproblems to form the solution.

```
solve(problem) {
    if (base case) return direct answer;
    left  = solve(left half);
    right = solve(right half);
    return combine(left, right);
}
```

**Master Theorem** for T(n) = aT(n/b) + f(n):
- f(n) = O(n^log_b(a) - ε): T(n) = O(n^log_b(a))
- f(n) = O(n^log_b(a)): T(n) = O(n^log_b(a) × log n)
- f(n) = O(n^log_b(a) + ε): T(n) = O(f(n))

---

## Template 1 — Merge Sort (LC 912)

T(n) = 2T(n/2) + O(n) → **O(n log n)**, Space O(n).

```rust
fn sort_array(nums: &mut Vec<i32>) -> Vec<i32> {
    let n = nums.len();
    if n > 1 {
        merge_sort(nums, 0, n - 1);
    }
    nums.clone()
}

fn merge_sort(arr: &mut Vec<i32>, lo: usize, hi: usize) {
    if lo >= hi { return; }
    let mid = lo + (hi - lo) / 2;
    merge_sort(arr, lo, mid);
    merge_sort(arr, mid + 1, hi);
    merge(arr, lo, mid, hi);
}

fn merge(arr: &mut Vec<i32>, lo: usize, mid: usize, hi: usize) {
    let left = arr[lo..=mid].to_vec();
    let right = arr[mid + 1..=hi].to_vec();
    let (mut i, mut j, mut k) = (0usize, 0usize, lo);
    while i < left.len() && j < right.len() {
        if left[i] <= right[j] {
            arr[k] = left[i];
            i += 1;
        } else {
            arr[k] = right[j];
            j += 1;
        }
        k += 1;
    }
    while i < left.len() { arr[k] = left[i]; i += 1; k += 1; }
    while j < right.len() { arr[k] = right[j]; j += 1; k += 1; }
}
```

**Stable sort:** Equal elements from left half appear before right half (`left[i] <= right[j]` — `<=` not `<`).

**Use for: Count Inversions** — count `j - mid - 1` inversions each time a right-half element is merged before left-half elements remain.

---

## Template 2 — Quick Sort

Average O(n log n), worst O(n²) (sorted input + bad pivot). Space O(log n) avg stack.

```rust
fn quick_sort(arr: &mut Vec<i32>, lo: usize, hi: usize) {
    if lo >= hi { return; }
    let p = partition(arr, lo, hi);
    if p > 0 { quick_sort(arr, lo, p - 1); }
    quick_sort(arr, p + 1, hi);
}

fn partition(arr: &mut Vec<i32>, lo: usize, hi: usize) -> usize {
    // Lomuto partition: pivot = arr[hi]
    let pivot = arr[hi];
    let mut i = lo;
    for j in lo..hi {
        if arr[j] <= pivot {
            arr.swap(i, j);
            i += 1;
        }
    }
    arr.swap(i, hi);
    i
}
```

**3-way partition (Dutch National Flag) for duplicates:**
```rust
fn partition_3way(arr: &mut Vec<i32>, lo: usize, hi: usize) -> (usize, usize) {
    let pivot = arr[lo];
    let (mut lt, mut gt, mut i) = (lo, hi, lo);
    while i <= gt {
        use std::cmp::Ordering;
        match arr[i].cmp(&pivot) {
            Ordering::Less => {
                arr.swap(lt, i);
                lt += 1;
                i += 1;
            }
            Ordering::Greater => {
                arr.swap(i, gt);
                if gt == 0 { break; }
                gt -= 1;
            }
            Ordering::Equal => {
                i += 1;
            }
        }
    }
    (lt, gt) // arr[lt..=gt] == pivot
}
```

---

## Template 3 — Pow(x, n) (LC 50)

T(n) = T(n/2) + O(1) → **O(log n)**, Space O(log n) stack.

```rust
fn my_pow(x: f64, n: i32) -> f64 {
    power(x, n as i64) // cast to i64 — handles i32::MIN
}

fn power(x: f64, n: i64) -> f64 {
    if n == 0 { return 1.0; }
    if n < 0 { return power(1.0 / x, -n); }
    let half = power(x, n / 2);
    if n % 2 == 0 { half * half } else { half * half * x }
}
```

**Iterative fast power (O(1) space):**
```rust
fn power(mut x: f64, mut n: i64) -> f64 {
    if n < 0 { x = 1.0 / x; n = -n; }
    let mut result = 1.0_f64;
    while n > 0 {
        if (n & 1) == 1 { result *= x; }
        x *= x;
        n >>= 1;
    }
    result
}
```

---

## Template 4 — K-th Symbol in Grammar (LC 779)

Row 1: `0`. Each `0` → `01`, each `1` → `10`. Find kth symbol (1-indexed) in row n.

```rust
fn kth_grammar(n: i32, k: i32) -> i32 {
    if n == 1 { return 0; }
    let parent = kth_grammar(n - 1, (k + 1) / 2);
    let is_left_child = k % 2 == 1;
    if is_left_child { parent } else { 1 - parent } // left = same, right = flipped
}
```

**Why:** Symbol k in row n is the left child of symbol ⌈k/2⌉ in row n-1 (if k is odd), or right child (if k is even). Left child = same as parent, right child = 1 - parent.

---

## Template 5 — Regular Expression Matching (LC 10)

Pattern: `.` matches any char; `*` matches zero or more of the preceding.

```rust
fn is_match(s: String, p: String) -> bool {
    let s: Vec<char> = s.chars().collect();
    let p: Vec<char> = p.chars().collect();
    let mut memo = vec![vec![-1i32; p.len() + 1]; s.len() + 1];
    match_helper(&s, &p, 0, 0, &mut memo)
}

fn match_helper(s: &[char], p: &[char], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> bool {
    if j == p.len() { return i == s.len(); }
    if memo[i][j] != -1 { return memo[i][j] == 1; }

    let first_match = i < s.len() && (p[j] == s[i] || p[j] == '.');
    let result;

    if j + 1 < p.len() && p[j + 1] == '*' {
        // Two choices for '*': use zero of p[j], or use one and stay on same pattern char
        result = match_helper(s, p, i, j + 2, memo)              // zero occurrences
              || (first_match && match_helper(s, p, i + 1, j, memo)); // one or more
    } else {
        result = first_match && match_helper(s, p, i + 1, j + 1, memo);
    }
    memo[i][j] = if result { 1 } else { 0 };
    result
}
```

---

## Merge Sort Extensions

| Problem | Key Addition to Merge Sort |
|---------|--------------------------|
| Count inversions | `inversions += (mid - i + 1)` when right element merges first |
| Count smaller numbers after self | Use merge sort on index array; count crosses |
| Sort colors (3-way) | Dutch National Flag on merge step |

---

## Divide & Conquer vs DP

| | D&C | DP |
|--|-----|----|
| Subproblem overlap | No overlap (independent halves) | Yes (recomputed many times) |
| Memoization needed | No | Yes (or tabulation) |
| Example | Merge Sort, Quick Sort | Fibonacci, Longest Common Subsequence |
| Space | O(log n) stack usually | O(n) or O(n²) table |

Recursion + overlapping subproblems → DP (memoize). Non-overlapping → pure D&C.

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Merge Sort | O(n log n) | O(n) aux |
| Quick Sort (avg) | O(n log n) | O(log n) |
| Quick Sort (worst) | O(n²) | O(n) |
| Pow(x, n) | O(log n) | O(log n) → O(1) iterative |
| K-th Grammar | O(n) | O(n) stack |
| Regex Match (memoized) | O(m × n) | O(m × n) |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| `mid = (lo + hi) / 2` — overflow for large indices | `lo + (hi - lo) / 2` |
| Pow: `-n` on `i32::MIN` overflows | Cast `n` to `i64` before negating |
| Merge: `left[i] < right[j]` instead of `<=` — unstable | Use `<=` for stable merge |
| Quick Sort: always picking first/last as pivot on sorted input → O(n²) | Random pivot or median-of-three |

---

## Related Patterns

- [Basic Recursion](./Basic%20Recursion.md) — simpler non-split recursion
- [Binary Search](../../2.Binary_Search/Patterns/Classic%20Binary%20Search.md) — D&C with single-branch pruning
- [Merge Linked Lists](../../4.Linked_List/Patterns/Merge%20Linked%20Lists.md) — merge step applied to linked lists

---

**Back:** [Recursion README](../README.md) | **Prev:** [Permutations](./Permutations.md)

> **Last Updated:** 2026-06-26
