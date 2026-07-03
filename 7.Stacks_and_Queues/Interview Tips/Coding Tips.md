# Coding Tips — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Always Use `std::deque`, Not `std::stack<>` Alone

```cpp
#include <bits/stdc++.h>
using namespace std;

// Simple LIFO — stack<int> adapter (limited to top access only)
stack<int> stk;
stk.push(x);    // push to top
stk.pop();      // remove from top
stk.top();      // view top

// FLEXIBLE — deque<int> supports both ends
deque<int> dq;
dq.push_front(x);  // push to front (top)
dq.pop_front();    // remove from front (top)
dq.front();        // view front (top)
```

`std::stack<T>` is a simple adapter with only top access. `std::deque<T>` is more flexible and supports bidirectional access, making it the preferred choice when you need explicit front/back control.

---

## 2. Monotonic Stack — Store Indices, Not Values

```cpp
#include <bits/stdc++.h>
using namespace std;

// BAD — can't compute distances or write back to result array
stack<int> stk;
stk.push(nums[i]);   // pushing value

// GOOD
stack<int> stk;
stk.push(i);         // pushing index
// Then: nums[stk.top()] gives value, stk.top() gives position
```

Reason: Distance-based answers (Daily Temperatures: `i - prevIdx`) and writing answers back to result arrays both require the index.

---

## 3. Monotonic Deque — Use `push_back/pop_back` and `push_front/pop_front`

```cpp
#include <bits/stdc++.h>
using namespace std;

// For sliding window maximum (decreasing deque):
deque<int> dq;
// Add to back (maintain decreasing order from front to back)
while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
dq.push_back(i);
// Get max from front
int maxVal = nums[dq.front()];
// Expire old window from front
if (dq.front() <= i - k) dq.pop_front();
```

Never mix `push_front/pop_front` (stack style) with `push_back/pop_front` (queue style) haphazardly. Be explicit about which end you're accessing.

---

## 4. Histogram Width Formula

```cpp
#include <bits/stdc++.h>
using namespace std;

while (!stk.empty() && heights[stk.top()] > h) {
    int height = heights[stk.top()]; stk.pop();
    // LEFT boundary is exclusive: stk.top() + 1
    // RIGHT boundary is exclusive: i (current index)
    int width = stk.empty() ? i : i - stk.top() - 1;
    maxArea = max(maxArea, height * width);
}
```

Mnemonic: "Width is the gap between the two shorter walls." `i` is the right wall, `stk.top()` is the left wall. Width = `right - left - 1` (excluding both walls).

---

## 5. Basic Calculator — The `sign` Variable Pattern

```cpp
int num = 0;
char sign = '+';   // sign of the UPCOMING number, not current

// When we see an operator or reach end of string:
// 1. Apply 'sign' to 'num' (process the number we just built)
// 2. Update 'sign' = current operator
// 3. Reset num = 0
```

This lookahead trick works because in `"3+5*2"`, when we see `+` we're done reading `3`. The sign `+` applies to the next operand `5` (we don't know yet).

---

## 6. Decode String — Multi-Digit Counts

```cpp
if (isdigit(c)) {
    k = k * 10 + (c - '0');  // accumulate: "12[" → k = 1*10+2 = 12
}
```

Easy to forget when writing the solution quickly. `"12[a]"` → `"aaaaaaaaaaaa"` (12 a's). Without the `k * 10`, you'd get `k = 2`.

---

## 7. Queue via Two Stacks — Transfer Only When Needed

```cpp
void transfer() {
    if (outStack.empty()) {    // only transfer when outStack is EMPTY
        while (!inStack.empty()) {
            outStack.push(inStack.top());
            inStack.pop();
        }
    }
}
```

Partial transfer (when outStack has some items) would break FIFO ordering. Transfer only when `outStack` is completely drained.

---

## 8. Max Frequency Stack — Default-Init Pattern

```cpp
group[f].push(val);  // operator[] default-initializes if key absent
```

Equivalent to:
```cpp
if (!group.count(f)) group[f] = stack<int>();
group[f].push(val);
```

The `group[f].push(val)` version is idiomatic C++ — `operator[]` default-initializes missing keys automatically, avoiding the double-lookup.

---

## 9. Circular Queue — `size` Variable vs One Slot Wasted

```cpp
// Option A (recommended): Track size separately
if (isFull()) return false;  // size == capacity
tail = (tail + 1) % capacity;
data[tail] = value;
size++;

// Option B: One slot wasted (capacity+1 slots for N-element queue)
// isFull = (tail + 1) % capacity == head
// This wastes one slot but avoids a separate size counter
```

Option A with `size` is cleaner and avoids the off-by-one confusion inherent in Option B.

---

## 10. Sentinel in Histogram

```cpp
for (int i = 0; i <= n; i++) {          // i goes to n (inclusive)
    int h = (i == n) ? 0 : heights[i];   // sentinel height = 0
    // ...
}
```

The sentinel forces all remaining stack entries to flush after the loop. Forgetting this is the most common bug in histogram problems — elements that are never smaller than their right neighbor stay in the stack forever.

---

## Quick Reference: Method Names

| Operation | stack\<int\> | queue\<int\> | deque\<int\> (explicit) |
|-----------|-------------|-------------|------------------------|
| Add top/front | `push(x)` | `push(x)` | `push_front(x)` |
| Add bottom/back | — | — | `push_back(x)` |
| Remove top/front | `pop()` | `pop()` | `pop_front()` |
| Remove back | — | — | `pop_back()` |
| View top/front | `top()` | `front()` | `front()` |
| View back | — | — | `back()` |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)
