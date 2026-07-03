# Coding Tips — Strings (C++)

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## C++ String Essentials

```cpp
#include <bits/stdc++.h>
using namespace std;
// std::string is mutable; += is O(n) amortized
string result = "";
for (auto& s : list) result += s; // OK — string is mutable in C++

// For complex formatting, prefer ostringstream
ostringstream sb;
for (auto& s : list) sb << s;
string result = sb.str();
```

## Char Access Patterns

```cpp
#include <bits/stdc++.h>
using namespace std;
char c = s[i];                     // O(1)
// iterate string directly with range-for: for (char c : s)
int freq = c - 'a';                // 0-25 for lowercase letters
int ascii = (int) c;               // full ASCII value

isalpha(c)                         // true for a-z, A-Z
isdigit(c)                         // true for 0-9
isalnum(c)                         // used in palindrome problems
tolower(c)                         // case-insensitive compare
toupper(c)
```

## Common Frequency Array Patterns

```cpp
#include <bits/stdc++.h>
using namespace std;
// Lowercase only
vector<int> freq(26, 0);
for (char c : s) freq[c - 'a']++;

// All ASCII
vector<int> freq(128, 0);
for (char c : s) freq[c]++;

// Check equal frequency
freq1 == freq2;                    // O(26) — fast
```

## String Comparison & Operations

```cpp
#include <bits/stdc++.h>
using namespace std;
s == t                             // content equality
// equalsIgnoreCase → convert both with tolower/toupper first
s.compare(t)                       // lexicographic; returns 0 if equal
s.find(sub) != string::npos        // O(n×m) — use KMP for O(n+m)
s.rfind(prefix, 0) == 0            // startsWith equivalent
s.find(c)                          // first occurrence, string::npos if not found
s.substr(l, r - l)                 // [l, r) exclusive; O(r-l)
// split on whitespace → use istringstream
s.erase(0, s.find_first_not_of(" \t\n")); // trim leading
s.erase(s.find_last_not_of(" \t\n") + 1); // trim trailing
replace(s.begin(), s.end(), 'a', 'b'); // char replacement
reverse(s.begin(), s.end());       // reverse in-place
```

## Interview Quick-Start Template

```cpp
#include <bits/stdc++.h>
using namespace std;
// 1. Always clarify charset first
// 2. Choose data structure:
//    - vector<int>(26, 0)   → lowercase only
//    - vector<int>(128, 0)  → ASCII
//    - unordered_map        → Unicode or unknown

// 3. Sliding window skeleton
int l = 0, maxLen = 0;
unordered_map<char, int> window;
for (int r = 0; r < (int)s.length(); r++) {
    window[s[r]]++;
    while (/* invalid */) {
        window[s[l]]--;
        if (window[s[l]] == 0) window.erase(s[l]);
        l++;
    }
    maxLen = max(maxLen, r - l + 1);
}
```

---

## Related Files

- [Java String API](./Java%20String%20API.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
