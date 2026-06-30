# Coding Tips — Binary Search

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Tips

---

## The Three Decisions Every Binary Search Requires

Before writing a single line of code, answer these three questions:

| Decision | Classic (find exact) | Lower Bound (first ≥) | BS on Answer (min feasible) |
|----------|---------------------|----------------------|----------------------------|
| **`lo` init** | `0` | `0` | `min_possible_answer` |
| **`hi` init** | `n-1` | `n` | `max_possible_answer` |
| **Loop condition** | `lo <= hi` | `lo < hi` | `lo < hi` |
| **When `f(mid)` is true** | `return mid` | `hi = mid` | `hi = mid` |
| **When `f(mid)` is false** | `lo = mid+1` or `hi = mid-1` | `lo = mid+1` | `lo = mid+1` |
| **Answer location** | returned during loop | `lo` after loop | `lo` after loop |

---

## Java Checklist

```java
// ✅ Always use this mid formula (prevents integer overflow)
int mid = lo + (hi - lo) / 2;

// ✅ For maximize problems, use upper-mid to prevent infinite loop
int mid = lo + (hi - lo + 1) / 2;

// ✅ Prefer hi = n (not n-1) for lower/upper bound templates
int hi = nums.length; // allows returning "not found" = n

// ✅ Verify target exists after lower bound
int lb = lowerBound(nums, target);
if (lb == nums.length || nums[lb] != target) return -1;

// ✅ Ceil division for feasibility checks
int hoursNeeded = (pile + speed - 1) / speed; // equivalent to Math.ceil(pile / speed)

// ✅ Use long for sums that may overflow
long hi = 0;
for (int w : weights) hi += w; // int sum of int[] can overflow
```

---

## Common Java Patterns

```java
// Lower bound (first index where arr[i] >= target)
private int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] < target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// Upper bound (first index where arr[i] > target)
private int upperBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] <= target) lo = mid + 1;
        else hi = mid;
    }
    return lo;
}

// Binary search on answer (minimize)
private int bsOnAnswer(int lo, int hi) {
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (isFeasible(mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Identifying Search Space](./Identifying%20Search%20Space.md)
- [Binary Search README](../README.md)

> **Last Updated:** 2026-06-26
