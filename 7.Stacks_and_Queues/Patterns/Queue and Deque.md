# Queue & Deque

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 2 of 5**
> **Problems:** Implement Queue using Stacks · Implement Stack using Queues · Sliding Window Maximum · Number of Recent Calls

---

## Core Concepts

### Queue — FIFO
```cpp
#include <bits/stdc++.h>
using namespace std;
// std::queue is the standard Queue implementation in C++
queue<int> q;
q.push(x);      // enqueue
q.front();      // view front without removing
q.pop();        // dequeue (returns void — read front() first)
q.empty();
q.size();
```

### Deque — Double-Ended Queue
```cpp
#include <bits/stdc++.h>
using namespace std;
// Both stack and queue operations available
deque<int> dq;
// Front operations
dq.push_front(x);   dq.pop_front();   dq.front();
// Back operations
dq.push_back(x);    dq.pop_back();    dq.back();
// Stack-style (front = top)
dq.push_front(x);   dq.pop_front();   dq.front();
```

**Key distinction:** Use `deque<T>` (not `stack<T>`) when you need both-end access. `deque` is O(1) amortized for all operations. `list<T>` is O(1) but has more overhead per node.

---

## Problem 1: Implement Queue using Stacks — LC 232

**Constraint:** Use only push, pop, peek, empty operations of standard stack.

```cpp
#include <bits/stdc++.h>
using namespace std;
class MyQueue {
    stack<int> inStack;   // for push
    stack<int> outStack;  // for pop/peek

public:
    void push(int x) {
        inStack.push(x);
    }

    int pop() {
        move();
        int val = outStack.top();
        outStack.pop();
        return val;
    }

    int peek() {
        move();
        return outStack.top();
    }

    bool empty() {
        return inStack.empty() && outStack.empty();
    }

private:
    void move() {
        if (outStack.empty()) {
            while (!inStack.empty()) {
                outStack.push(inStack.top());
                inStack.pop();
            }
        }
    }
};
```

**Amortized O(1) — the key insight:**
Each element crosses from inStack → outStack exactly **once** total. Even though a single `pop()` might cost O(n) if a transfer occurs, across N operations the total work is O(N). This is amortized O(1) per operation.

**Trace:** push(1), push(2), push(3)
- inStack: [3,2,1] (3 on top)
- First pop() triggers move: outStack: [1,2,3] (1 on top) → returns 1
- Second pop(): outStack has [2,3] → returns 2 (no transfer needed)

---

## Problem 2: Implement Stack using Queues — LC 225

### Approach: One Queue — Push is O(n), Pop/Peek is O(1)

**Invariant:** Queue always stores elements with the most recently pushed at front.

```cpp
#include <bits/stdc++.h>
using namespace std;
class MyStack {
    queue<int> q;

public:
    void push(int x) {
        q.push(x);
        // Rotate: move all elements before x to the back
        for (int i = 0; i < (int)q.size() - 1; i++) {
            q.push(q.front());
            q.pop();
        }
    }

    int pop() {
        int val = q.front();
        q.pop();
        return val;
    }
    int top()  { return q.front(); }
    bool empty() { return q.empty(); }
};
```

**Why rotate on push?** After adding `x` at the tail, we rotate all previous elements behind it, making `x` the new front. Front always = top of stack.

**Trace:** push(1): q=[1]; push(2): after push [1,2], rotate 1→back: [2,1]; push(3): after push [2,1,3], rotate 2,1→back: [3,2,1]

---

## Problem 3: Sliding Window Maximum — LC 239

**Input:** `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`
**Output:** `[3,3,5,5,6,7]` (max of each window of size k)

**Naive:** O(nk) — recompute max for each window.
**Optimal:** Monotonic Deque — O(n).

