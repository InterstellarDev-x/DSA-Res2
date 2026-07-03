# Common Mistakes — Binary Search

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Tips

---

## Top Bugs That Kill Binary Search Solutions

| # | Mistake | Buggy Code | Fix |
|---|---------|-----------|-----|
| 1 | Integer overflow in mid | `(lo + hi) / 2` | `lo + (hi - lo) / 2` |
| 2 | Infinite loop — `hi = mid - 1` when `hi = mid` needed | `if ok { hi = mid - 1; }` | `if ok { hi = mid; }` (mid is still a candidate) |
| 3 | Infinite loop — lower-mid with maximize template | `mid = lo + (hi-lo)/2; if ok { lo = mid; }` | `mid = lo + (hi-lo+1)/2` (upper-mid) |
| 4 | Off-by-one: `hi = n-1` for lower bound | `let hi = n - 1` | `let hi = n` — answer may be at index n ("not found") |
| 5 | Accessing `nums[mid-1]` without bounds check | `if nums[mid] == nums[mid - 1]` | Guard: `mid > 0 &&` |
| 6 | Wrong feasibility `lo`/`hi` | `lo = 0, hi = max` for capacity | `lo = *weights.iter().max().unwrap()` — must carry single heaviest item |
| 7 | Not verifying target exists after lower bound | `return lower_bound(&nums, target)` for exact search | Check `arr[lb] == target` before returning |
| 8 | `while lo <= hi` for lower bound template | Causes `lo > hi` — misses the invariant | Use `while lo < hi`; exit when `lo == hi` |

---

## Rotated Array Mistakes

```rust
// WRONG: not determining which half is sorted first
if nums[mid] > target { hi = mid - 1; } // incorrect for rotated array

// CORRECT: always check which half is sorted, then test if target is in that range
if nums[lo] <= nums[mid] { // left half sorted
    if nums[lo] <= target && target < nums[mid] { hi = mid - 1; }
    else { lo = mid + 1; }
}
```

## Feasibility Check Mistakes

```rust
// WRONG: using i32 sum that can overflow
let mut hi: i32 = 0;
for &w in &weights { hi += w; } // OVERFLOW if weights are large

// CORRECT:
let mut hi: i64 = 0;
for &w in &weights { hi += w as i64; }
```

---

## Mental Model Checklist

Before submitting any binary search:

```
□ Can lo == mid? (if lo+1 == hi, lower-mid gives lo — check for infinite loop)
□ Is mid ever equal to hi? (inclusive template: yes; half-open: no)
□ After the loop, does lo point to the answer?
□ Is the feasibility check O(n) or better?
□ Did I handle the case where no answer exists?
□ Integer overflow in mid? sum? product?
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Identifying Search Space](./Identifying%20Search%20Space.md)
- [Binary Search README](../README.md)

> **Last Updated:** 2026-06-26
