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
5. [Java Templates](#java-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Two Pointers on strings applies the same opposite-ends or same-direction pointer logic as on arrays, with one key difference: **Java strings are immutable**, so in-place operations require converting to `char[]`.

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
| Palindrome check | O(n) | O(1) if on `char[]`; O(n) if `toCharArray()` |
| Reverse string | O(n) | O(1) in-place |
| Reverse words | O(n) | O(n) Java (immutable string) |
| Two-string compare | O(m+n) | O(1) |

---

## Java Templates

### 1. Valid Palindrome (Ignore Non-Alphanumeric, LC 125)

```java
public boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
        while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
        if (Character.toLowerCase(s.charAt(l)) !=
            Character.toLowerCase(s.charAt(r))) return false;
        l++; r--;
    }
    return true;
}
// Time: O(n) | Space: O(1)
```

### 2. Valid Palindrome II — At Most One Deletion (LC 680)

```java
public boolean validPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l) != s.charAt(r))
            return isPalin(s, l + 1, r) || isPalin(s, l, r - 1);
        l++; r--;
    }
    return true;
}

private boolean isPalin(String s, int l, int r) {
    while (l < r) {
        if (s.charAt(l++) != s.charAt(r--)) return false;
    }
    return true;
}
// Time: O(n) | Space: O(1)
```

### 3. Reverse String In-Place (LC 344)

```java
public void reverseString(char[] s) {
    int l = 0, r = s.length - 1;
    while (l < r) {
        char tmp = s[l]; s[l++] = s[r]; s[r--] = tmp;
    }
}
// Time: O(n) | Space: O(1)
```

### 4. Reverse Words in a String (LC 151)

```java
public String reverseWords(String s) {
    // Trim, split on whitespace, reverse word order
    String[] words = s.trim().split("\\s+");
    int l = 0, r = words.length - 1;
    while (l < r) { String tmp = words[l]; words[l++] = words[r]; words[r--] = tmp; }
    return String.join(" ", words);
}
// Time: O(n) | Space: O(n) — Java string immutability forces copy
```

### 5. Rotate String — Three-Reverse Trick (LC 796)

```java
// Check if s can become goal by rotation
public boolean rotateString(String s, String goal) {
    return s.length() == goal.length() && (s + s).contains(goal);
    // Alternative: KMP search in doubled string
}
// Time: O(n²) with contains; O(n) with KMP
```

### 6. Compare Version Numbers (LC 165)

```java
public int compareVersion(String version1, String version2) {
    int i = 0, j = 0;
    while (i < version1.length() || j < version2.length()) {
        int v1 = 0, v2 = 0;
        while (i < version1.length() && version1.charAt(i) != '.') v1 = v1 * 10 + (version1.charAt(i++) - '0');
        while (j < version2.length() && version2.charAt(j) != '.') v2 = v2 * 10 + (version2.charAt(j++) - '0');
        if (v1 != v2) return v1 > v2 ? 1 : -1;
        i++; j++; // skip '.'
    }
    return 0;
}
// Time: O(m+n) | Space: O(1)
```

### 7. Long Pressed Name (LC 925)

```java
public boolean isLongPressedName(String name, String typed) {
    int i = 0, j = 0;
    while (j < typed.length()) {
        if (i < name.length() && name.charAt(i) == typed.charAt(j)) { i++; j++; }
        else if (j > 0 && typed.charAt(j) == typed.charAt(j - 1)) j++;
        else return false;
    }
    return i == name.length();
}
// Time: O(m+n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Palindrome: not converting to lowercase | `Character.toLowerCase()` both chars before comparing |
| Palindrome: not skipping non-alphanumeric | Use `Character.isLetterOrDigit()` guard inside while |
| Reverse words: using `split(" ")` | Use `split("\\s+")` then `.trim()` to handle multiple spaces |
| In-place reverse on `String` | Must convert to `char[]` first; Java strings are immutable |
| Two-string pointer: forgetting to advance both `i` and `j` | After a match, both pointers advance |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Palindrome with ignored chars | Two pointer + `Character.isLetterOrDigit()` filter |
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
