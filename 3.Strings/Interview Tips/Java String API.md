# C++ String API Reference

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Most Used Methods (Interview Context)

| Method | Returns | Time | Notes |
|--------|---------|------|-------|
| `s[i]` | `char` | O(1) | Direct index access |
| `s.length()` or `s.size()` | `size_t` | O(1) | |
| `s.substr(l, r-l)` | `string` | O(r-l) | Second arg is length, not end index |
| `s.find(c)` | `size_t` | O(n) | First occurrence; `string::npos` if absent |
| `s.find(sub) != string::npos` | `bool` | O(n×m) | Use KMP for O(n+m) |
| `s == t` | `bool` | O(n) | Content equality |
| use `transform` + compare | `bool` | O(n) | Case-insensitive compare |
| `s.compare(t)` | `int` | O(n) | Lexicographic; 0 = equal |
| `s.rfind(p, 0) == 0` | `bool` | O(p.len) | Check prefix |
| `s.size() >= p.size() && s.compare(s.size()-p.size(), p.size(), p) == 0` | `bool` | O(p.len) | Check suffix |
| iterate string directly | — | O(n) | Range-for or index; no separate copy needed |
| manual trim with `find_first_not_of` / `find_last_not_of` | `string` | O(n) | Leading/trailing whitespace |
| manual trim (Unicode-aware) | `string` | O(n) | Use `isspace` per character |
| `stringstream` or manual split | `vector<string>` | O(n) | Split on delimiter |
| `s.replace(pos, len, newstr)` | `string&` | O(n) | Modifies in-place |
| `transform(s.begin(),s.end(),s.begin(),::tolower)` | — | O(n) | Lowercase in-place |
| `transform(s.begin(),s.end(),s.begin(),::toupper)` | — | O(n) | Uppercase in-place |
| `to_string(n)` | `string` | O(digits) | Int → string |
| `stoi(s)` | `int` | O(n) | string → int |
| manual join with ostringstream | `string` | O(total) | Join with delimiter |

## `string` (C++ equivalent of StringBuilder)

| Method | Notes |
|--------|-------|
| `s += x` or `s.append(x)` | O(1) amortized |
| `s.insert(i, str)` | O(n) |
| `s.erase(i, 1)` | O(n) |
| `reverse(s.begin(), s.end())` | O(n) |
| `s` (already a string) | O(1) — no conversion needed |
| `s.clear()` or `s = ""` | Clear without new allocation |
| `s[i]` | O(1) |
| `s.length()` | O(1) |

## Character Utility Methods

```cpp
isalpha(c)           // a-z, A-Z
isdigit(c)           // 0-9
isalnum(c)           // alphanumeric
isspace(c)           // space, tab, newline
isupper(c)
islower(c)
toupper(c)
tolower(c)
```

## Char Arithmetic

```cpp
c - 'a'              // 0-25 for lowercase
c - 'A'              // 0-25 for uppercase
c - '0'              // 0-9 for digit characters
(char)('a' + k)      // k-th lowercase letter
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-07-03
