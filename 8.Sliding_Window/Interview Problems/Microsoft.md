# Microsoft — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Microsoft

---

## Problem 1: Container With Most Water — Deep Dive

**LC 11** · Medium

```rust
fn max_area(height: &[i32]) -> i32 {
    let mut left = 0;
    let mut right = height.len() - 1;
    let mut max_water = 0;
    while left < right {
        max_water = max_water.max(height[left].min(height[right]) * (right - left) as i32);
        if height[left] <= height[right] {
            left += 1;
        } else {
            right -= 1;
        }
    }
    max_water
}
```

**Q: Prove we never miss the optimal pair.**
A: Suppose the optimal pair is `(i, j)` with `i < j`. WLOG, `height[i] ≤ height[j]`. When our pointers are at `left = i`, we move left (since `height[left] ≤ height[right]`). This is safe because any pair `(i, r)` for `r < j` (i.e., before we've reached `j`) has width `< j - i` and height `≤ height[i]`, so area `< height[i] × (j-i)`. Moving left is the only way to potentially find a taller left wall.

**Q: What if there are multiple optimal pairs?**
A: The algorithm finds at least one — it doesn't need to enumerate all. For a minimum answer or counting follow-up, a different approach would be needed.

---

## Problem 2: 3Sum — Deep Dive

**LC 15** · Medium

```rust
fn three_sum(nums: &mut Vec<i32>) -> Vec<Vec<i32>> {
    nums.sort();
    let mut result = Vec::new();
    let n = nums.len();
    if n < 3 {
        return result;
    }
    for i in 0..n - 2 {
        if i > 0 && nums[i] == nums[i - 1] {
            continue;
        }
        let mut left = i + 1;
        let mut right = n - 1;
        while left < right {
            let sum = nums[i] + nums[left] + nums[right];
            if sum == 0 {
                result.push(vec![nums[i], nums[left], nums[right]]);
                while left < right && nums[left] == nums[left + 1] {
                    left += 1;
                }
                while left < right && nums[right] == nums[right - 1] {
                    right -= 1;
                }
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

**Microsoft edge case focus:**
- All zeros: `[0,0,0]` → `[[0,0,0]]` — valid, one triplet
- All same positive: `[1,1,1]` → `[]` — no triplet sums to 0
- Empty/one/two elements → return `[]`

**Q: Why `i < nums.size() - 2` not `i < nums.size()`?**
A: We need at least 2 more elements for `left` and `right`. If `i = n-1`, `left = i+1 = n` which is out of bounds.

---

## Problem 3: Minimum Window Substring — Microsoft Style

**LC 76** · Hard

Microsoft asks this with emphasis on clean code and edge case handling:

```rust
fn min_window(s: &str, t: &str) -> String {
    if s.is_empty() || t.is_empty() || s.len() < t.len() {
        return String::new();
    }

    let s_bytes = s.as_bytes();
    let mut need = [0i32; 128];
    for &c in t.as_bytes() {
        need[c as usize] += 1;
    }
    let required = t.len() as i32;
    let mut have = 0i32;
    let mut left = 0;
    let mut min_len = i32::MAX;
    let mut min_left = 0;

    for right in 0..s_bytes.len() {
        let c = s_bytes[right] as usize;
        if need[c] > 0 {
            have += 1;
        }
        need[c] -= 1; // need[c] > 0 before decrement → satisfying need

        while have == required {
            if (right - left + 1) as i32 < min_len {
                min_len = (right - left + 1) as i32;
                min_left = left;
            }
            let lc = s_bytes[left] as usize;
            need[lc] += 1; // restore and check
            if need[lc] > 0 {
                have -= 1;
            }
            left += 1;
        }
    }

    if min_len == i32::MAX {
        String::new()
    } else {
        s[min_left..min_left + min_len as usize].to_string()
    }
}
```

**Microsoft interview emphasis:**
- Empty checks (`.is_empty()`)
- Length check early return
- Clear variable naming (`required`, `have`, `need`)
- Correct slice `s[start..start+len].to_string()` call

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Two Pointers](../Patterns/Two%20Pointers.md)

> **Last Updated:** 2026-06-26
