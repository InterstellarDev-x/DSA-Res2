# Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Pattern 4 of 4**
> **Problems:** Container with Most Water · 3Sum · Move Zeroes · Sort Colors · Remove Duplicates · Trapping Rain Water (see Stacks topic)

---

## Core Concept

Two-pointer problems use two indices that move toward each other or in the same direction to reduce O(n²) brute force to O(n).

**Two variants:**

| Variant | Pointer Direction | Typical Use |
|---------|-----------------|-------------|
| **Opposite ends** | `left` starts at 0, `right` at n-1, move toward each other | Sum problems, max area, palindrome |
| **Same direction** | Both start at 0, `fast` moves ahead of `slow` | Remove elements, partition, detect cycle |

---

## Template 1: Opposite Ends

```rust
let mut left = 0usize;
let mut right = nums.len() - 1;
while left < right {
    if condition {
        // process or record answer
        left += 1;
    } else {
        right -= 1;
    }
}
```

---

## Template 2: Same Direction (Fast/Slow)

```rust
let mut slow = 0usize;
for fast in 0..nums.len() {
    if should_keep(nums[fast]) {
        nums[slow] = nums[fast];
        slow += 1;
    }
}
// slow = new length
```

---

## Problem 1: Container With Most Water — LC 11

Find two lines forming a container that holds the most water.

```rust
fn max_area(height: &[i32]) -> i32 {
    let mut left = 0usize;
    let mut right = height.len() - 1;
    let mut max_water = 0;

    while left < right {
        let water = height[left].min(height[right]) * (right - left) as i32;
        max_water = max_water.max(water);

        // Move the shorter line — moving the taller one can only decrease width
        // and can't increase the bounded height (still limited by the shorter)
        if height[left] <= height[right] {
            left += 1;
        } else {
            right -= 1;
        }
    }
    max_water
}
```

**Proof of greedy choice:** When `height[left] ≤ height[right]`, the current container is bounded by `height[left]`. Moving `right` inward decreases width AND keeps the bound at `height[left]` or worse. We can only potentially improve by moving `left` to find a taller left wall.

**Complexity:** O(n) time, O(1) space

---

## Problem 2: 3Sum — LC 15

Find all unique triplets summing to zero.

```rust
fn three_sum(nums: &mut Vec<i32>) -> Vec<Vec<i32>> {
    nums.sort();
    let mut result = Vec::new();
    let n = nums.len();

    for i in 0..n.saturating_sub(2) {
        if i > 0 && nums[i] == nums[i - 1] { continue; } // skip duplicate i

        let mut left = i + 1;
        let mut right = n - 1;
        while left < right {
            let sum = nums[i] + nums[left] + nums[right];
            if sum == 0 {
                result.push(vec![nums[i], nums[left], nums[right]]);
                while left < right && nums[left] == nums[left + 1] { left += 1; }   // skip dup left
                while left < right && nums[right] == nums[right - 1] { right -= 1; } // skip dup right
                left += 1;
                right -= 1;
            } else if sum < 0 {
                left += 1;
            } else {
                right -= 1;
            }
        }
    }
    result
}
```

**Complexity:** O(n²) time — outer loop O(n), inner two-pointer O(n). O(log n) space for sort.

**Duplicate skipping logic:**
- Outer: `i > 0 && nums[i] == nums[i-1]` — skip if same as previous `i` (not `i > 0` alone — we need to allow `i=0`)
- Inner: After finding a valid triplet, skip duplicates for both left and right before advancing

---

## Problem 3: Move Zeroes — LC 283

Move all zeroes to the end while maintaining order of non-zero elements.

```rust
fn move_zeroes(nums: &mut Vec<i32>) {
    let mut slow = 0usize; // position to write next non-zero
    for fast in 0..nums.len() {
        if nums[fast] != 0 {
            nums[slow] = nums[fast];
            slow += 1;
        }
    }
    // Fill rest with zeroes
    while slow < nums.len() {
        nums[slow] = 0;
        slow += 1;
    }
}
```

