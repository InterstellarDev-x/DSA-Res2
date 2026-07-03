# Coding Tips — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `interview-tips` `coding` `arrays` `rust`

---

## Table of Contents

1. [Before You Code](#before-you-code)
2. [Rust-Specific Tips](#rust-specific-tips)
3. [Array Manipulation Tricks](#array-manipulation-tricks)
4. [Complexity Shortcuts](#complexity-shortcuts)
5. [Checklist Before Submitting](#checklist-before-submitting)
6. [Related Files](#related-files)

---

## Before You Code

1. **Read the full problem** — Don't start until you know input/output format, constraints, and return type.
2. **Ask clarifying questions:**
   - Are values positive/negative/zero?
   - Can the array be empty? Single element?
   - Can values repeat? Are they sorted?
   - What should be returned if no answer exists?
3. **Write examples by hand** — Pick an example, trace through it before coding.
4. **State your approach aloud** — "I'll use prefix sum + hashmap. Here's why..."
5. **State complexity upfront** — "This will be O(n) time and O(n) space."

---

## Rust-Specific Tips

### Sorting

```rust
arr.sort();                                                                                // O(n log n) introsort
intervals.sort_by(|a, b| a[0].cmp(&b[0]));                                               // sort by first element
arr.sort_by(|a, b| b.cmp(a));                                                             // reverse sort — works on Vec<i32>
```

### Copy & Fill

```rust
let copy = arr.clone();                                                // full copy
let sub = arr[l..=r].to_vec();                                        // [l, r] inclusive
arr.fill(0);                                                           // fill entire array
arr[l..=r].fill(-1);                                                   // fill [l, r]
```

### Two-Pointer Swap

```rust
fn swap_arr(arr: &mut Vec<i32>, i: usize, j: usize) {
    arr.swap(i, j);
}
```

### Common Initializations

```rust
let mut max_val = i32::MIN;  // not 0! (handles all-negative arrays)
let mut min_val = i32::MAX;
let mut sum: i32 = 0;        // use i64 if values can overflow
```

### 2D Array

```rust
let matrix: Vec<Vec<i32>> = vec![vec![0; n]; m];
let rows = matrix.len();
let cols = matrix[0].len();
```

---

## Array Manipulation Tricks

| Operation | Code | Time |
|-----------|------|------|
| Reverse array | Two pointer from ends | O(n) |
| Rotate right by k | Three reverses | O(n) |
| Check palindrome | Two pointer | O(n) |
| Move zeros to end | Slow/fast pointer | O(n) |
| Remove element in-place | Slow/fast pointer | O(n) |
| Prefix sum | Single pass, store in extra array | O(n) |
| Suffix max | Right to left pass | O(n) |

### Rotate Array by K (Three Reversal Trick)

```rust
// Rotate right by k: [1,2,3,4,5], k=2 → [4,5,1,2,3]
k %= n;
arr.reverse();                             // [5,4,3,2,1]
arr[..k].reverse();                        // [4,5,3,2,1]
arr[k..].reverse();                        // [4,5,1,2,3]
```

### In-Place Matrix Transpose

```rust
// Transpose square matrix: arr[i][j] ↔ arr[j][i]
for i in 0..n {
    for j in (i + 1)..n {
        let val_ij = matrix[i][j];
        let val_ji = matrix[j][i];
        matrix[i][j] = val_ji;
        matrix[j][i] = val_ij;
    }
}
// Then reverse each row for 90° clockwise rotation
```

---

## Complexity Shortcuts

| Pattern | Time | Space |
|---------|------|-------|
| [Prefix Sum](../Patterns/Prefix%20Sum.md) | O(n) | O(n) |
| [Sliding Window](../Patterns/Sliding%20Window.md) | O(n) | O(1) or O(k) |
| [Two Pointers](../Patterns/Two%20Pointers.md) | O(n) or O(n²) for 3Sum | O(1) |
| [Kadane's](../Patterns/Kadane's%20Algorithm.md) | O(n) | O(1) |
| Sort + scan | O(n log n) | O(1) |
| [Cyclic Sort](../Patterns/Cyclic%20Sort.md) | O(n) | O(1) |
| [Dutch National Flag](../Patterns/Dutch%20National%20Flag.md) | O(n) | O(1) |

---

## Checklist Before Submitting

```
□ Empty array handled?
□ Single element array handled?
□ All same elements handled?
□ Negative values handled (if applicable)?
□ Integer overflow possible? (use i64)
□ Index out of bounds impossible?
□ Time complexity stated?
□ Space complexity stated?
□ All return paths correct?
□ Test with 2–3 examples mentally
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Dry Run Technique](./Dry%20Run.md)
- [Communication Tips](./Communication.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
