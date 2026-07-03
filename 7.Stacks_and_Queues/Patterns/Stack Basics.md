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

```rust
// Always use Vec<T> as a stack in Rust
let mut stk: Vec<char> = Vec::new();
stk.push(c);          // push to top
stk.pop();            // remove from top (returns Option<char>)
stk.last();           // view top without removing (returns Option<&char>)
stk.is_empty();       // check empty
```

---

## Pattern: Matching Brackets

**Template:**

```rust
use std::collections::HashMap;
let map: HashMap<char, char> = [(')', '('), (']', '['), ('}', '{')].iter().cloned().collect();
let mut stk: Vec<char> = Vec::new();
for c in s.chars() {
    if !map.contains_key(&c) {              // opening bracket
        stk.push(c);
    } else {                               // closing bracket
        if stk.is_empty() || stk.last() != Some(&map[&c]) { return false; }
        stk.pop();
    }
}
stk.is_empty()
```

**Why `stk.is_empty()` at end?** Unclosed `((` would leave items in stack — must be empty for valid.

---

## Problem 1: Valid Parentheses — LC 20

**Input:** `s = "()[]{}"` → `true`; `s = "([)]"` → `false`

```rust
fn is_valid(s: String) -> bool {
    let mut stk: Vec<char> = Vec::new();
    for c in s.chars() {
        if c == '(' || c == '[' || c == '{' {
            stk.push(c);
        } else {
            if stk.is_empty() { return false; }
            let top = stk.pop().unwrap();
            if c == ')' && top != '(' { return false; }
            if c == ']' && top != '[' { return false; }
            if c == '}' && top != '{' { return false; }
        }
    }
    stk.is_empty()
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

```rust
fn process(s: &str) -> String {
    let mut stk: Vec<char> = Vec::new();
    for c in s.chars() {
        if c != '#' {
            stk.push(c);
        } else if !stk.is_empty() {
            stk.pop();
        }
    }
    stk.iter().collect()
}

fn backspace_compare(s: String, t: String) -> bool {
    process(&s) == process(&t)
}
```

### Approach 2: Two Pointers from End — O(n) time, O(1) space

```rust
fn backspace_compare(s: String, t: String) -> bool {
    let s: Vec<char> = s.chars().collect();
    let t: Vec<char> = t.chars().collect();
    let mut i = s.len() as i32 - 1;
    let mut j = t.len() as i32 - 1;
    let mut skip_s = 0i32;
    let mut skip_t = 0i32;
    while i >= 0 || j >= 0 {
        while i >= 0 {
            if s[i as usize] == '#' { skip_s += 1; i -= 1; }
            else if skip_s > 0 { skip_s -= 1; i -= 1; }
            else { break; }
        }
        while j >= 0 {
            if t[j as usize] == '#' { skip_t += 1; j -= 1; }
            else if skip_t > 0 { skip_t -= 1; j -= 1; }
            else { break; }
        }
        if i >= 0 && j >= 0 && s[i as usize] != t[j as usize] { return false; }
        if (i >= 0) != (j >= 0) { return false; }
        i -= 1;
        j -= 1;
    }
    true
}
```

**Why scan from right?** `#` cancels the character before it. Scanning left-to-right requires building the full string first. Scanning right-to-left, we can count pending backspaces and skip accordingly without any extra storage.

---

## Problem 3: Remove Outermost Parentheses — LC 1021

**Input:** `"(()())(())"` → `"()()()"`
**Rule:** Decompose into primitive parenthesizations (balanced, no proper prefix is balanced). Remove outer layer of each.

```rust
fn remove_outer_parentheses(s: String) -> String {
    let mut result = String::new();
    let mut depth = 0;
    for c in s.chars() {
        if c == '(' {
            if depth > 0 { result.push(c); }  // append inner (not outermost opening)
            depth += 1;
        } else {
            depth -= 1;
            if depth > 0 { result.push(c); }  // append inner (not outermost closing)
        }
    }
    result
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

```rust
fn simplify_path(path: String) -> String {
    let mut stk: Vec<String> = Vec::new();
    for part in path.split('/') {
        if part == ".." {
            if !stk.is_empty() { stk.pop(); }
        } else if !part.is_empty() && part != "." {
            stk.push(part.to_string());
        }
    }
    let mut result = String::new();
    // Vec has most recent at end — reverse into another Vec for correct order
    let mut reversed: Vec<String> = Vec::new();
    while let Some(top) = stk.pop() { reversed.push(top); }
    while let Some(top) = reversed.pop() {
        result.push('/');
        result.push_str(&top);
    }
    if result.is_empty() { "/".to_string() } else { result }
}
```

**Alternative: Use `Vec<String>` as stack with `push`/`pop` — iterate directly for correct order.**

```rust
fn simplify_path(path: String) -> String {
    let mut stk: Vec<String> = Vec::new();
    for part in path.split('/') {
        if part == ".." { if !stk.is_empty() { stk.pop(); } }
        else if !part.is_empty() && part != "." { stk.push(part.to_string()); }
    }
    let mut result = String::new();
    for s in &stk { result.push('/'); result.push_str(s); }
    if result.is_empty() { "/".to_string() } else { result }
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
A: Yes — the `HashMap` approach handles `N` types without code changes.

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
