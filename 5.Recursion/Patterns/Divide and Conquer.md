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

```java
public int[] sortArray(int[] nums) {
    mergeSort(nums, 0, nums.length - 1);
    return nums;
}

private void mergeSort(int[] arr, int lo, int hi) {
    if (lo >= hi) return;
    int mid = lo + (hi - lo) / 2;
    mergeSort(arr, lo, mid);
    mergeSort(arr, mid + 1, hi);
    merge(arr, lo, mid, hi);
}

private void merge(int[] arr, int lo, int mid, int hi) {
    int[] tmp = new int[hi - lo + 1];
    int i = lo, j = mid + 1, k = 0;
    while (i <= mid && j <= hi) {
        if (arr[i] <= arr[j]) tmp[k++] = arr[i++];
        else                  tmp[k++] = arr[j++];
    }
    while (i <= mid) tmp[k++] = arr[i++];
    while (j <= hi)  tmp[k++] = arr[j++];
    System.arraycopy(tmp, 0, arr, lo, tmp.length);
}
```

**Stable sort:** Equal elements from left half appear before right half (`arr[i] <= arr[j]` — `<=` not `<`).

**Use for: Count Inversions** — count `j - mid - 1` inversions each time a right-half element is merged before left-half elements remain.

---

## Template 2 — Quick Sort

Average O(n log n), worst O(n²) (sorted input + bad pivot). Space O(log n) avg stack.

```java
public void quickSort(int[] arr, int lo, int hi) {
    if (lo >= hi) return;
    int p = partition(arr, lo, hi);
    quickSort(arr, lo, p - 1);
    quickSort(arr, p + 1, hi);
}

private int partition(int[] arr, int lo, int hi) {
    // Lomuto partition: pivot = arr[hi]
    int pivot = arr[hi], i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (arr[j] <= pivot) swap(arr, ++i, j);
    }
    swap(arr, i + 1, hi);
    return i + 1;
}
```

**3-way partition (Dutch National Flag) for duplicates:**
```java
private int[] partition3Way(int[] arr, int lo, int hi) {
    int pivot = arr[lo], lt = lo, gt = hi, i = lo;
    while (i <= gt) {
        if      (arr[i] < pivot)  swap(arr, lt++, i++);
        else if (arr[i] > pivot)  swap(arr, i, gt--);
        else                      i++;
    }
    return new int[]{lt, gt}; // arr[lt..gt] == pivot
}
```

---

## Template 3 — Pow(x, n) (LC 50)

T(n) = T(n/2) + O(1) → **O(log n)**, Space O(log n) stack.

```java
public double myPow(double x, int n) {
    return power(x, (long) n); // cast to long — handles Integer.MIN_VALUE
}

private double power(double x, long n) {
    if (n == 0) return 1.0;
    if (n < 0)  return power(1.0 / x, -n);
    double half = power(x, n / 2);
    return (n % 2 == 0) ? half * half : half * half * x;
}
```

**Iterative fast power (O(1) space):**
```java
private double power(double x, long n) {
    if (n < 0) { x = 1.0 / x; n = -n; }
    double result = 1.0;
    while (n > 0) {
        if ((n & 1) == 1) result *= x;
        x *= x;
        n >>= 1;
    }
    return result;
}
```

---

## Template 4 — K-th Symbol in Grammar (LC 779)

Row 1: `0`. Each `0` → `01`, each `1` → `10`. Find kth symbol (1-indexed) in row n.

```java
public int kthGrammar(int n, int k) {
    if (n == 1) return 0;
    int parent = kthGrammar(n - 1, (k + 1) / 2);
    boolean isLeftChild = (k % 2 == 1);
    return isLeftChild ? parent : (1 - parent); // left = same, right = flipped
}
```

**Why:** Symbol k in row n is the left child of symbol ⌈k/2⌉ in row n-1 (if k is odd), or right child (if k is even). Left child = same as parent, right child = 1 - parent.

---

## Template 5 — Regular Expression Matching (LC 10)

Pattern: `.` matches any char; `*` matches zero or more of the preceding.

```java
public boolean isMatch(String s, String p) {
    return match(s, p, 0, 0, new Boolean[s.length() + 1][p.length() + 1]);
}

private boolean match(String s, String p, int i, int j, Boolean[][] memo) {
    if (j == p.length()) return i == s.length();
    if (memo[i][j] != null) return memo[i][j];

    boolean firstMatch = (i < s.length()) && (p.charAt(j) == s.charAt(i) || p.charAt(j) == '.');
    boolean result;

    if (j + 1 < p.length() && p.charAt(j + 1) == '*') {
        // Two choices for '*': use zero of p[j], or use one and stay on same pattern char
        result = match(s, p, i, j + 2, memo)              // zero occurrences
              || (firstMatch && match(s, p, i + 1, j, memo)); // one or more
    } else {
        result = firstMatch && match(s, p, i + 1, j + 1, memo);
    }
    return memo[i][j] = result;
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
| Pow: `-n` on `Integer.MIN_VALUE` overflows | Cast `n` to `long` before negating |
| Merge: `arr[i] < arr[j]` instead of `<=` — unstable | Use `<=` for stable merge |
| Quick Sort: always picking first/last as pivot on sorted input → O(n²) | Random pivot or median-of-three |

---

## Related Patterns

- [Basic Recursion](./Basic%20Recursion.md) — simpler non-split recursion
- [Binary Search](../../2.Binary_Search/Patterns/Classic%20Binary%20Search.md) — D&C with single-branch pruning
- [Merge Linked Lists](../../4.Linked_List/Patterns/Merge%20Linked%20Lists.md) — merge step applied to linked lists

---

**Back:** [Recursion README](../README.md) | **Prev:** [Permutations](./Permutations.md)

> **Last Updated:** 2026-06-26