```cpp
#include <bits/stdc++.h>
using namespace std;
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    int n = nums.size();
    vector<int> result(n - k + 1);
    // Deque stores INDICES; front = index of current window max
    deque<int> dq;

    for (int i = 0; i < n; i++) {
        // 1. Remove indices outside current window
        while (!dq.empty() && dq.front() <= i - k) {
            dq.pop_front();
        }
        // 2. Maintain decreasing order: remove smaller elements from back
        while (!dq.empty() && nums[dq.back()] <= nums[i]) {
            dq.pop_back();
        }
        dq.push_back(i);

        // 3. Window is full starting from i == k-1
        if (i >= k - 1) {
            result[i - k + 1] = nums[dq.front()];
        }
    }
    return result;
}
```

**Why store indices (not values)?** We need to check if the front index has expired (left the window). Values alone don't tell us position.

**Invariant:** Deque stores indices in decreasing order of their values. The front index always holds the maximum for the current window.

**Trace for [1,3,-1,-3,5,3,6,7], k=3:**
```
i=0: deque=[0(val=1)]
i=1: nums[0]=1 <= nums[1]=3 → remove 0. deque=[1(val=3)]
i=2: nums[2]=-1 < nums[1]=3 → keep. deque=[1,2]. result[0]=nums[1]=3
i=3: nums[3]=-3 < nums[2]=-1 → keep. deque=[1,2,3]. result[1]=nums[1]=3
i=4: remove expired 1 (1<=4-3=1 ✅). remove 3(-3),2(-1) < 5. deque=[4]. result[2]=nums[4]=5
i=5: nums[5]=3 < nums[4]=5 → keep. deque=[4,5]. result[3]=nums[4]=5
i=6: remove 5(3),4(5) <= 6. deque=[6]. result[4]=nums[6]=6
i=7: nums[7]=7 > nums[6]=6 → remove. deque=[7]. result[5]=nums[7]=7
```

**Complexity:** O(n) time — each index enters and leaves deque at most once. O(k) space for deque.

---

## Problem 4: Number of Recent Calls — LC 933

**Design:** `RecentCounter` — `ping(t)` adds call at time `t`, returns number of calls in `[t-3000, t]`.

```cpp
#include <bits/stdc++.h>
using namespace std;
class RecentCounter {
    queue<int> q;

public:
    int ping(int t) {
        q.push(t);
        while (q.front() < t - 3000) {
            q.pop();
        }
        return q.size();
    }
};
```

**Why works:** `t` is strictly increasing (guaranteed). Queue maintains a sliding window of calls within the last 3000 ms. Each call is added at most once and removed at most once — O(1) amortized per `ping`.

---

## Monotonic Deque Pattern Summary

A **monotonic deque** maintains a deque where values are always in a consistent order (increasing or decreasing). It enables O(1) range max/min queries as a window slides.

| Variant | Deque Order | Gives You |
|---------|-------------|-----------|
| Decreasing deque | Front = max | Sliding window maximum |
| Increasing deque | Front = min | Sliding window minimum |

**Template:**
```cpp
#include <bits/stdc++.h>
using namespace std;
for (int i = 0; i < n; i++) {
    // Expire out-of-window indices
    while (!dq.empty() && dq.front() < i - k + 1) dq.pop_front();
    // Maintain monotonicity (decreasing for max)
    while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back();
    dq.push_back(i);
    if (i >= k - 1) result[i - k + 1] = nums[dq.front()];
}
```

---

## Queue vs Deque — Decision Table

| Need | Use |
|------|-----|
| Pure FIFO | `queue<T>` |
| LIFO (stack) | `stack<T>` or `deque<T>` with `push_front`/`pop_front` |
| Both ends (sliding window) | `deque<T>` with `push_front`/`push_back`, `pop_front`/`pop_back` |
| Priority FIFO | `priority_queue<T>` |
| Thread-safe queue | No direct STL equivalent — use mutex + `queue<T>` or a concurrent library |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Monotonic Stack](./Monotonic%20Stack.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Stack vs Queue Usage](../Interview%20Tips/Stack%20vs%20Queue%20Usage.md)

> **Last Updated:** 2026-06-26
