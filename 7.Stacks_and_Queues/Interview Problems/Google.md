# Google — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Google
> [← Google OA](../OA-Qns/Google.md)

---

## Problem 1: Maximum Frequency Stack — Deep Dive

**LC 895** · Hard · O(1) push, O(1) pop

### Full Solution

```cpp
#include <bits/stdc++.h>
using namespace std;

class FreqStack {
    unordered_map<int, int> freq;
    unordered_map<int, stack<int>> group;
    int maxFreq = 0;

public:
    void push(int val) {
        int f = ++freq[val];  // freq[val]++, get new freq
        maxFreq = max(maxFreq, f);
        group[f].push(val);
    }

    int pop() {
        auto& stk = group[maxFreq];
        int val = stk.top(); stk.pop();
        --freq[val];          // freq[val]--
        if (stk.empty()) { group.erase(maxFreq--); }
        return val;
    }
};
```

**Q: Why does `maxFreq` always decrement by exactly 1 (never skip)?**
A: When `group[maxFreq]` becomes empty, `maxFreq - 1` is guaranteed non-empty. Here's why: the element we just popped was in `group[maxFreq]`. Before it reached frequency `maxFreq`, it was in `group[maxFreq - 1]`. Other elements that were in `group[maxFreq - 1]` haven't been popped yet (we always pop from `maxFreq`), so `group[maxFreq - 1]` still has elements.

**Q: What if we need to also support `top()` (peek without pop)?**
A: `return group[maxFreq].top()` — O(1).

**Q: Design challenge: What if capacity limit N is added?**
A: On `push`, if `freq.size() == N` and `val` is not in `freq`: evict the element at `group[1]` (lowest frequency, and if tie, oldest = bottom of that group's stack). Then insert new element. This turns it into an LFU cache with stack-based tie-breaking.

---

## Problem 2: Basic Calculator (All Variants)

Google often asks you to solve all three variants in sequence within a single interview.

### LC 227: Basic Calculator II (`+`, `-`, `*`, `/`, no parens)

```cpp
#include <bits/stdc++.h>
using namespace std;

int calculate(string s) {
    stack<int> stk;
    int num = 0; char sign = '+';
    for (int i = 0; i < (int)s.length(); i++) {
        char c = s[i];
        if (isdigit(c)) num = num * 10 + (c - '0');
        if ((!isdigit(c) && c != ' ') || i == (int)s.length() - 1) {
            if (sign == '+') stk.push(num);
            else if (sign == '-') stk.push(-num);
            else if (sign == '*') { int t = stk.top(); stk.pop(); stk.push(t * num); }
            else if (sign == '/') { int t = stk.top(); stk.pop(); stk.push(t / num); }
            sign = c; num = 0;
        }
    }
    int result = 0;
    while (!stk.empty()) { result += stk.top(); stk.pop(); }
    return result;
}
```

### LC 224: Basic Calculator (`+`, `-`, parens, no `*`/`/`)

```cpp
#include <bits/stdc++.h>
using namespace std;

int calculate(string s) {
    stack<int> stk;
    int result = 0, num = 0, sign = 1;
    for (char c : s) {
        if (isdigit(c)) num = num * 10 + (c - '0');
        else if (c == '+') { result += sign * num; num = 0; sign = 1; }
        else if (c == '-') { result += sign * num; num = 0; sign = -1; }
        else if (c == '(') { stk.push(result); stk.push(sign); result = 0; sign = 1; }
        else if (c == ')') {
            result += sign * num; num = 0;
            int s1 = stk.top(); stk.pop();
            int s2 = stk.top(); stk.pop();
            result = result * s1 + s2;
        }
    }
    return result + sign * num;
}
```

### LC 772: Basic Calculator III (all operators + parens)

Combine both: when encountering `(`, push current accumulated result and sign; apply `*/` immediately via stack; on `)`, flush and restore.

```cpp
#include <bits/stdc++.h>
using namespace std;

int parse(const string& s, int& idx) {
    stack<int> stk;
    int num = 0; char sign = '+';
    while (idx < (int)s.length()) {
        char c = s[idx++];
        if (isdigit(c)) num = num * 10 + (c - '0');
        else if (c == '(') num = parse(s, idx);   // recurse for subexpr
        if ((!isdigit(c) && c != ' ') || idx == (int)s.length()) {
            if (sign == '+') stk.push(num);
            else if (sign == '-') stk.push(-num);
            else if (sign == '*') { int t = stk.top(); stk.pop(); stk.push(t * num); }
            else if (sign == '/') { int t = stk.top(); stk.pop(); stk.push(t / num); }
            sign = c; num = 0;
            if (c == ')') break;  // return from recursion
        }
    }
    int result = 0;
    while (!stk.empty()) { result += stk.top(); stk.pop(); }
    return result;
}

int calculate(string s) {
    // Recursive descent approach (Google prefers this at L5+)
    int idx = 0;
    return parse(s, idx);
}
```

**Q: How does the recursive approach handle nested parentheses?**
A: On `(`, we recurse with `idx` as a shared reference so both levels advance through the same string. On `)`, the inner call breaks out of its loop and returns. The outer call receives the inner expression's value as `num` and continues.

---

## Problem 3: Number of Visible People in Queue — Google L4

**LC 1944** · Hard · O(n) time

`heights[i]` = height of person i. Person `i` can see person `j` (i < j) if everyone between them is shorter than both `heights[i]` and `heights[j]`.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> canSeePersonsCount(vector<int>& heights) {
    int n = heights.size();
    vector<int> result(n);
    stack<int> stk;  // decreasing stack of heights

    for (int i = n - 1; i >= 0; i--) {
        int count = 0;
        // Count how many people to the right are visible
        while (!stk.empty() && stk.top() < heights[i]) {
            stk.pop();
            count++;
        }
        // The first person not popped (taller or equal) is also visible if it exists
        if (!stk.empty()) count++;
        result[i] = count;
        stk.push(heights[i]);
    }
    return result;
}
```

**Q: Why scan right-to-left?**
A: We want to know what person `i` can see to their right. Scanning right-to-left, the stack represents people seen so far (to the right). People shorter than `heights[i]` are visible (and are popped). The first person taller than `heights[i]` is also visible (but blocks everyone behind it).

**Q: Why are popped people visible to person i?**
A: Each popped person is shorter than `heights[i]`. Since they were on the decreasing stack, no one between them and `i` is taller than them — so `i` can see over all intervening people to reach them.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | Valid Parens, Min Stack, Daily Temperatures |
| L4 | Largest Rectangle, Basic Calc, Sliding Window Max |
| L5+ | Max Freq Stack, Number of Visible People, Basic Calc III (recursive) |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [Maximum Frequency Stack Design](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
