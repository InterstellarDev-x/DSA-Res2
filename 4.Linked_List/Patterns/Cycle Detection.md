# Cycle Detection

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** LC 141, 142, 287 (Find Duplicate), Happy Number

---

## Core Idea

Floyd's algorithm uses **two pointers** (slow/fast) to detect a cycle in O(n) time and O(1) space. The mathematical invariant is the key to finding the **entry point** of the cycle.

---

## Floyd's Invariant (Proof)

Let:
- **F** = distance from head to cycle entry
- **a** = distance from cycle entry to meeting point
- **c** = cycle length

When they meet:
- slow traveled: `F + a`
- fast traveled: `F + a + n×c` (some number of full loops)
- Since fast = 2×slow: `F + a + nc = 2(F + a)` → `F = nc - a`

This means: starting from the **meeting point** and **head** simultaneously, both moving 1 step, they meet at the **cycle entry** (both travel exactly F steps, which equals nc−a — which is equivalent to reaching the entry from different directions).

---

## Template 1 — Detect Cycle (LC 141)

```rust
// Index-based linked list: next[i] is the index of the next node; usize::MAX represents null
fn has_cycle(next: &[usize]) -> bool {
    if next.is_empty() {
        return false;
    }
    let null = usize::MAX;
    let mut slow = 0usize;
    let mut fast = 0usize;
    while fast != null && next[fast] != null {
        slow = next[slow];
        fast = next[next[fast]];
        if slow == fast {
            return true;
        }
    }
    false
}
```

---

## Template 2 — Find Cycle Entry (LC 142)

```rust
// Index-based linked list: next[i] is the index of the next node; usize::MAX represents null
fn detect_cycle(next: &[usize]) -> Option<usize> {
    if next.is_empty() {
        return None;
    }
    let null = usize::MAX;
    let mut slow = 0usize;
    let mut fast = 0usize;

    // Phase 1: find meeting point inside cycle
    loop {
        if fast == null || next[fast] == null {
            return None; // No cycle
        }
        slow = next[slow];
        fast = next[next[fast]];
        if slow == fast {
            break;
        }
    }

    // Phase 2: slow resets to head, fast stays at meeting; both move 1x
    slow = 0;
    while slow != fast {
        slow = next[slow];
        fast = next[fast];
    }
    Some(slow)
}
```

---

## Template 3 — Find Duplicate Number (LC 287)

The key insight: treat array values as "next pointers". Index `i` points to `nums[i]`. A duplicate value means **two indices point to the same next** — creating a cycle.

```rust
fn find_duplicate(nums: &[i32]) -> i32 {
    // Phase 1: find meeting point
    let mut slow = nums[0] as usize;
    let mut fast = nums[nums[0] as usize] as usize;
    while slow != fast {
        slow = nums[slow] as usize;
        fast = nums[nums[fast] as usize] as usize;
    }

    // Phase 2: find entry (the duplicate)
    slow = 0; // reset to "head" (index 0)
    while slow != fast {
        slow = nums[slow] as usize;
        fast = nums[fast] as usize;
    }
    slow as i32
}
```

**Constraint:** Array is `nums[1..n]` containing values in `[1, n]`. Index 0 is the entry point (like `head`). No modification to array. O(n) time, O(1) space.

---

## Template 4 — Detect Cycle Length

After detecting a meeting point, count steps before they meet again:

```rust
// Index-based linked list: next[i] is the index of the next node; usize::MAX represents null
fn cycle_length(next: &[usize]) -> usize {
    if next.is_empty() {
        return 0;
    }
    let null = usize::MAX;
    let mut slow = 0usize;
    let mut fast = 0usize;
    while fast != null && next[fast] != null {
        slow = next[slow];
        fast = next[next[fast]];
        if slow == fast {
            // Count cycle length from meeting point
            let mut length = 1;
            let mut curr = next[slow];
            while curr != slow {
                curr = next[curr];
                length += 1;
            }
            return length;
        }
    }
    0 // no cycle
}
```

---

## Comparison: Floyd's vs HashSet

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| `HashSet` `visited` | O(n) | O(n) | Clarity in non-constrained problems |
| Floyd's | O(n) | O(1) | Memory-constrained or interview requirement |
| Floyd's + entry | O(n) | O(1) | When cycle start needed |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Checking `slow == fast` before advancing — always `true` at start | Check AFTER advancing both |
| In phase 2 of cycle entry: moving fast at 2x still | Both pointers move 1x in phase 2 |
| For LC 287: starting `fast = nums[nums[0]]` not `nums[0]` | Must advance fast once more than slow at start to separate them |
| Using cycle detection on a sorted LL (no cycle) and expecting entry | Always verify no-cycle case first |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Detect cycle | O(n) | O(1) |
| Find cycle entry | O(n) | O(1) |
| Find duplicate (LC 287) | O(n) | O(1) |

---

## Related Patterns

- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — same algorithm, generalizes to middle/reorder
- [Two Pointers on LL](./Two%20Pointers%20on%20LL.md) — different "two pointer" flavor (gap-based)

---

**Back:** [Linked List README](../README.md) | **Prev:** [Merge Linked Lists](./Merge%20Linked%20Lists.md) | **Next:** [Two Pointers on LL](./Two%20Pointers%20on%20LL.md)

> **Last Updated:** 2026-06-26
