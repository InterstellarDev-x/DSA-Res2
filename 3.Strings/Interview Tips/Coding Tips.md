# Coding Tips — Strings (Java)

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Java String Essentials

```java
// Never do this in a loop — O(n²) due to immutability
String result = "";
for (String s : list) result += s; // BAD

// Always use StringBuilder — O(n) amortized
StringBuilder sb = new StringBuilder();
for (String s : list) sb.append(s); // GOOD
String result = sb.toString();
```

## Char Access Patterns

```java
char c = s.charAt(i);              // O(1)
char[] arr = s.toCharArray();      // O(n) copy — do once, index freely
int freq = c - 'a';                // 0-25 for lowercase letters
int ascii = (int) c;               // full ASCII value

Character.isLetter(c)              // true for a-z, A-Z
Character.isDigit(c)               // true for 0-9
Character.isLetterOrDigit(c)       // used in palindrome problems
Character.toLowerCase(c)           // case-insensitive compare
Character.toUpperCase(c)
```

## Common Frequency Array Patterns

```java
// Lowercase only
int[] freq = new int[26];
for (char c : s.toCharArray()) freq[c - 'a']++;

// All ASCII
int[] freq = new int[128];
for (char c : s.toCharArray()) freq[c]++;

// Check equal frequency
Arrays.equals(freq1, freq2);       // O(26) — fast
```

## String Comparison & Operations

```java
s.equals(t)                        // content equality (not ==)
s.equalsIgnoreCase(t)
s.compareTo(t)                     // lexicographic; returns 0 if equal
s.contains(sub)                    // O(n×m) — use KMP for O(n+m)
s.startsWith(prefix)
s.indexOf(c)                       // first occurrence, -1 if not found
s.substring(l, r)                  // [l, r) exclusive; O(r-l)
s.split("\\s+")                    // split on whitespace
s.trim()                           // remove leading/trailing whitespace
s.replace('a', 'b')                // char replacement
new StringBuilder(s).reverse().toString() // reverse
```

## Interview Quick-Start Template

```java
// 1. Always clarify charset first
// 2. Choose data structure:
//    - int[26]  → lowercase only
//    - int[128] → ASCII
//    - HashMap  → Unicode or unknown

// 3. Sliding window skeleton
int l = 0, maxLen = 0;
Map<Character, Integer> window = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    window.merge(s.charAt(r), 1, Integer::sum);
    while (/* invalid */) {
        window.merge(s.charAt(l), -1, Integer::sum);
        if (window.get(s.charAt(l)) == 0) window.remove(s.charAt(l));
        l++;
    }
    maxLen = Math.max(maxLen, r - l + 1);
}
```

---

## Related Files

- [Java String API](./Java%20String%20API.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
