# Stack Basics

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 1 of 5**
> **Problems:** Valid Parentheses · Backspace String Compare · Remove Outermost Parentheses · Simplify Path

---

## Core Concept

A stack is LIFO (Last-In-First-Out). It naturally handles:
- **Matching/nesting**: brackets, parentheses, HTML tags
- **Undo/backspace**: cancel the last action
- **Path resolution**: directory traversal with `.` and `..`
- **DFS state**: explicit stack instead of call stack

```cpp
#include <bits/stdc++.h>
using namespace std;
// Always use stack<> from STL
stack<char> stk;
stk.push(c);          // push to top
stk.pop();            // remove from top
stk.top();            // view top without removing
stk.empty();          // check empty
```

---

## Pattern: Matching Brackets

**Template:**

```cpp
#include <bits/stdc++.h>
using namespace std;
unordered_map<char, char> map = {{')', '('}, {']', '['}, {'}', '{'}};
stack<char> stk;
for (auto& c : s) {
    if (!map.count(c)) {              // opening bracket
        stk.push(c);
    } else {                          // closing bracket
        if (stk.empty() || stk.top() != map[c]) return false;
        stk.pop();
    }
}
return stk.empty();
```

**Why `stk.empty()` at end?** Unclosed `((` would leave items in stack — must be empty for valid.

---

## Problem 1: Valid Parentheses — LC 20

**Input:** `s = "()[]{}"` → `true`; `s = "([)]"` → `false`

```cpp
#include <bits/stdc++.h>
using namespace std;
bool isValid(string s) {
    stack<char> stk;
    for (auto& c : s) {
        if (c == '(' || c == '[' || c == '{') {
            stk.push(c);
        } else {
            if (stk.empty()) return false;
            char top = stk.top(); stk.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stk.empty();
}
```

**Complexity:** O(n) time, O(n) space

**Edge cases:**
- `""` → `true` (empty string is valid)
- `"]"` → `false` (closing without opening)
- `"((("` → `false` (unclosed)
- `"([)]"` → `false` (interleaved)

---

## Problem 2: Backspace String Compare — LC 844

**Input:** `s = "ab#c"`, `t = "ad#c"` → `true` (both become `"ac"`)

### Approach 1: Stack Simulation — O(n) time, O(n) space

```cpp
#include <bits/stdc++.h>
using namespace std;
string process(string s) {
    stack<char> stk;
    for (auto& c : s) {
        if (c != '#') {
            stk.push(c);
        } else if (!stk.empty()) {
            stk.pop();
        }
    }
    string result = "";
    while (!stk.empty()) { result += stk.top(); stk.pop(); }
    return result;
}

bool backspaceCompare(string s, string t) {
    return process(s) == process(t);
}
```

### Approach 2: Two Pointers from End — O(n) time, O(1) space

```cpp
#include <bits/stdc++.h>
using namespace std;
bool backspaceCompare(string s, string t) {
    int i = s.length() - 1, j = t.length() - 1;
    int skipS = 0, skipT = 0;
    while (i >= 0 || j >= 0) {
        while (i >= 0) {
            if (s[i] == '#') { skipS++; i--; }
            else if (skipS > 0) { skipS--; i--; }
            else break;
        }
        while (j >= 0) {
            if (t[j] == '#') { skipT++; j--; }
            else if (skipT > 0) { skipT--; j--; }
            else break;
        }
        if (i >= 0 && j >= 0 && s[i] != t[j]) return false;
        if ((i >= 0) != (j >= 0)) return false;
        i--; j--;
    }
    return true;
}
```

**Why scan from right?** `#` cancels the character before it. Scanning left-to-right requires building the full string first. Scanning right-to-left, we can count pending backspaces and skip accordingly without any extra storage.

---

## Problem 3: Remove Outermost Parentheses — LC 1021

**Input:** `"(()())(())"` → `"()()()"`
**Rule:** Decompose into primitive parenthesizations (balanced, no proper prefix is balanced). Remove outer layer of each.

```cpp
#include <bits/stdc++.h>
using namespace std;
string removeOuterParentheses(string s) {
    string result = "";
    int depth = 0;
    for (auto& c : s) {
        if (c == '(') {
            if (depth > 0) result += c;   // append inner (not outermost opening)
            depth++;
        } else {
            depth--;
            if (depth > 0) result += c;   // append inner (not outermost closing)
        }
    }
    return result;
}
```

**Key insight:** A `(` at depth 0 before increment is the outermost opening — skip it. A `)` at depth 0 after decrement is the outermost closing — skip it. Everything else is inner content.

**Complexity:** O(n) time, O(n) space (output)

---

## Problem 4: Simplify Path — LC 71

**Input:** `"/home//foo/../bar/"` → `"/home/bar"`

Rules:
- `.` = current directory — ignore
- `..` = parent directory — pop stack
- `""` = multiple slashes — ignore
- Anything else = valid directory name — push

```cpp
#include <bits/stdc++.h>
using namespace std;
string simplifyPath(string path) {
    stack<string> stk;
    stringstream ss(path);
    string part;
    while (getline(ss, part, '/')) {
        if (part == "..") {
            if (!stk.empty()) stk.pop();
        } else if (!part.empty() && part != ".") {
            stk.push(part);
        }
    }
    string result = "";
    // stack has most recent on top — need to reverse
    stack<string> reversed;
    while (!stk.empty()) { reversed.push(stk.top()); stk.pop(); }
    while (!reversed.empty()) { result += "/" + reversed.top(); reversed.pop(); }
    return result.empty() ? "/" : result;
}
```

**Alternative: Use `std::vector<string>` as stack with `push_back`/`pop_back` — iterate directly for correct order.**

```cpp
#include <bits/stdc++.h>
using namespace std;
string simplifyPath(string path) {
    vector<string> stk;
    stringstream ss(path);
    string part;
    while (getline(ss, part, '/')) {
        if (part == "..") { if (!stk.empty()) stk.pop_back(); }
        else if (!part.empty() && part != ".") stk.push_back(part);
    }
    string result = "";
    for (auto& s : stk) result += "/" + s;
    return result.empty() ? "/" : result;
}
```

**Complexity:** O(n) time, O(n) space

---

## Pattern Recognition

| Signal | Use Stack Basics |
|--------|-----------------|
| `(`, `[`, `{` matching | ✅ Classic bracket matching |
| "undo last action" / `#` backspace | ✅ Stack pop |
| `..` in paths / undo navigation | ✅ Stack pop |
| "depth" tracking | ✅ Counter instead of full stack |
| Nested structure (outer vs inner) | ✅ Depth counter trick |

---

## Common Follow-ups

**Q: Can Valid Parentheses handle multiple types of brackets?**
A: Yes — the `std::unordered_map` approach handles `N` types without code changes.

**Q: Backspace Compare with O(1) space?**
A: Two-pointer from right (shown above) — skip characters using counters.

**Q: Simplify Path if path is absolute vs relative?**
A: For absolute, always start from root. For relative, would need current directory context passed in.

---

## Related Files

- [Queue & Deque](./Queue%20and%20Deque.md)
- [Monotonic Stack](./Monotonic%20Stack.md)
- [Stack for Expressions](./Stack%20for%20Expressions.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
