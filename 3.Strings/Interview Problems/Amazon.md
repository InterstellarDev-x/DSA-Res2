# Amazon — Strings Interview Questions

> **Topic:** [Strings](../README.md) · **Section:** Interview Problems
> **Tags:** `amazon` `interview` `strings`

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **String concatenation** | Use `string` with `+=`; avoid repeated string concatenation in a loop |
| **Char frequency** | `int[26]` over `std::unordered_map` when possible |
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

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> reorderLogFiles(vector<string>& logs) {
    vector<string> letters, digits;
    for (auto& log : logs) {
        int spacePos = log.find(' ');
        if (isdigit(log[spacePos + 1])) digits.push_back(log);
        else letters.push_back(log);
    }
    stable_sort(letters.begin(), letters.end(), [](const string& a, const string& b) {
        string ca = a.substr(a.find(' ') + 1);
        string cb = b.substr(b.find(' ') + 1);
        int cmp = ca.compare(cb);
        return cmp != 0 ? cmp < 0 : a < b; // tie-break by identifier
    });
    letters.insert(letters.end(), digits.begin(), digits.end());
    return letters;
}
```
**Follow-ups:** "What if two logs have the same content AND same identifier?" → they're equal, preserve original order → `std::stable_sort` is stable in C++.

---

### 2. Longest Substring Without Repeating Characters

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Round 1 |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |

**Follow-ups:**
- "What if you need the actual substring, not just length?" → Track start index and `maxStart`
- "Unicode chars?" → Use `std::unordered_map<char, int>` instead of `int[128]`

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

```cpp
#include <bits/stdc++.h>
using namespace std;

int myAtoi(string s) {
    int i = 0, n = s.length(), sign = 1;
    long result = 0;
    while (i < n && s[i] == ' ') i++;           // skip spaces
    if (i < n && (s[i] == '+' || s[i] == '-'))
        sign = s[i++] == '-' ? -1 : 1;
    while (i < n && isdigit(s[i])) {
        result = result * 10 + (s[i++] - '0');
        if (result * sign > INT_MAX) return INT_MAX;
        if (result * sign < INT_MIN) return INT_MIN;
    }
    return (int)(result * sign);
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
- "What's the space complexity?" → O(1) with `int[128]` vs O(Σ) with `std::unordered_map`
- "What if pattern has duplicate characters?" → Frequency-based `formed` handles it correctly

---

## Related Files

- [Amazon OA Questions](../OA-Qns/Amazon.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window%20on%20Strings.md)
- [Interview Tips — Java String API](../Interview%20Tips/Java%20String%20API.md)

> **Last Updated:** 2026-06-26
