# Common Mistakes — Strings

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Top String Bugs

| # | Mistake | Buggy Code | Fix |
|---|---------|-----------|-----|
| 1 | String concatenation in loop | `result += s` in a loop | use `ostringstream` or `string::reserve()` + `+=` |
| 2 | `==` for string equality (Java pitfall) | `if (s == t)` | In C++, `==` works correctly for `std::string` value comparison |
| 3 | `split(" ")` misses multiple spaces | `"  a  b".split(" ")` gives empty tokens | Use `istringstream` to skip whitespace tokens |
| 4 | `c - 'a'` on uppercase char | `'A' - 'a'` = -32 → array index -32 | Lowercase first: `tolower(c) - 'a'` |
| 5 | `int[26]` for mixed-case input | Index out of bounds for uppercase | Use `int[128]` or normalize to lowercase |
| 6 | Not checking empty string | `s[0]` on `""` | Guard: `if (s.empty()) return ...` |
| 7 | Sliding window: `if` instead of `while` to shrink | One shrink may not be enough | Always `while (invalid)` |
| 8 | KMP: `i++` after `len = lps[len-1]` | Wrong LPS values | Do NOT advance `i` when falling back |
| 9 | Palindrome: not trying both skip directions | `isPalin(s, l+1, r)` only | Also try `isPalin(s, l, r-1)` |
| 10 | `substr(pos, len)` takes length, not end index | `s.substr(0, 3)` returns chars 0,1,2 (length=3) | For inclusive `r`: `s.substr(l, r-l+1)` |

---

## Sliding Window String Mistakes

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG: frequency key bug
string key = "";
for (auto f : freq) key += to_string(f);
// "12" and "1" + "2" collision → use separator
key += '#'; key += to_string(f); // CORRECT: "#1#2#..."

// WRONG: formed counter uses >=
if (window[rc] >= need[rc]) formed++; // double-counts
// CORRECT: only exact match triggers formed
if (need[rc] > 0 && window[rc] == need[rc]) formed++;
```

---

## Palindrome Mistakes

```cpp
#include <bits/stdc++.h>
using namespace std;

// OK: comparing chars directly (no boxing issue in C++)
if (s[l] == s[r]) // OK for char (C++ strings return char, not a boxed type)
// Note: unlike Java, there is no "boxed Character" pitfall in C++

// WRONG: palindrome II only trying one skip direction
if (s[l] != s[r])
    return isPalin(s, l+1, r); // WRONG - misses cases
    // CORRECT:
    return isPalin(s, l+1, r) || isPalin(s, l, r-1);
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Java String API](./Java%20String%20API.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
