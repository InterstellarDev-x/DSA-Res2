# Google — Strings Interview Questions

> **Topic:** [Strings](../README.md) · **Section:** Interview Problems
> **Tags:** `google` `interview` `strings`

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **KMP derivation** | Build LPS from scratch, explain each step |
| **O(n) proof** | Why does KMP not exceed O(n)? |
| **Application insight** | Recognise rotation / palindrome as KMP in disguise |
| **Expand around center** | Palindrome in O(n) without DP |
| **No regex shortcuts** | Don't use `String.matches()` — implement manually |

---

## Frequently Asked Questions

### 1. Longest Palindromic Substring ⭐ (Google Flagship)

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, R1, R2 |
| **Skill** | Expand around center (O(n)) vs DP (O(n²) space) |
| **LeetCode** | [LC 5](https://leetcode.com/problems/longest-palindromic-substring/) |

```rust
fn expand(s: &[u8], l: i32, r: i32) -> i32 {
    let mut l = l;
    let mut r = r;
    while l >= 0 && r < s.len() as i32 && s[l as usize] == s[r as usize] {
        l -= 1;
        r += 1;
    }
    r - l - 1 // length of palindrome
}

fn longest_palindrome(s: String) -> String {
    let bytes = s.as_bytes();
    let n = s.len() as i32;
    let mut start = 0usize;
    let mut max_len = 1i32;
    for i in 0..n {
        let odd  = expand(bytes, i, i);      // odd-length
        let even = expand(bytes, i, i + 1);  // even-length
        let len  = odd.max(even);
        if len > max_len {
            max_len = len;
            start = (i - (len - 1) / 2) as usize;
        }
    }
    s[start..start + max_len as usize].to_string()
}
// Time: O(n) | Space: O(1)
```

**Follow-ups:**
- "Manacher's algorithm?" → O(n) but complex; expand-around-center is preferred unless asked explicitly
- "Count all palindromic substrings?" → Same loop, count instead of track max (LC 647)
- "Longest palindromic subsequence?" → DP, different problem (LC 516)

---

### 2. Minimum Window Substring

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | R1, R2 |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 76](https://leetcode.com/problems/minimum-window-substring/) |

**Follow-ups:**
- "Can you reduce space below O(charset)?" → Only if you sort the window — not worth it
- "What if we want minimum window with exactly K distinct chars?" → Different problem, different sliding window

---

### 3. Shortest Palindrome (KMP Application)

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | R2, R3 |
| **Pattern** | [KMP](../Patterns/KMP%20and%20Z%20Algorithm.md) — LPS of `s + "#" + reverse(s)` |
| **LeetCode** | [LC 214](https://leetcode.com/problems/shortest-palindrome/) |

**Key insight:**
> The longest palindromic prefix of `s` = LPS value at the last index of `s + "#" + reverse(s)`. The `#` separator prevents the LPS from crossing the midpoint.

---

### 4. Repeated Substring Pattern

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy |
| **Round** | Phone |
| **Pattern** | [KMP](../Patterns/KMP%20and%20Z%20Algorithm.md) — `n % (n - lps[n-1]) == 0` |
| **LeetCode** | [LC 459](https://leetcode.com/problems/repeated-substring-pattern/) |

---

## Related Files

- [Google OA Questions](../OA-Qns/Google.md)
- [KMP and Z-Algorithm Pattern](../Patterns/KMP%20and%20Z%20Algorithm.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window%20on%20Strings.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)

> **Last Updated:** 2026-06-26
