# Two Pointers on Strings

> **Topic:** [Strings](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `two-pointers` `palindrome` `reverse` `in-place`
> **See also:** [Arrays Two Pointers](../../1.Arrays/Patterns/Two%20Pointers.md)

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

Two Pointers on strings applies the same opposite-ends or same-direction pointer logic as on arrays, with one key difference: **Rust `String` values are mutable when owned**, so in-place operations can be performed directly on `Vec<char>` or byte slices.

**Three sub-styles:**

| Style | Pointer Start | Typical Use |
|-------|--------------|-------------|
| Opposite ends | `l=0, r=n-1` | Palindrome check, reverse string |
| Same direction (fast/slow) | `i=0, j=0` | Remove spaces, deduplicate |
| Two strings | `i=0, j=0` | Merge, compare version numbers |

---

## When to Use

- Check if string is a palindrome (with/without constraints)
- Reverse a string or part of it in-place
- Remove duplicates or specific characters
- Compare two strings with wildcard matching
- Merge or interleave two strings

---

## Recognition Cues

| Cue | Style |
|-----|-------|
| "valid palindrome ignoring non-alphanumeric" | Opposite ends |
| "reverse words in a string" | Reverse whole then each word |
| "valid palindrome after deleting at most one char" | Try skip left OR skip right |
| "compare version numbers" | Two pointer across two strings |
| "merge two sorted strings" | Two pointer merge |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Palindrome check | O(n) | O(1) in-place on `String` |
| Reverse string | O(n) | O(1) in-place |
| Reverse words | O(n) | O(n) — copy required |
| Two-string compare | O(m+n) | O(1) |

---

## Rust Templates

### 1. Valid Palindrome (Ignore Non-Alphanumeric, LC 125)

```rust
fn is_palindrome(s: String) -> bool {
    let s = s.as_bytes();
    let n = s.len();
    let mut l = 0usize;
    let mut r = n.saturating_sub(1);
    while l < r {
        while l < r && !s[l].is_ascii_alphanumeric() { l += 1; }
        while l < r && !s[r].is_ascii_alphanumeric() { r -= 1; }
        if s[l].to_ascii_lowercase() != s[r].to_ascii_lowercase() { return false; }
        l += 1;
        r -= 1;
    }
    true
}
// Time: O(n) | Space: O(1)
```

### 2. Valid Palindrome II — At Most One Deletion (LC 680)

```rust
fn is_palin(s: &[u8], mut l: usize, mut r: usize) -> bool {
    while l < r {
        if s[l] != s[r] { return false; }
        l += 1;
        r -= 1;
    }
    true
}

fn valid_palindrome(s: String) -> bool {
    let s = s.as_bytes();
    let n = s.len();
    let mut l = 0usize;
    let mut r = n.saturating_sub(1);
    while l < r {
        if s[l] != s[r] {
            return is_palin(s, l + 1, r) || is_palin(s, l, r - 1);
        }
        l += 1;
        r -= 1;
    }
    true
}
// Time: O(n) | Space: O(1)
```

### 3. Reverse String In-Place (LC 344)

```rust
fn reverse_string(s: &mut Vec<char>) {
    let n = s.len();
    let mut l = 0usize;
    let mut r = n.saturating_sub(1);
    while l < r {
        s.swap(l, r);
        l += 1;
        r -= 1;
    }
}
// Time: O(n) | Space: O(1)
```

### 4. Reverse Words in a String (LC 151)

```rust
fn reverse_words(s: String) -> String {
    // Split on whitespace, reverse word order
    let mut words: Vec<&str> = s.split_whitespace().collect();
    let mut l = 0usize;
    let mut r = words.len().saturating_sub(1);
    while l < r {
        words.swap(l, r);
        l += 1;
        r -= 1;
    }
    words.join(" ")
}
// Time: O(n) | Space: O(n) — copy required
```

### 5. Rotate String — Three-Reverse Trick (LC 796)

```rust
// Check if s can become goal by rotation
fn rotate_string(s: String, goal: String) -> bool {
    s.len() == goal.len() && s.repeat(2).contains(goal.as_str())
    // Alternative: KMP search in doubled string
}
// Time: O(n²) with contains; O(n) with KMP
```

### 6. Compare Version Numbers (LC 165)

```rust
fn compare_version(version1: String, version2: String) -> i32 {
    let v1 = version1.as_bytes();
    let v2 = version2.as_bytes();
    let mut i = 0usize;
    let mut j = 0usize;
    while i < v1.len() || j < v2.len() {
        let mut num1 = 0i32;
        let mut num2 = 0i32;
        while i < v1.len() && v1[i] != b'.' {
            num1 = num1 * 10 + (v1[i] - b'0') as i32;
            i += 1;
        }
        while j < v2.len() && v2[j] != b'.' {
            num2 = num2 * 10 + (v2[j] - b'0') as i32;
            j += 1;
        }
        if num1 != num2 { return if num1 > num2 { 1 } else { -1 }; }
        i += 1; // skip '.'
        j += 1; // skip '.'
    }
    0
}
// Time: O(m+n) | Space: O(1)
```

### 7. Long Pressed Name (LC 925)

```rust
fn is_long_pressed_name(name: String, typed: String) -> bool {
    let name = name.as_bytes();
    let typed = typed.as_bytes();
    let mut i = 0usize;
    let mut j = 0usize;
    while j < typed.len() {
        if i < name.len() && name[i] == typed[j] {
            i += 1;
            j += 1;
        } else if j > 0 && typed[j] == typed[j - 1] {
            j += 1;
        } else {
            return false;
        }
    }
    i == name.len()
}
// Time: O(m+n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Palindrome: not converting to lowercase | `to_ascii_lowercase()` both chars before comparing |
| Palindrome: not skipping non-alphanumeric | Use `is_ascii_alphanumeric()` guard inside while |
| Reverse words: splitting on single space only | Use `split_whitespace()` to handle multiple spaces automatically |
| In-place reverse: unnecessary copy | Can modify `Vec<char>` directly in Rust using `.swap()` |
| Two-string pointer: forgetting to advance both `i` and `j` | After a match, both pointers advance |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Palindrome with ignored chars | Two pointer + `is_ascii_alphanumeric()` filter |
| Palindrome after at most k deletions | DP for k > 1; greedy for k = 1 |
| Merge alternately | Two pointer, append from each |
| Backspace string compare (LC 844) | Simulate with stack or reverse pointer |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy | LC 125 |
| [Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii/) | Easy | LC 680 |
| [Reverse String](https://leetcode.com/problems/reverse-string/) | Easy | LC 344 |
| [Reverse Words in a String](https://leetcode.com/problems/reverse-words-in-a-string/) | Medium | LC 151 |
| [Compare Version Numbers](https://leetcode.com/problems/compare-version-numbers/) | Medium | LC 165 |
| [Long Pressed Name](https://leetcode.com/problems/long-pressed-name/) | Easy | LC 925 |
| [Backspace String Compare](https://leetcode.com/problems/backspace-string-compare/) | Easy | LC 844 |

---

## Related Patterns

- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — when window size changes
- [Frequency Count](./Frequency%20Count.md) — static char comparison
- [KMP and Z-Algorithm](./KMP%20and%20Z%20Algorithm.md) — for rotation and substring search

---

> **Interview Tip:** For "valid palindrome after one deletion", the key is to try skipping both options (`l+1` and `r-1`) when a mismatch is found — not just one. Picking arbitrarily breaks on strings like `"aguokebbisssbibbguoqauikoobk"`.

> **Last Updated:** 2026-06-26
