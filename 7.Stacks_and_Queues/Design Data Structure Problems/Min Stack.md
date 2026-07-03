# Min Stack & Design Circular Queue

> **Topic:** [Stacks & Queues](../README.md) · **Design 1 of 2**
> **Problems:** Min Stack (LC 155) · Design Circular Queue (LC 622)

---

## Min Stack — LC 155

**Requirements:** Design a stack that supports `push`, `pop`, `top`, and `getMin` in O(1) time.

The challenge: standard stacks don't track minimums. After a `pop`, the minimum might change — so we need to track minimums through the history.

---

### Approach 1: Auxiliary Min Stack — O(1) all operations, O(n) space

Maintain a second stack (`minStack`) that tracks the running minimum at each level.

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinStack {
    stack<int> stk;
    stack<int> minStk;

public:
    void push(int val) {
        stk.push(val);
        // Push to minStk if it's the new minimum (or minStk is empty)
        int mn = minStk.empty() ? val : min(val, minStk.top());
        minStk.push(mn);
    }

    void pop() {
        stk.pop();
        minStk.pop();    // always pop both together
    }

    int top()    { return stk.top(); }
    int getMin() { return minStk.top(); }
};
```

**Why push min every time (not only when it changes)?** Because `pop()` must always pop both stacks together. If we only pushed to `minStack` when value decreases, the two stacks would have different sizes and fall out of sync.

**Trace:**
```
push(5): stack=[5], minStack=[5]
push(3): stack=[3,5], minStack=[3,3]
push(7): stack=[7,3,5], minStack=[3,3,3] — min doesn't change but we sync
push(2): stack=[2,7,3,5], minStack=[2,2,3,3]
getMin()=2 ✓
pop(): stack=[7,3,5], minStack=[2,3,3]
getMin()=2 ✓
pop(): stack=[3,5], minStack=[3,3]
getMin()=3 ✓
```

---

### Approach 2: Single Stack with Pairs — O(1) all operations, O(n) space

Store `(value, minAtThisPoint)` as a pair in each stack entry. Uses one structure instead of two.

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinStack {
    stack<pair<int,int>> stk;  // {val, minSoFar}

public:
    void push(int val) {
        int mn = stk.empty() ? val : min(val, stk.top().second);
        stk.push({val, mn});
    }

    void pop()    { stk.pop(); }
    int top()     { return stk.top().first; }
    int getMin()  { return stk.top().second; }
};
```

**Trade-off:** Slightly more memory per entry (two ints), but one data structure. The pair approach is cleaner in code.

---

### Approach 3: Encoding trick — O(1) time, O(n) space (avoids extra structure)

Store the difference `(val - currentMin)` in the stack. When difference is negative, we know `val` was a new minimum.

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinStack {
    stack<long> stk;
    long mn = LONG_MAX;

public:
    void push(int val) {
        if (stk.empty()) {
            stk.push(0L);
            mn = val;
        } else {
            stk.push((long) val - mn);
            if (val < mn) mn = val;
        }
    }

    void pop() {
        long diff = stk.top(); stk.pop();
        if (diff < 0) mn = mn - diff;   // restore previous min
    }

    int top() {
        long diff = stk.top();
        return diff < 0 ? (int) mn : (int)(mn + diff);
    }

    int getMin() { return (int) mn; }
};
```

**Why `long`?** `val - min` can overflow `int` if `val` is large positive and `min` is large negative (or vice versa).

**Restore logic on pop:** If stored `diff < 0`, it means `min` was updated to `val = min + diff` when this was pushed. The previous min = `min - diff`.

---

## Design Circular Queue — LC 622

**Requirements:** `enQueue`, `deQueue`, `Front`, `Rear`, `isEmpty`, `isFull` — all O(1).

### Array-based Circular Buffer

```cpp
#include <bits/stdc++.h>
using namespace std;

class MyCircularQueue {
    vector<int> data;
    int head, tail, sz, capacity;

public:
    MyCircularQueue(int k) : data(k), head(0), tail(-1), sz(0), capacity(k) {}

    bool enQueue(int value) {
        if (isFull()) return false;
        tail = (tail + 1) % capacity;
        data[tail] = value;
        sz++;
        return true;
    }

    bool deQueue() {
        if (isEmpty()) return false;
        head = (head + 1) % capacity;
        sz--;
        return true;
    }

    int Front() { return isEmpty() ? -1 : data[head]; }
    int Rear()  { return isEmpty() ? -1 : data[tail]; }
    bool isEmpty() { return sz == 0; }
    bool isFull()  { return sz == capacity; }
};
```

**Why track `size` separately?** Avoids the "one slot wasted" trick (reserving one empty slot to distinguish full from empty based on head/tail positions).

**Modulo wrap-around:** `(index + 1) % capacity` is the key operation that makes the array circular. When `tail` reaches the end, it wraps to 0.

**Trace for capacity=3:**
```
enQueue(1): tail=0, data=[1,_,_], size=1
enQueue(2): tail=1, data=[1,2,_], size=2
enQueue(3): tail=2, data=[1,2,3], size=3  ← full
deQueue(): head=1, data=[_,2,3], size=2
enQueue(4): tail=(2+1)%3=0, data=[4,2,3], size=3  ← wraps!
Front()=data[1]=2, Rear()=data[0]=4
```

---

### Alternative: Size-based tracking vs head==tail

**Ambiguity problem:** If `head == tail`, it could mean empty OR full. Solutions:
1. Track `size` (used above — cleanest)
2. Use a `boolean full` flag
3. Reserve one slot (capacity+1 allocated, full when `(tail+1)%cap == head`)

Option 1 is cleanest for interviews.

---

## Interview Discussion Points

### Min Stack
**Q: Can you do O(1) getMin with O(1) push/pop using less space?**
A: The pair/two-stack approaches are O(n) by necessity — we must remember the minimum at each stack level in case pops reveal a new minimum. The encoding trick reduces constant factor but is still O(n). Fundamental lower bound: O(n) space for a stack supporting getMin in O(1).

**Q: Thread safety?**
A: Wrap methods with `std::mutex` and `std::lock_guard` (or `std::unique_lock`). The two-stack approach has a window where stack has been popped but minStack hasn't — this is a critical section.

### Circular Queue
**Q: Why circular instead of `std::list`?**
A: Array has O(1) index access and better cache locality. Linked list has pointer overhead per node. For a fixed-size bounded queue, circular array is standard.

**Q: How to implement a circular DEQUE?**
A: Same structure but add `enqueueFront` and `dequeueRear`:
- `enqueueFront`: `head = (head - 1 + capacity) % capacity`
- `dequeueRear`: `tail = (tail - 1 + capacity) % capacity`

---

## Related Files

- [Maximum Frequency Stack](./Maximum%20Frequency%20Stack.md)
- [Stack Basics](../Patterns/Stack%20Basics.md)
- [Queue & Deque](../Patterns/Queue%20and%20Deque.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
