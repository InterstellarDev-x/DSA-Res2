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

## Rust Checklist

```rust
// ✅ Always use this mid formula (prevents integer overflow)
let mid = lo + (hi - lo) / 2;

// ✅ For maximize problems, use upper-mid to prevent infinite loop
let mid = lo + (hi - lo + 1) / 2;

// ✅ Prefer hi = n (not n-1) for lower/upper bound templates
let hi = nums.len(); // allows returning "not found" = n

// ✅ Verify target exists after lower bound
let lb = lower_bound(&nums, target);
if lb == nums.len() || nums[lb] != target { return -1; }

// ✅ Ceil division for feasibility checks
let hours_needed = (pile + speed - 1) / speed; // equivalent to ceil(pile / speed)

// ✅ Use i64 for sums that may overflow
let mut hi: i64 = 0;
for &w in &weights { hi += w as i64; } // i32 sum of Vec<i32> can overflow
```

---

## Common Rust Patterns

```rust
// Lower bound (first index where arr[i] >= target)
fn lower_bound(arr: &[i32], target: i32) -> usize {
    let mut lo = 0;
    let mut hi = arr.len();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] < target { lo = mid + 1; }
        else { hi = mid; }
    }
    lo
}

// Upper bound (first index where arr[i] > target)
fn upper_bound(arr: &[i32], target: i32) -> usize {
    let mut lo = 0;
    let mut hi = arr.len();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] <= target { lo = mid + 1; }
        else { hi = mid; }
    }
    lo
}

// Binary search on answer (minimize)
fn bs_on_answer(mut lo: i32, mut hi: i32) -> i32 {
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if is_feasible(mid) { hi = mid; }
        else { lo = mid + 1; }
    }
    lo
}
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Identifying Search Space](./Identifying%20Search%20Space.md)
- [Binary Search README](../README.md)

> **Last Updated:** 2026-06-26