**In-place swap variant (fewer writes):**
```rust
fn move_zeroes(nums: &mut Vec<i32>) {
    let mut slow = 0usize;
    for fast in 0..nums.len() {
        if nums[fast] != 0 {
            nums.swap(slow, fast);
            slow += 1;
        }
    }
}
```

---

## Problem 4: Sort Colors (Dutch National Flag) — LC 75

Sort array of 0s, 1s, 2s in one pass.

```rust
fn sort_colors(nums: &mut Vec<i32>) {
    if nums.is_empty() { return; }
    let mut low = 0usize;
    let mut mid = 0usize;
    let mut high = nums.len() - 1;

    while mid <= high {
        if nums[mid] == 0 {
            nums.swap(low, mid);
            low += 1;
            mid += 1;
        } else if nums[mid] == 1 {
            mid += 1;
        } else { // nums[mid] == 2
            nums.swap(mid, high);
            // Don't increment mid — swapped element needs to be examined
            if high == 0 { break; }
            high -= 1;
        }
    }
}
```

**Three regions:** `[0..low-1]` = 0s, `[low..mid-1]` = 1s, `[mid..high]` = unexamined, `[high+1..n-1]` = 2s.

**Why not `mid += 1` when swapping with `high`?** The element swapped from `high` to `mid` is unexamined — we don't know if it's a 0, 1, or 2. We must examine it before advancing.

---

## Problem 5: Remove Duplicates from Sorted Array — LC 26

```rust
fn remove_duplicates(nums: &mut Vec<i32>) -> usize {
    if nums.is_empty() { return 0; }
    let mut slow = 1usize;
    for fast in 1..nums.len() {
        if nums[fast] != nums[fast - 1] {
            nums[slow] = nums[fast];
            slow += 1;
        }
    }
    slow
}
```

**Variant — keep at most 2 duplicates (LC 80):**
```rust
fn remove_duplicates(nums: &mut Vec<i32>) -> usize {
    let mut slow = 2usize;
    for fast in 2..nums.len() {
        if nums[fast] != nums[slow - 2] { // compare with position 2 back
            nums[slow] = nums[fast];
            slow += 1;
        }
    }
    slow
}
```

---

## Problem 6: 4Sum — LC 18

Extension of 3Sum — add an outer loop.

```rust
fn four_sum(nums: &mut Vec<i32>, target: i32) -> Vec<Vec<i32>> {
    nums.sort();
    let mut result = Vec::new();
    let n = nums.len();

    for i in 0..n.saturating_sub(3) {
        if i > 0 && nums[i] == nums[i - 1] { continue; }
        for j in (i + 1)..n.saturating_sub(2) {
            if j > i + 1 && nums[j] == nums[j - 1] { continue; }
            let mut left = j + 1;
            let mut right = n - 1;
            while left < right {
                let sum = nums[i] as i64 + nums[j] as i64 + nums[left] as i64 + nums[right] as i64;
                if sum == target as i64 {
                    result.push(vec![nums[i], nums[j], nums[left], nums[right]]);
                    while left < right && nums[left] == nums[left + 1] { left += 1; }
                    while left < right && nums[right] == nums[right - 1] { right -= 1; }
                    left += 1;
                    right -= 1;
                } else if sum < target as i64 {
                    left += 1;
                } else {
                    right -= 1;
                }
            }
        }
    }
    result
}
```

**Use `i64` for sum** — 4 ints can overflow.

---

## Two Pointers vs Sliding Window

| Aspect | Two Pointers | Sliding Window |
|--------|-------------|---------------|
| Pointers | 2 independent | left + right (window boundaries) |
| Movement | Toward each other OR same direction | Always right expands, left shrinks |
| Array requirement | Often sorted (for opposite ends) | Usually unsorted |
| Goal | Find elements satisfying condition | Optimize subarray/substring |
| Count result | Usually pairs/triplets | Usually length or count of windows |

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Two Pointers on Linked List](../../4.Linked_List/Patterns/Two%20Pointers%20on%20LL.md)
- [Dutch National Flag (Arrays)](../../1.Arrays/Patterns/Dutch%20National%20Flag.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
