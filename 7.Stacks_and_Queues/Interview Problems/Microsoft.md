# Microsoft — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Microsoft
> [← Microsoft OA](../OA-Qns/Microsoft.md)

---

## Problem 1: Implement Queue using Stacks — Design Deep Dive

**LC 232** · Easy · Amortized O(1)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MyQueue {
    stack<int> inStack, outStack;

    void transfer() {
        if (outStack.empty())
            while (!inStack.empty()) {
                outStack.push(inStack.top());
                inStack.pop();
            }
    }
public:
    void push(int x) { inStack.push(x); }

    int pop() {
        transfer();
        int val = outStack.top();
        outStack.pop();
        return val;
    }

    int peek() {
        transfer();
        return outStack.top();
    }

    bool empty() { return inStack.empty() && outStack.empty(); }
};
```

**Q: What is the amortized time complexity?**
A: O(1) amortized. Each element moves from `inStack` to `outStack` exactly once. So over N operations, total work = O(N). Worst case per single `pop` is O(N) (when transfer occurs), but that can only happen after N pushes, spreading the O(N) cost across N previous operations.

**Q: What if we need peek() and pop() to always be O(1) worst case?**
A: Not possible with only stacks. You'd need to transfer on every push instead — O(N) push, O(1) pop/peek. There's a fundamental tradeoff: you can make either push or pop O(1), but not both worst-case.

**Q: Microsoft follow-up — extend to thread-safe queue.**

```cpp
#include <bits/stdc++.h>
#include <mutex>
using namespace std;

class ThreadSafeQueue {
    stack<int> inStack, outStack;
    mutex mtx;

    void transfer() {
        if (outStack.empty())
            while (!inStack.empty()) {
                outStack.push(inStack.top());
                inStack.pop();
            }
    }
public:
    void push(int x) {
        lock_guard<mutex> lock(mtx);
        inStack.push(x);
    }

    int pop() {
        lock_guard<mutex> lock(mtx);
        transfer();
        int val = outStack.top();
        outStack.pop();
        return val;
    }
};
```

---

## Problem 2: Basic Calculator II — Microsoft Style

**LC 227** · Medium

Microsoft often asks this in a "live coding" format with follow-ups about edge cases.

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
            else if (sign == '*') { int top = stk.top(); stk.pop(); stk.push(top * num); }
            else if (sign == '/') { int top = stk.top(); stk.pop(); stk.push(top / num); }
            sign = c; num = 0;
        }
    }
    int result = 0;
    while (!stk.empty()) { result += stk.top(); stk.pop(); }
    return result;
}
```

**Microsoft Edge Case checklist:**
- Spaces between tokens: `"3 + 2"` — handled by `c != ' '` skip
- Multi-digit numbers: `"14+2"` — handled by `num = num * 10 + digit`
- Negative results: `"1-2"` — handled by pushing `-num` for `-` sign
- Division truncation: C++ integer division truncates toward zero by default ✓
- Single number: `"5"` — final `i == s.length()-1` condition handles last number

---

## Problem 3: Trapping Rain Water — Microsoft Interview Discussion

**LC 42** · Hard

Microsoft interviewers focus on code quality and approach justification.

**Preferred approach for Microsoft interview:**

```cpp
#include <bits/stdc++.h>
using namespace std;

int trap(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1;
    int maxLeft = 0, maxRight = 0, water = 0;
    while (left < right) {
        if (height[left] <= height[right]) {
            if (height[left] >= maxLeft) maxLeft = height[left];
            else water += maxLeft - height[left];
            left++;
        } else {
            if (height[right] >= maxRight) maxRight = height[right];
            else water += maxRight - height[right];
            right--;
        }
    }
    return water;
}
```

**How to walk through this at Microsoft:**
1. "The key insight is that water at position `x` = `min(maxLeft, maxRight) - height[x]`."
2. "If `height[left] ≤ height[right]`, the right side is guaranteed to be the larger boundary, so we can compute water for the left side using just `maxLeft`."
3. "This gives us O(1) space, O(n) time — optimal."

**Q: What if the array contains negative heights?**
A: Problem constraints guarantee non-negative heights. But if it could, negative heights would still work — water calculation uses subtraction, which would be negative (meaning no water trapped there).

---

## Microsoft Interview Style Notes

- Microsoft prioritizes code readability and correctness over cleverness.
- Walk through your approach verbally before coding — they want to see your thought process.
- Common follow-up after any stack problem: "How would you make this thread-safe?"
- For design problems (Min Stack, Circular Queue), mention invariants explicitly: "My invariant is that `outStack` always has elements in FIFO order."

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Queue & Deque Pattern](../Patterns/Queue%20and%20Deque.md)
- [Stack for Expressions](../Patterns/Stack%20for%20Expressions.md)

> **Last Updated:** 2026-06-26
