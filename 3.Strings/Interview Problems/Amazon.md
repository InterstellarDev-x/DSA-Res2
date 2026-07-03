# Amazon — Strings Interview Questions

> **Topic:** [Strings](../README.md) · **Section:** Interview Problems
> **Tags:** `amazon` `interview` `strings`

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **String concatenation** | Use `String` with `push_str`; avoid repeated string concatenation in a loop |
| **Char frequency** | `[i32; 26]` over `HashMap` when possible |
| **Edge cases** | Empty string, single char, all same chars, spaces |
| **Custom sort** | Write a comparator correctly (total ordering) |
| **Clarify charset** | "Are there only lowercase letters? ASCII? Unicode?" |

---

## Frequently Asked Questions

### 1. Reorder Data in Log Files ⭐ (Amazon Signature)

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | OA, Phone |
| **Skill** | Custom comparator, stable sort |
| **LeetCode** | [LC 937](https://leetcode.com/problems/reorder-data-in-log-files/) |

```rust
fn reorder_log_files(logs: Vec<String>) -> Vec<String> {
    let mut letters: Vec<String> = Vec::new();
    let mut digits: Vec<String> = Vec::new();
    for log in &logs {
        let space_pos = log.find(' ').unwrap();
        let after_space = log.as_bytes()[space_pos + 1] as char;
        if after_space.is_ascii_digit() {
            digits.push(log.clone());
        } else {
            letters.push(log.clone());
        }
    }
    // sort_by in Rust is stable
    letters.sort_by(|a, b| {
        let ca = &a[a.find(' ').unwrap() + 1..];
        let cb = &b[b.find(' ').unwrap() + 1..];
        let cmp = ca.cmp(cb);
        if cmp != std::cmp::Ordering::Equal {
            cmp
        } else {
            a.cmp(b) // tie-break by identifier
        }
    });
    letters.extend(digits);
    letters
}
```
**Follow-ups:** "What if two logs have the same content AND same identifier?" → they're equal, preserve original order → Rust's `sort_by` is stable.

---

### 2. Longest Substring Without Repeating Characters

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Round 1 |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |

**Follow-ups:**
- "What if you need the actual substring, not just length?" → Track start index and `max_start`
- "Unicode chars?" → Use `HashMap<char, i32>` instead of `[i32; 128]`

---

### 3. Group Anagrams

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1, Round 2 |
| **Pattern** | [Frequency Count](../Patterns/Frequency%20Count.md) |
| **LeetCode** | [LC 49](https://leetcode.com/problems/group-anagrams/) |

**Follow-ups:**
- "Sort vs frequency key — which is better?" → Frequency O(n×k) vs Sort O(n×k log k); prefer freq key
- "What if strings can be empty?" → `""` is its own anagram group

---

### 4. String to Integer (atoi)

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1 (edge-case heavy) |
| **Skill** | State machine parsing, overflow detection |
| **LeetCode** | [LC 8](https://leetcode.com/problems/string-to-integer-atoi/) |

```rust
fn my_atoi(s: String) -> i32 {
    let bytes = s.as_bytes();
    let n = bytes.len();
    let mut i = 0;
    let mut sign: i64 = 1;
    let mut result: i64 = 0;
    // skip spaces
    while i < n && bytes[i] == b' ' {
        i += 1;
    }
    if i < n && (bytes[i] == b'+' || bytes[i] == b'-') {
        sign = if bytes[i] == b'-' { -1 } else { 1 };
        i += 1;
    }
    while i < n && (bytes[i] as char).is_ascii_digit() {
        result = result * 10 + (bytes[i] - b'0') as i64;
        if result * sign > i32::MAX as i64 {
            return i32::MAX;
        }
        if result * sign < i32::MIN as i64 {
            return i32::MIN;
        }
        i += 1;
    }
    (result * sign) as i32
}
```

---

### 5. Minimum Window Substring

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Round 2, Bar Raiser |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 76](https://leetcode.com/problems/minimum-window-substring/) |

**Follow-ups:**
- "What's the space complexity?" → O(1) with `[i32; 128]` vs O(Σ) with `HashMap`
- "What if pattern has duplicate characters?" → Frequency-based `formed` handles it correctly

---

## Related Files

- [Amazon OA Questions](../OA-Qns/Amazon.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window%20on%20Strings.md)
- [Interview Tips — Java String API](../Interview%20Tips/Java%20String%20API.md)

> **Last Updated:** 2026-06-26
