# Common Mistakes — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `interview-tips` `mistakes` `arrays` `debugging`

---

## Table of Contents

1. [Algorithm Mistakes](#algorithm-mistakes)
2. [Rust Implementation Mistakes](#rust-implementation-mistakes)
3. [Edge Case Blindspots](#edge-case-blindspots)
4. [Communication Mistakes](#communication-mistakes)
5. [Pattern-Specific Mistakes](#pattern-specific-mistakes)
6. [Related Files](#related-files)

---

## Algorithm Mistakes

| Mistake | Example | Fix |
|---------|---------|-----|
| Initializing max/min to 0 | `let mut max = 0_i32` fails for all-negative | Use `i32::MIN` or `nums[0]` |
| Applying sliding window to arrays with negative values | Sum can grow and shrink unpredictably | Use [Prefix Sum + HashMap](../Patterns/Prefix%20Sum.md) instead |
| Sorting when problem requires O(n) | Adds unnecessary O(n log n) | Check if `HashSet` / `HashMap` can achieve O(n) |
| Two pointer on unsorted array | Wrong pairs found | Sort first OR use `HashMap` |
| Not considering that sorting changes indices | Two Sum wants original indices | If indices matter, don't sort — use `HashMap` |
| Missing the "verify" step in Moore's Voting | Returns wrong candidate | Always verify when majority not guaranteed |

---

## Rust Implementation Mistakes

| Mistake | Bad Code | Fix |
|---------|----------|-----|
| Integer overflow | `let sum: i32 = a + b` where a, b ≈ 10^9 | Use `let sum: i64 = a as i64 + b as i64` |
| Off-by-one in prefix sum | `prefix[i] = arr[0..i]` | Use 1-indexed: `prefix[0]=0, prefix[i]=prefix[i-1]+arr[i-1]` |
| Modifying array during iteration | Remove while iterating `Vec` | Use index-based loop or collect separately |
| Comparing integers with wrong operator | `if a == b` where a, b are objects | In Rust, `==` works for all `PartialEq` types |
| Not initializing `i64` accumulator | `let sum: i32` overflows | `let mut sum: i64 = 0` |
| `sort()` on `Vec<i32>` with comparator | Wrong comparator signature | Use closure: `v.sort_by(\|a, b\| b.cmp(a))` |

---

## Edge Case Blindspots

| Edge Case | Why It Breaks | How to Test |
|-----------|--------------|-------------|
| Empty array `[]` | Out-of-bounds or wrong answer | Guard: `if arr.is_empty() { return ...; }` |
| Single element `[x]` | Two-pointer loop never runs | Trace with n=1 |
| All same elements `[2,2,2,2]` | Duplicate handling off | Verify cyclic sort, 3Sum de-dup logic |
| All negative `[-3,-1,-2]` | Max initialized to 0 is wrong | Initialize to `nums[0]` |
| Array of zeros `[0,0,0]` | Division by zero in product array | Handle zeros separately |
| K = 0 (window size) | Division by zero or empty window | Guard: `if k == 0 { return }` |
| K > array length | Out of bounds | Guard: `if k > n { k %= n }` |
| Integers at boundary | `i32::MIN - 1` overflow | Use `.abs()` carefully |

---

## Communication Mistakes

| Mistake | Impact | Fix |
|---------|--------|-----|
| Coding silently for 5+ minutes | Interviewer doesn't know your thinking | Narrate every decision |
| Not asking for clarification | Solving wrong problem | Ask: "Can the array be empty? Can values be negative?" |
| Jumping to optimal without brute force | Misses partial credit, seems rehearsed | Always state brute force first, then optimize |
| Not testing your own code | Bugs caught by interviewer | Do a manual trace with example |
| Saying "this is O(n)" without justification | Interviewer probes and finds weakness | Explain: "The while loop runs at most n times total because..." |

---

## Pattern-Specific Mistakes

### Sliding Window

```rust
// WRONG: Forgetting to shrink window
for r in 0..n {
    window += arr[r];
    // forgot: while window > k { window -= arr[l]; l += 1; }
    max_len = max_len.max(r - l + 1);
}

// WRONG: Using 'if' instead of 'while' to shrink
if window > k { window -= arr[l]; l += 1; } // misses multiple shrinks needed
```

### Prefix Sum

```rust
use std::collections::HashMap;
// WRONG: Missing seed
let mut freq: HashMap<i32, i32> = HashMap::new();
// forgot: freq.insert(0, 1); — misses subarrays starting at index 0
let mut sum = 0;
for &num in &nums { sum += num; count += freq.get(&(sum - k)).copied().unwrap_or(0); *freq.entry(sum).or_insert(0) += 1; }
```

### Cyclic Sort

```rust
// WRONG: Advancing i after every swap
while i < n {
    let c = (nums[i] - 1) as usize;
    if nums[i] != nums[c] { nums.swap(i, c); }
    i += 1; // BUG: should only advance when nums[i] == i + 1
}
```

### Dutch National Flag

```rust
// WRONG: Advancing mid when swapping with hi
} else { // nums[mid] == 2
    nums.swap(mid, hi); mid += 1; hi -= 1; // BUG: don't advance mid here
}
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Follow-up Questions](./Follow-up%20Questions.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
