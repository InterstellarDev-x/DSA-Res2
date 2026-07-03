# Common Mistakes — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Mistake 1: Using `std::stack<>` Instead of `std::deque`

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD
stack<int> stk;
stk.push(1);

// GOOD
deque<int> stk;
stk.push_front(1);
```

`std::stack<T>` is a container adaptor that only exposes one end and does not support random access via `operator[]`. `std::deque` provides direct access to both ends and does not expose unintended access patterns. `std::deque` is the correct choice when you need a double-ended structure.

---

## Mistake 2: Pushing Values Instead of Indices in Monotonic Stack

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — for Daily Temperatures
while (!stk.empty() && nums[stk.top()] < nums[i]) {
    int prevIdx = stk.top(); stk.pop();
    result[prevIdx] = i - prevIdx;  // ERROR: stk.top() returns the VALUE, can't do i - value
}

// GOOD
while (!stk.empty() && temperatures[stk.top()] < temperatures[i]) {
    int prevIdx = stk.top(); stk.pop();
    result[prevIdx] = i - prevIdx;  // prevIdx is an INDEX, subtraction gives days
}
stk.push(i);  // push index, not temperatures[i]
```

If you push values, you can't compute distances or write back to the result array by position.

---

## Mistake 3: Histogram Width Formula Off-By-One

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — wrong width
int width = i - stk.top();           // off by 1 (includes left wall)

// GOOD
int width = stk.empty() ? i : i - stk.top() - 1;

// Explanation:
// - Right boundary (exclusive): i (heights[i] is shorter, so rectangle ends before i)
// - Left boundary (exclusive): stk.top() (this is the next-shorter bar to the left)
// - Width: (i-1) - (stk.top()+1) + 1 = i - stk.top() - 1
```

The `-1` is the most common off-by-one in this problem. Verify with a 1-element array: `heights=[5]`, at sentinel `i=1`, stk.pop()=0, stk is empty, width=1 ✓.

---

## Mistake 4: Forgetting the Histogram Sentinel

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — bars that are never smaller than their right neighbor stay in stack
for (int i = 0; i < n; i++) {
    while (!stk.empty() && heights[stk.top()] > heights[i]) {
        // compute area
    }
    stk.push(i);
}
// Stack still has entries! Never flushed for bars like [3, 5, 7]

// GOOD
for (int i = 0; i <= n; i++) {         // <= n, not < n
    int h = (i == n) ? 0 : heights[i]; // height 0 flushes all remaining
    // ...
}
```

For input `[3, 5, 7]` (monotonically increasing), no pops occur in the main loop. The sentinel forces all three to be popped and their areas computed.

---

## Mistake 5: Wrong Transfer Order in Queue via Two Stacks

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — partial transfer breaks FIFO
void transfer() {
    while (!inStack.empty()) { outStack.push(inStack.top()); inStack.pop(); }  // always transfers
}

// GOOD — only transfer when outStack is completely empty
void transfer() {
    if (outStack.empty())
        while (!inStack.empty()) { outStack.push(inStack.top()); inStack.pop(); }
}
```

If `outStack` has `[3,2,1]` (1 on top = dequeue next) and you transfer `inStack=[6,5,4]` (4 on top), you'd get `outStack=[4,5,6,3,2,1]` with 4 as the next to dequeue — wrong order.

---

## Mistake 6: Circular Queue — Forgetting `size` or Using Wrong Condition

```cpp
// BAD — ambiguous: head==tail could mean empty OR full
bool isEmpty() { return head == tail; }
bool isFull()  { return head == tail; }  // same condition!

// GOOD — track size separately
int size = 0;
bool isEmpty() { return size == 0; }
bool isFull()  { return size == capacity; }
```

Without a size counter (or a `isFull` boolean flag), you can't distinguish an empty circular buffer from a full one when `head == tail`.

---

## Mistake 7: Basic Calculator — Forgetting the Last Number

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — misses the last token (no operator after it)
for (int i = 0; i < (int)s.length(); i++) {
    char c = s[i];
    if (isdigit(c)) num = num * 10 + (c - '0');
    if (!isdigit(c) && c != ' ') {  // WRONG: only processes on operator
        // apply sign to num
    }
}
// Last number never gets processed!

// GOOD — process on operator OR at end of string
if ((!isdigit(c) && c != ' ') || i == (int)s.length() - 1) {
    // apply sign to num
}
```

The input `"3+5"` ends with a digit. Without the `i == (int)s.length() - 1` condition, `5` is never pushed to the stack.

---

## Mistake 8: Decode String — Not Resetting `k` After `[`

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD
} else if (c == '[') {
    countStack.push(k);
    strStack.push(current);
    current = "";
    // MISSING: k = 0
}

// GOOD
} else if (c == '[') {
    countStack.push(k);
    strStack.push(current);
    k = 0;                    // reset for the next number
    current = "";
}
```

Without resetting `k`, nested brackets like `"2[3[a]]"` would use `k=3` again for the outer bracket instead of finding `k=2`.

---

## Mistake 9: Max Frequency Stack — `maxFreq` Decrement Condition

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — always decrement maxFreq on pop
int pop() {
    auto& stk = group[maxFreq];
    int val = stk.top(); stk.pop();
    freq[val]--;
    maxFreq--;   // WRONG: only decrement if group[maxFreq] is now empty
    return val;
}

// GOOD
int pop() {
    auto& stk = group[maxFreq];
    int val = stk.top(); stk.pop();
    freq[val]--;
    if (stk.empty()) {
        group.erase(maxFreq);
        maxFreq--;   // only decrement when bucket is empty
    }
    return val;
}
```

After popping from `group[3]`, if `group[3]` still has elements, `maxFreq` should stay 3. Decrementing unconditionally would skip valid elements.

---

## Mistake 10: 132 Pattern — Confusing Which Pointer Is Which

The pattern requires three pointers: `nums[i] < nums[k] < nums[j]` where `i < j < k`.
- `stk` = candidates for `nums[j]` (the "3" in 132 — the largest)
- `third` = best candidate for `nums[k]` (the "2" in 132 — middle value)
- `nums[i]` = current element being checked as the smallest ("1")

```cpp
// BAD — checking wrong condition
if (nums[i] > third) return true;   // WRONG: we want nums[i] < third

// GOOD
if (nums[i] < third) return true;   // nums[i] is "1", must be LESS than "2" (third)
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
