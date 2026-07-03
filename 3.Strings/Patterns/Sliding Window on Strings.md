# Sliding Window on Strings

> **Topic:** [Strings](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `sliding-window` `frequency-map` `substring` `anagram`
> **See also:** [Arrays Sliding Window](../../1.Arrays/Patterns/Sliding%20Window.md)

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Rust Templates](#rust-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Sliding Window on strings uses a **frequency map** (usually `[i32; 26]` for lowercase letters) instead of a numeric sum. The window expands right on each step; when the window violates a constraint, it shrinks from the left.

**Two flavors:**
| Flavor | Shrink condition | Goal |
|--------|-----------------|------|
| **Fixed window** | size > k | Find matching window |
| **Variable window** | constraint violated | Maximize or minimize length |

**Critical string-specific difference from numeric arrays:**
- The invariant tracks a `formed` counter (how many chars satisfy their required frequency) rather than a running sum.
- This avoids scanning the entire map on every shrink step — O(26) per step vs O(1).

---

## When to Use

- Longest/shortest substring with some character-count constraint
- Find all anagrams / permutations of a pattern in a text
- Minimum window containing all chars of a pattern
- Substring with at most/exactly K distinct chars

---

## Recognition Cues

| Phrase | Pattern |
|--------|---------|
| "find all anagrams of p in s" | Fixed window = `len(p)`, char freq comparison |
| "longest substring without repeating characters" | Variable window + `lastSeen` map |
| "minimum window substring" | Variable window + `formed` counter |
| "at most K distinct characters" | Variable window + freq map size |
| "permutation of s in t" | Fixed window = `len(s)`, freq match |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Fixed window (freq array) | O(n) | O(1) — `[i32; 26]` |
| Variable window (freq map) | O(n) | O(charset) |
| Min window (formed counter) | O(n + m) | O(charset) |

---

## Rust Templates

### 1. Fixed Window — Find All Anagrams (LC 438)

```rust
fn find_anagrams(s: &str, p: &str) -> Vec<i32> {
    let mut result = Vec::new();
    let s_bytes = s.as_bytes();
    let p_bytes = p.as_bytes();
    if s_bytes.len() < p_bytes.len() {
        return result;
    }

    let mut p_freq = [0i32; 26];
    let mut w_freq = [0i32; 26];
    for &c in p_bytes {
        p_freq[(c - b'a') as usize] += 1;
    }

    let k = p_bytes.len();
    for r in 0..s_bytes.len() {
        w_freq[(s_bytes[r] - b'a') as usize] += 1;
        if r >= k {
            w_freq[(s_bytes[r - k] - b'a') as usize] -= 1; // remove leftmost
        }
        if p_freq == w_freq {
            result.push((r - k + 1) as i32);
        }
    }
    result
}
// Time: O(26n) = O(n) | Space: O(1) — arrays of size 26
```

### 2. Variable Window — Longest Substring Without Repeating Chars (LC 3)

```rust
fn length_of_longest_substring(s: &str) -> i32 {
    let s_bytes = s.as_bytes();
    let mut last_seen = [-1i32; 128]; // ASCII
    let mut l = 0i32;
    let mut max_len = 0i32;

    for r in 0..s_bytes.len() {
        let c = s_bytes[r] as usize;
        if last_seen[c] >= l {
            l = last_seen[c] + 1; // jump past duplicate
        }
        last_seen[c] = r as i32;
        max_len = max_len.max(r as i32 - l + 1);
    }
    max_len
}
// Time: O(n) | Space: O(1) — fixed size array
```

### 3. Variable Window — Minimum Window Substring (LC 76)

```rust
use std::collections::HashSet;

fn min_window(s: &str, t: &str) -> String {
    let s_bytes = s.as_bytes();
    let mut need = [0i32; 128];
    for &c in t.as_bytes() {
        need[c as usize] += 1;
    }

    let required = t.as_bytes().iter().collect::<HashSet<_>>().len() as i32;
    let mut formed = 0i32;
    let mut l = 0usize;
    let mut ans = (i32::MAX, 0usize, 0usize); // (len, l, r)

    let mut window = [0i32; 128];
    for r in 0..s_bytes.len() {
        let rc = s_bytes[r] as usize;
        window[rc] += 1;
        if need[rc] > 0 && window[rc] == need[rc] {
            formed += 1;
        }

        while formed == required {
            if (r - l + 1) as i32 < ans.0 {
                ans = ((r - l + 1) as i32, l, r);
            }
            let lc = s_bytes[l] as usize;
            l += 1;
            window[lc] -= 1;
            if need[lc] > 0 && window[lc] < need[lc] {
                formed -= 1;
            }
        }
    }
    if ans.0 == i32::MAX {
        String::new()
    } else {
        s[ans.1..=ans.2].to_string()
    }
}
// Time: O(|s| + |t|) | Space: O(1) — fixed ASCII arrays
```

### 4. Variable Window — Longest Substring with At Most K Distinct Chars (LC 340)

```rust
use std::collections::HashMap;

fn length_of_longest_substring_k_distinct(s: &str, k: usize) -> i32 {
    let s_bytes = s.as_bytes();
    let mut freq: HashMap<u8, i32> = HashMap::new();
    let mut l = 0usize;
    let mut max_len = 0i32;

    for r in 0..s_bytes.len() {
        *freq.entry(s_bytes[r]).or_insert(0) += 1;
        while freq.len() > k {
            let lc = s_bytes[l];
            l += 1;
            let count = freq.get_mut(&lc).unwrap();
            *count -= 1;
            if *count == 0 {
                freq.remove(&lc);
            }
        }
        max_len = max_len.max((r - l + 1) as i32);
    }
    max_len
}
// Time: O(n) | Space: O(k)
```

### 5. Exactly K Distinct = atMost(K) − atMost(K−1)

```rust
// Count subarrings with exactly K distinct chars
fn at_most(s: &[u8], k: i32) -> i32 {
    let mut freq = [0i32; 26];
    let mut l = 0usize;
    let mut count = 0i32;
    let mut distinct = 0i32;
    for r in 0..s.len() {
        if freq[(s[r] - b'a') as usize] == 0 {
            distinct += 1;
        }
        freq[(s[r] - b'a') as usize] += 1;
        while distinct > k {
            freq[(s[l] - b'a') as usize] -= 1;
            if freq[(s[l] - b'a') as usize] == 0 {
                distinct -= 1;
            }
            l += 1;
        }
        count += (r - l + 1) as i32; // all substrings ending at r with <= k distinct
    }
    count
}

fn subarrays_with_k_distinct(s: &str, k: i32) -> i32 {
    let s_bytes = s.as_bytes();
    at_most(s_bytes, k) - at_most(s_bytes, k - 1)
}
// Time: O(n) | Space: O(1)
```

### 6. Longest Repeating Character Replacement (LC 424)

```rust
fn character_replacement(s: &str, k: i32) -> i32 {
    let s_bytes = s.as_bytes();
    let mut freq = [0i32; 26];
    let mut l = 0usize;
    let mut max_freq = 0i32;
    let mut max_len = 0i32;

    for r in 0..s_bytes.len() {
        freq[(s_bytes[r] - b'a') as usize] += 1;
        max_freq = max_freq.max(freq[(s_bytes[r] - b'a') as usize]);
        // window size - max_freq = number of replacements needed
        while (r - l + 1) as i32 - max_freq > k {
            freq[(s_bytes[l] - b'a') as usize] -= 1;
            l += 1;
            // Note: max_freq may be stale but never decreases incorrectly
        }
        max_len = max_len.max((r - l + 1) as i32);
    }
    max_len
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `p_freq == w_freq` is O(26) per step | For larger alphabets, use a `matches` counter instead |
| Forgetting to remove key from map when freq hits 0 | `if *freq.get(&c).unwrap() == 0 { freq.remove(&c); }` — otherwise `.len()` is wrong |
| Min window: using `>=` for `formed` increment | Only `==` needed — catches the exact moment a char becomes satisfied |
| Fixed window: off-by-one when removing the outgoing char | Remove `s[r - k]`, not `s[r - k + 1]` |
| Counting `last_seen[c] >= 0` instead of `>= l` | Without `>= l` guard, stale entries cause wrong left boundary jumps |

---

## Variations

| Variation | Key Change |
|-----------|-----------|
| Permutation in string (LC 567) | Fixed window, same as anagram but return bool |
| Minimum window with at most duplicates | Modify formed counter logic |
| Max vowels in substring of length k | Fixed window, count vowels |
| Binary string max ones after k flips | Window with at most k zeros |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | LC 3 |
| [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | LC 76 |
| [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | LC 438 |
| [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | LC 567 |
| [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | LC 424 |
| [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) | Hard | LC 992 |
| [Longest Substring with At Most Two Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-two-distinct-characters/) | Medium | LC 159 |
| [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | LC 1004 |

---

## Related Patterns

- [Arrays Sliding Window](../../1.Arrays/Patterns/Sliding%20Window.md) — numeric version
- [Two Pointers on Strings](./Two%20Pointers%20on%20Strings.md) — single-pointer problems
- [Frequency Count](./Frequency%20Count.md) — static frequency comparison

---

> **Interview Tip:** For the `formed` counter pattern (min window), the key insight is: increment `formed` only when `window[c] == need[c]` (not `>=`). This gives O(1) per step instead of scanning the whole frequency map.

> **Last Updated:** 2026-06-26
