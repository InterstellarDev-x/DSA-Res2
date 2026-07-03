# Coding Tips — Sliding Window & Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## 1. The `right - left + 1` Counting Pattern

When a window `[left..right]` satisfies a condition and you need to count valid subarrays ending at `right`:

```rust
// All subarrays ending at 'right' starting anywhere in [left..right] are valid
result += right - left + 1;
// Subarrays: [right..right], [right-1..right], ..., [left..right]
// Count = right - left + 1
```

This is the core of the `atMost` counting technique.

---

## 2. Fixed Window — `right - k` for the Outgoing Element

```rust
for right in k..n {
    add(nums[right]);
    remove(nums[right - k]);   // right - k is the element leaving the window
}
```

Don't maintain a separate `left` pointer — `left = right - k + 1` always, so the outgoing element is always `nums[right - k]`.

---

## 3. Variable Window — `while` vs `if` for Shrinking

```rust
// Use WHILE when finding MINIMUM length (keep shrinking as long as valid)
while satisfied() {
    update_answer();
    remove_left();
    left += 1;
}

// Use IF when finding MAXIMUM length (shrink by exactly 1 when violated)
if violated() {
    remove_left();
    left += 1;
}
// Window size is non-decreasing — we never need to shrink more than 1
```

The `if` trick works for "maximum length" problems because shrinking more than 1 can only decrease the window size below `maxLen`, which is already recorded.

---

## 4. Min Window Substring — `need[c]` Dual Role

```rust
// 'need' serves as both frequency tracker AND satisfiedness indicator
// need[c] > 0: we still need more of c
// need[c] == 0: c is exactly satisfied
// need[c] < 0: we have excess of c

if need[c] > 0 { have += 1; }   // if we needed this (>0 before -=), increment have
need[c] -= 1;
// On shrink:
need[l] += 1;
if need[l] > 0 { have -= 1; }   // if restoring makes need positive, we lost a needed char
```

---

## 5. At Most K Trick — Always Guard `goal < 0`

```rust
fn at_most(nums: &[i32], goal: i32) -> i32 {
    if goal < 0 { return 0; }   // ← CRITICAL when called with goal-1 and goal=0
    // ...
    0
}
```

For `exactly(k) = atMost(k) - atMost(k-1)`, when `k=0` we call `atMost(-1)`. Without the guard, negative goal causes incorrect behavior.

---

## 6. Character Replacement — `if` Not `while`

```rust
// CORRECT: if not while — we want max window, not min
if (right - left + 1) as i32 - max_freq > k {
    freq[(s.as_bytes()[left] - b'A') as usize] -= 1;
    left += 1;
    // Don't recompute max_freq — it can only grow or stay same for a better answer
}
```

Key consequence: window size is **non-decreasing**. The answer is the maximum window size seen, which equals the final window size.

---

## 7. Two Pointers — Sort First for Sum Problems

```rust
// 3Sum, 4Sum, Two Sum II — ALWAYS sort first
nums.sort();
// Then: sum too small → left += 1 (increase sum)
//       sum too large → right -= 1 (decrease sum)
```

Without sorting, you can't make greedy choices about which pointer to move.

---

## 8. Duplicate Skipping in kSum

```rust
// Skip duplicate i (outer loop)
if i > 0 && nums[i] == nums[i-1] { continue; }

// Skip duplicate left/right after finding a valid triplet
while left < right && nums[left] == nums[left+1] { left += 1; }
while left < right && nums[right] == nums[right-1] { right -= 1; }
left += 1; right -= 1;   // ← MUST advance both after skipping
```

The skip condition is `i > 0` (not just `i > start`) to allow the first occurrence. The advance after skipping is easy to forget.

---

## 9. Product Window — Guard for `k <= 1`

```rust
if k <= 1 { return 0; }   // all elements ≥ 1, product ≥ 1, nothing < 1
```

Without this guard, `k=1` causes an infinite loop: product starts at 1, `1 >= 1` triggers shrink, shrink never reduces product below 1 for positive arrays.

---

## Quick Pattern Reference

| Signal in Problem | Pattern |
|------------------|---------|
| "exactly k [property]" | `atMost(k) - atMost(k-1)` |
| "at most k distinct" | Variable window with map |
| "minimum window containing" | Variable window with need/have |
| "all windows of size k" | Fixed window |
| "maximum window where [condition]" | Variable window, `if` for shrink |
| "minimum window satisfying [condition]" | Variable window, `while` for shrink |
| "pairs/triplets summing to target" | Sort + two pointers |
| "maximum area/water" | Two pointers from ends |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Window Template](./Window%20Template.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
