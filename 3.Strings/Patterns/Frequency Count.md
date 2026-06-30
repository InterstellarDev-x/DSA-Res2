# Frequency Count Pattern

> **Topic:** [Strings](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `frequency` `anagram` `char-array` `hashmap`

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

Frequency Count reduces string comparison problems to **counting character occurrences** in an array of size 26 (lowercase) or 128/256 (ASCII). Two strings with identical frequency arrays are anagrams; a frequency array can be used as a canonical hash key.

**Key data structures:**

| Structure | Size | Use When |
|-----------|------|----------|
| `int[26]` | 26 | Only lowercase letters `a–z` |
| `int[128]` | 128 | All ASCII characters |
| `int[256]` | 256 | Extended ASCII |
| `HashMap<Character, Integer>` | Dynamic | Unicode or unknown charset |

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
| Build frequency array | O(n) | O(1) — `int[26]` |
| Compare two frequency arrays | O(26) = O(1) | O(1) |
| Group anagrams (n strings, avg len k) | O(n × k) | O(n × k) |
| `Arrays.equals` on `int[26]` | O(26) = O(1) | O(1) |

---

## Java Templates

### 1. Valid Anagram (LC 242)

```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (char c : t.toCharArray()) freq[c - 'a']--;
    for (int f : freq) if (f != 0) return false;
    return true;
}
// Time: O(n) | Space: O(1)
```

### 2. Group Anagrams (LC 49) — Sorted Key

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Time: O(n × k log k) | Space: O(n × k)
```

### 3. Group Anagrams — Frequency Key (No Sort, O(n×k))

```java
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;
        // Build canonical key: "#2#0#1#..."
        StringBuilder key = new StringBuilder();
        for (int f : freq) key.append('#').append(f);
        map.computeIfAbsent(key.toString(), k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
// Time: O(n × k) | Space: O(n × k)
```

### 4. Ransom Note (LC 383)

```java
public boolean canConstruct(String ransomNote, String magazine) {
    int[] freq = new int[26];
    for (char c : magazine.toCharArray()) freq[c - 'a']++;
    for (char c : ransomNote.toCharArray()) {
        if (--freq[c - 'a'] < 0) return false;
    }
    return true;
}
// Time: O(m + n) | Space: O(1)
```

### 5. First Unique Character (LC 387)

```java
public int firstUniqChar(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (int i = 0; i < s.length(); i++)
        if (freq[s.charAt(i) - 'a'] == 1) return i;
    return -1;
}
// Time: O(n) | Space: O(1)
```

### 6. Minimum Steps to Make Two Strings Anagram (LC 1347)

```java
public int minSteps(String s, String t) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    for (char c : t.toCharArray()) freq[c - 'a']--;
    int steps = 0;
    for (int f : freq) if (f > 0) steps += f; // count chars s has in excess
    return steps;
}
// Time: O(n) | Space: O(1)
```

### 7. Sort Characters By Frequency (LC 451)

```java
public String frequencySort(String s) {
    int[] freq = new int[128];
    for (char c : s.toCharArray()) freq[c]++;

    // Sort characters by frequency descending
    List<Character> chars = new ArrayList<>();
    for (int i = 0; i < 128; i++) if (freq[i] > 0) chars.add((char) i);
    chars.sort((a, b) -> freq[b] - freq[a]);

    StringBuilder sb = new StringBuilder();
    for (char c : chars) {
        for (int i = 0; i < freq[c]; i++) sb.append(c);
    }
    return sb.toString();
}
// Time: O(n + 128 log 128) = O(n) | Space: O(n)
```

### 8. Isomorphic Strings (LC 205)

```java
public boolean isIsomorphic(String s, String t) {
    int[] sToT = new int[256], tToS = new int[256];
    for (int i = 0; i < s.length(); i++) {
        int sc = s.charAt(i), tc = t.charAt(i);
        if (sToT[sc] == 0 && tToS[tc] == 0) {
            sToT[sc] = tc; tToS[tc] = sc;
        } else if (sToT[sc] != tc || tToS[tc] != sc) return false;
    }
    return true;
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only checking one direction for isomorphic | Check both `s→t` and `t→s` mappings |
| Group anagrams: using `Arrays.toString(freq)` as key | `[1,0,0...]` has spaces; use `#` separator to distinguish `[12,0]` from `[1,20]` |
| Not checking length before anagram check | Early return if `s.length() != t.length()` |
| Using `HashMap` when `int[26]` suffices | 26x faster; always prefer for lowercase-only problems |
| `c - 'a'` on uppercase or non-alpha chars | Guard with `Character.isLowerCase(c)` or use `int[128]` |

---

## Variations

| Variation | Key Idea |
|-----------|----------|
| Scramble string | Frequency check + recursion |
| Check if subset anagram | All freqs of s ≤ freqs of t |
| Minimum window anagram | See [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) |
| Word pattern (LC 290) | Two-way HashMap mapping |

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

> **Interview Tip:** For `groupAnagrams`, the frequency-based key (`"#2#0#1..."`) is O(n×k) vs sorted key O(n×k log k). Mentioning both and explaining the trade-off signals strong algorithmic thinking.

> **Last Updated:** 2026-06-26
