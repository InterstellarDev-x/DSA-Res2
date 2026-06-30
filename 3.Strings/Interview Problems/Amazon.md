# Amazon — Strings Interview Questions

> **Topic:** [Strings](../README.md) · **Section:** Interview Problems
> **Tags:** `amazon` `interview` `strings`

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **Java String immutability** | Use `StringBuilder`; never `+` in a loop |
| **Char frequency** | `int[26]` over `HashMap` when possible |
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

```java
public String[] reorderLogFiles(String[] logs) {
    List<String> letters = new ArrayList<>(), digits = new ArrayList<>();
    for (String log : logs) {
        if (Character.isDigit(log.split(" ")[1].charAt(0))) digits.add(log);
        else letters.add(log);
    }
    letters.sort((a, b) -> {
        String ca = a.substring(a.indexOf(' ') + 1);
        String cb = b.substring(b.indexOf(' ') + 1);
        int cmp = ca.compareTo(cb);
        return cmp != 0 ? cmp : a.compareTo(b); // tie-break by identifier
    });
    letters.addAll(digits);
    return letters.toArray(new String[0]);
}
```
**Follow-ups:** "What if two logs have the same content AND same identifier?" → they're equal, preserve original order → `Arrays.sort` is stable in Java.

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
- "Unicode chars?" → Use `HashMap<Character, Integer>` instead of `int[128]`

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

```java
public int myAtoi(String s) {
    int i = 0, n = s.length(), sign = 1;
    long result = 0;
    while (i < n && s.charAt(i) == ' ') i++;           // skip spaces
    if (i < n && (s.charAt(i) == '+' || s.charAt(i) == '-'))
        sign = s.charAt(i++) == '-' ? -1 : 1;
    while (i < n && Character.isDigit(s.charAt(i))) {
        result = result * 10 + (s.charAt(i++) - '0');
        if (result * sign > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (result * sign < Integer.MIN_VALUE) return Integer.MIN_VALUE;
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
- "What's the space complexity?" → O(1) with `int[128]` vs O(Σ) with HashMap
- "What if pattern has duplicate characters?" → Frequency-based `formed` handles it correctly

---

## Related Files

- [Amazon OA Questions](../OA-Qns/Amazon.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window%20on%20Strings.md)
- [Interview Tips — Java String API](../Interview%20Tips/Java%20String%20API.md)

> **Last Updated:** 2026-06-26
