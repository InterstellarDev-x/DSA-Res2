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

```java
// Always use ArrayDeque — not Stack<> (legacy, synchronized, slow)
Deque<Character> stack = new ArrayDeque<>();
stack.push(c);          // push to top
stack.pop();            // remove from top
stack.peek();           // view top without removing
stack.isEmpty();        // check empty
```

---

## Pattern: Matching Brackets

**Template:**

```java
Map<Character, Character> map = Map.of(')', '(', ']', '[', '}', '{');
Deque<Character> stack = new ArrayDeque<>();
for (char c : s.toCharArray()) {
    if (!map.containsKey(c)) {        // opening bracket
        stack.push(c);
    } else {                          // closing bracket
        if (stack.isEmpty() || stack.peek() != map.get(c)) return false;
        stack.pop();
    }
}
return stack.isEmpty();
```

**Why `stack.isEmpty()` at end?** Unclosed `((` would leave items in stack — must be empty for valid.

---

## Problem 1: Valid Parentheses — LC 20

**Input:** `s = "()[]{}"` → `true`; `s = "([)]"` → `false`

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if (c == ')' && top != '(') return false;
            if (c == ']' && top != '[') return false;
            if (c == '}' && top != '{') return false;
        }
    }
    return stack.isEmpty();
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

```java
public boolean backspaceCompare(String s, String t) {
    return process(s).equals(process(t));
}

private String process(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        if (c != '#') {
            stack.push(c);
        } else if (!stack.isEmpty()) {
            stack.pop();
        }
    }
    StringBuilder sb = new StringBuilder();
    while (!stack.isEmpty()) sb.append(stack.pop());
    return sb.toString();
}
```

### Approach 2: Two Pointers from End — O(n) time, O(1) space

```java
public boolean backspaceCompare(String s, String t) {
    int i = s.length() - 1, j = t.length() - 1;
    int skipS = 0, skipT = 0;
    while (i >= 0 || j >= 0) {
        while (i >= 0) {
            if (s.charAt(i) == '#') { skipS++; i--; }
            else if (skipS > 0) { skipS--; i--; }
            else break;
        }
        while (j >= 0) {
            if (t.charAt(j) == '#') { skipT++; j--; }
            else if (skipT > 0) { skipT--; j--; }
            else break;
        }
        if (i >= 0 && j >= 0 && s.charAt(i) != t.charAt(j)) return false;
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

```java
public String removeOuterParentheses(String s) {
    StringBuilder sb = new StringBuilder();
    int depth = 0;
    for (char c : s.toCharArray()) {
        if (c == '(') {
            if (depth > 0) sb.append(c);   // append inner (not outermost opening)
            depth++;
        } else {
            depth--;
            if (depth > 0) sb.append(c);   // append inner (not outermost closing)
        }
    }
    return sb.toString();
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

```java
public String simplifyPath(String path) {
    Deque<String> stack = new ArrayDeque<>();
    for (String part : path.split("/")) {
        if (part.equals("..")) {
            if (!stack.isEmpty()) stack.pop();
        } else if (!part.isEmpty() && !part.equals(".")) {
            stack.push(part);
        }
    }
    StringBuilder sb = new StringBuilder();
    // stack has most recent on top — need to reverse
    Deque<String> reversed = new ArrayDeque<>();
    while (!stack.isEmpty()) reversed.push(stack.pop());
    while (!reversed.isEmpty()) sb.append("/").append(reversed.pop());
    return sb.length() == 0 ? "/" : sb.toString();
}
```

**Alternative: Use LinkedList as stack with addFirst/pollFirst — iterate with iterator for correct order.**

```java
public String simplifyPath(String path) {
    LinkedList<String> stack = new LinkedList<>();
    for (String part : path.split("/")) {
        if (part.equals("..")) { if (!stack.isEmpty()) stack.pollLast(); }
        else if (!part.isEmpty() && !part.equals(".")) stack.addLast(part);
    }
    return "/" + String.join("/", stack);
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
A: Yes — the Map approach handles `N` types without code changes.

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
