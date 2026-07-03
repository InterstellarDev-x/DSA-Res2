# Frequency Count Pattern

> **Topic:** [Strings](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `frequency` `anagram` `char-array` `hashmap`

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

Frequency Count reduces string comparison problems to **counting character occurrences** in an array of size 26 (lowercase) or 128/256 (ASCII). Two strings with identical frequency arrays are anagrams; a frequency array can be used as a canonical hash key.

**Key data structures:**

| Structure | Size | Use When |
|-----------|------|----------|
| `[i32; 26]` | 26 | Only lowercase letters `a–z` |
| `[i32; 128]` | 128 | All ASCII characters |
| `[i32; 256]` | 256 | Extended ASCII |
| `HashMap<char, i32>` | Dynamic | Unicode or unknown charset |

---

## When to Use

- Check if two strings are anagrams
- Group strings by anagram class
- Check if one string is a permutation of another
- Find characters present in one string but not another
- Verify character frequency constraints

---

## Recognition Cues

| Cue | Pattern |
|-----|---------|
| "check if two strings are anagrams" | Frequency count comparison |
| "group anagrams together" | Sorted string or frequency tuple as key |
| "ransom note: can words form message?" | Subtract one freq from another |
| "minimum deletions to make anagram" | Sum of absolute frequency differences |
| "character frequency ≥ k for all chars" | Count chars with freq ≥ k |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build frequency array | O(n) | O(1) — `[i32; 26]` |
| Compare two frequency arrays | O(26) = O(1) | O(1) |
| Group anagrams (n strings, avg len k) | O(n × k) | O(n × k) |
| `.iter().eq()` on `[i32; 26]` | O(26) = O(1) | O(1) |

---

## Rust Templates

### 1. Valid Anagram (LC 242)

```rust
fn is_anagram(s: &str, t: &str) -> bool {
    if s.len() != t.len() { return false; }
    let mut freq = [0i32; 26];
    for c in s.chars() { freq[(c as u8 - b'a') as usize] += 1; }
    for c in t.chars() { freq[(c as u8 - b'a') as usize] -= 1; }
    freq.iter().all(|&f| f == 0)
}
// Time: O(n) | Space: O(1)
```

### 2. Group Anagrams (LC 49) — Sorted Key

```rust
use std::collections::HashMap;

fn group_anagrams(strs: Vec<String>) -> Vec<Vec<String>> {
    let mut map: HashMap<String, Vec<String>> = HashMap::new();
    for s in strs {
        let mut key: Vec<char> = s.chars().collect();
        key.sort();
        let key: String = key.into_iter().collect();
        map.entry(key).or_default().push(s);
    }
    map.into_values().collect()
}
// Time: O(n × k log k) | Space: O(n × k)
```

### 3. Group Anagrams — Frequency Key (No Sort, O(n×k))

```rust
use std::collections::HashMap;

fn group_anagrams(strs: Vec<String>) -> Vec<Vec<String>> {
    let mut map: HashMap<String, Vec<String>> = HashMap::new();
    for s in strs {
        let mut freq = [0u32; 26];
        for c in s.chars() { freq[(c as u8 - b'a') as usize] += 1; }
        // Build canonical key: "#2#0#1#..."
        let key: String = freq.iter().map(|f| format!("#{}", f)).collect();
        map.entry(key).or_default().push(s);
    }
    map.into_values().collect()
}
// Time: O(n × k) | Space: O(n × k)
```

### 4. Ransom Note (LC 383)

```rust
fn can_construct(ransom_note: &str, magazine: &str) -> bool {
    let mut freq = [0i32; 26];
    for c in magazine.chars() { freq[(c as u8 - b'a') as usize] += 1; }
    for c in ransom_note.chars() {
        freq[(c as u8 - b'a') as usize] -= 1;
        if freq[(c as u8 - b'a') as usize] < 0 { return false; }
    }
    true
}
// Time: O(m + n) | Space: O(1)
```

### 5. First Unique Character (LC 387)

```rust
fn first_uniq_char(s: &str) -> i32 {
    let mut freq = [0u32; 26];
    for c in s.chars() { freq[(c as u8 - b'a') as usize] += 1; }
    let bytes = s.as_bytes();
    for i in 0..bytes.len() {
        if freq[(bytes[i] - b'a') as usize] == 1 { return i as i32; }
    }
    -1
}
// Time: O(n) | Space: O(1)
```

### 6. Minimum Steps to Make Two Strings Anagram (LC 1347)

```rust
fn min_steps(s: &str, t: &str) -> i32 {
    let mut freq = [0i32; 26];
    for c in s.chars() { freq[(c as u8 - b'a') as usize] += 1; }
    for c in t.chars() { freq[(c as u8 - b'a') as usize] -= 1; }
    freq.iter().filter(|&&f| f > 0).sum() // count chars s has in excess
}
// Time: O(n) | Space: O(1)
```

### 7. Sort Characters By Frequency (LC 451)

```rust
fn frequency_sort(s: &str) -> String {
    let mut freq = [0u32; 128];
    for c in s.chars() { freq[c as usize] += 1; }

    // Sort characters by frequency descending
    let mut chars: Vec<char> = (0u8..128)
        .filter(|&i| freq[i as usize] > 0)
        .map(|i| i as char)
        .collect();
    chars.sort_by(|&a, &b| freq[b as usize].cmp(&freq[a as usize]));

    let mut sb = String::new();
    for c in chars {
        for _ in 0..freq[c as usize] { sb.push(c); }
    }
    sb
}
// Time: O(n + 128 log 128) = O(n) | Space: O(n)
```

### 8. Isomorphic Strings (LC 205)

```rust
fn is_isomorphic(s: &str, t: &str) -> bool {
    let mut s_to_t = [0u8; 256];
    let mut t_to_s = [0u8; 256];
    let sb = s.as_bytes();
    let tb = t.as_bytes();
    for i in 0..sb.len() {
        let sc = sb[i] as usize;
        let tc = tb[i] as usize;
        if s_to_t[sc] == 0 && t_to_s[tc] == 0 {
            s_to_t[sc] = tb[i]; t_to_s[tc] = sb[i];
        } else if s_to_t[sc] != tb[i] || t_to_s[tc] != sb[i] { return false; }
    }
    true
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only checking one direction for isomorphic | Check both `s→t` and `t→s` mappings |
| Group anagrams: using array-to-string as key without separator | Use `#` separator to distinguish `[12,0]` from `[1,20]` |
| Not checking length before anagram check | Early return if `s.len() != t.len()` |
| Using `HashMap` when `[i32; 26]` suffices | 26x faster; always prefer for lowercase-only problems |
| `c - 'a'` on uppercase or non-alpha chars | Guard with `c.is_ascii_lowercase()` or use `[i32; 128]` |

---

## Variations

| Variation | Key Idea |
|-----------|----------|
| Scramble string | Frequency check + recursion |
| Check if subset anagram | All freqs of s ≤ freqs of t |
| Minimum window anagram | See [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) |
| Word pattern (LC 290) | Two-way `HashMap` mapping |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy | LC 242 |
| [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium | LC 49 |
| [Ransom Note](https://leetcode.com/problems/ransom-note/) | Easy | LC 383 |
| [First Unique Character in a String](https://leetcode.com/problems/first-unique-character-in-a-string/) | Easy | LC 387 |
| [Minimum Number of Steps to Make Two Strings Anagram](https://leetcode.com/problems/minimum-number-of-steps-to-make-two-strings-anagram/) | Medium | LC 1347 |
| [Sort Characters by Frequency](https://leetcode.com/problems/sort-characters-by-frequency/) | Medium | LC 451 |
| [Isomorphic Strings](https://leetcode.com/problems/isomorphic-strings/) | Easy | LC 205 |
| [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | LC 438 |

---

## Related Patterns

- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — when comparison is over a moving window
- [KMP and Z-Algorithm](./KMP%20and%20Z%20Algorithm.md) — for exact pattern matching
- [Two Pointers on Strings](./Two%20Pointers%20on%20Strings.md) — for in-place manipulation

---

> **Interview Tip:** For `group_anagrams`, the frequency-based key (`"#2#0#1..."`) is O(n×k) vs sorted key O(n×k log k). Mentioning both and explaining the trade-off signals strong algorithmic thinking.

> **Last Updated:** 2026-06-26
