# Amazon — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Amazon

---

## Problem 1: Minimum Window Substring — Deep Dive

**LC 76** · Hard · O(|s| + |t|) time, O(1) space

### Full Solution with `have/need` Counter

```rust
fn min_window(s: String, t: String) -> String {
    let s = s.as_bytes();
    let t = t.as_bytes();
    let mut need = [0i32; 128];
    for &c in t {
        need[c as usize] += 1;
    }

    let mut left = 0usize;
    let mut have = 0i32;
    let required = t.len() as i32;
    let mut min_len = i32::MAX;
    let mut min_left = 0usize;

    for right in 0..s.len() {
        let c = s[right] as usize;
        if need[c] > 0 {
            have += 1; // satisfying a genuine need
        }
        need[c] -= 1;

        while have == required {
            if (right - left + 1) as i32 < min_len {
                min_len = (right - left + 1) as i32;
                min_left = left;
            }
            let l = s[left] as usize;
            need[l] += 1;
            if need[l] > 0 {
                have -= 1; // we lost a needed char
            }
            left += 1;
        }
    }

    if min_len == i32::MAX {
        String::new()
    } else {
        String::from_utf8(s[min_left..min_left + min_len as usize].to_vec()).unwrap()
    }
}
```

**Q: Why does `need[c] > 0` determine if we increment `have`?**
A: `need` array starts with counts from `t`. As we add characters from `s`, `need[c]` decreases. If `need[c] > 0` before decrement, this character was genuinely needed — we're satisfying a requirement. If `need[c] ≤ 0`, this is an excess character (already have enough of this type) — `have` shouldn't increase.

**Q: On shrinking, why does `need[l]++ > 0` trigger `have--`?**
A: After `need[l] += 1`, if the count is now positive (> 0), it means we needed this char and removing it from the window means we're no longer satisfying that need. If the count is ≤ 0 after increment, we still have excess copies in the window — no need to decrement `have`.

**Q: What if `t` has duplicate characters like "AAB"?**
A: The `need` array handles this — `need['A' as usize] = 2`, so you need two A's in the window. Each A added from `s` decrements `need['A' as usize]`; `have` increments when `need['A' as usize]` goes from positive to zero/negative, meaning we've satisfied all A requirements.

**Amazon LP Alignment:**

| LP | Connection |
|----|-----------|
| Dive Deep | `need[c] > 0` vs `need[c] <= 0` distinction |
| Invent & Simplify | Using `need` both as frequency count and as "satisfied" tracker |
| Deliver Results | O(|s|) single pass with O(1) decision per character |

---

## Problem 2: Longest Repeating Character Replacement — Deep Dive

**LC 424** · Medium · O(n) time

```rust
fn character_replacement(s: String, k: i32) -> i32 {
    let s = s.as_bytes();
    let mut freq = [0i32; 26];
    let mut left = 0usize;
    let mut max_freq = 0i32;
    let mut max_len = 0i32;

    for right in 0..s.len() {
        let idx = (s[right] - b'A') as usize;
        freq[idx] += 1;
        max_freq = max_freq.max(freq[idx]);

        if (right - left + 1) as i32 - max_freq > k {
            freq[(s[left] - b'A') as usize] -= 1;
            left += 1;
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Q: Why `if` not `while` for shrinking?**
A: We're maximizing window size. Once we shrink by 1, the window is at the same size as before this iteration's expansion — it can never be smaller than `max_len`. We don't need to keep shrinking; we just maintain size until the window can grow again.

**Q: Prove `max_freq` doesn't need to be recomputed on shrink.**
A: We're looking for the maximum window. Any valid window must have `windowSize - max_freq ≤ k`. When we shrink, the window decreases to the previous maximum size. For the answer to improve, we need a larger `max_freq` in a future window. So not updating `max_freq` downward is safe — it represents "the best we've seen so far," and only a better value can make the window grow.

---

## Problem 3: Fruits Into Baskets — Amazon Phone Screen

**LC 904** · Medium

```rust
use std::collections::HashMap;

fn total_fruit(fruits: Vec<i32>) -> i32 {
    let mut basket: HashMap<i32, i32> = HashMap::new();
    let mut left = 0usize;
    let mut max_len = 0i32;

    for right in 0..fruits.len() {
        *basket.entry(fruits[right]).or_insert(0) += 1;

        while basket.len() > 2 {
            let f = fruits[left];
            let count = basket.get_mut(&f).unwrap();
            *count -= 1;
            if *count == 0 {
                basket.remove(&f);
            }
            left += 1;
        }

        max_len = max_len.max((right - left + 1) as i32);
    }
    max_len
}
```

**Q: Generalize to at most k distinct types?**
A: Replace `> 2` with `> k`. Same O(n) solution.

**Q: What if fruits[] can be very large values (not 0..n)?**
A: `HashMap` handles arbitrary values. If values were bounded (e.g., 0..1000), an array of size 1001 would be more efficient.

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
