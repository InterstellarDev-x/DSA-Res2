# Queue & Deque

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 2 of 5**
> **Problems:** Implement Queue using Stacks · Implement Stack using Queues · Sliding Window Maximum · Number of Recent Calls

---

## Core Concepts

### Queue — FIFO
```java
// ArrayDeque is the standard Queue implementation in Java
Queue<Integer> queue = new ArrayDeque<>();
queue.offer(x);     // enqueue (returns false if fails — prefer over add())
queue.poll();       // dequeue (returns null if empty — prefer over remove())
queue.peek();       // view front without removing
queue.isEmpty();
queue.size();
```

### Deque — Double-Ended Queue
```java
// Both stack and queue operations available
Deque<Integer> deque = new ArrayDeque<>();
// Front operations
deque.offerFirst(x);   deque.pollFirst();   deque.peekFirst();
// Back operations
deque.offerLast(x);    deque.pollLast();    deque.peekLast();
// Stack-style (front = top)
deque.push(x);         deque.pop();         deque.peek();
```

**Key distinction:** Use `Deque` (not `Stack`) for LIFO. `ArrayDeque` is O(1) amortized for all operations. `LinkedList` is O(1) but has more overhead per node.

---

## Problem 1: Implement Queue using Stacks — LC 232

**Constraint:** Use only push, pop, peek, empty operations of standard stack.

```java
class MyQueue {
    private Deque<Integer> inStack  = new ArrayDeque<>();  // for push
    private Deque<Integer> outStack = new ArrayDeque<>();  // for pop/peek

    public void push(int x) {
        inStack.push(x);
    }

    public int pop() {
        move();
        return outStack.pop();
    }

    public int peek() {
        move();
        return outStack.peek();
    }

    public boolean empty() {
        return inStack.isEmpty() && outStack.isEmpty();
    }

    private void move() {
        if (outStack.isEmpty()) {
            while (!inStack.isEmpty()) outStack.push(inStack.pop());
        }
    }
}
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

```java
class MyStack {
    private Queue<Integer> queue = new ArrayDeque<>();

    public void push(int x) {
        queue.offer(x);
        // Rotate: move all elements before x to the back
        for (int i = 0; i < queue.size() - 1; i++) {
            queue.offer(queue.poll());
        }
    }

    public int pop()  { return queue.poll(); }
    public int top()  { return queue.peek(); }
    public boolean empty() { return queue.isEmpty(); }
}
```

**Why rotate on push?** After adding `x` at the tail, we rotate all previous elements behind it, making `x` the new front. Front always = top of stack.

**Trace:** push(1): queue=[1]; push(2): after offer [1,2], rotate 1→back: [2,1]; push(3): after offer [2,1,3], rotate 2,1→back: [3,2,1]

---

## Problem 3: Sliding Window Maximum — LC 239

**Input:** `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`
**Output:** `[3,3,5,5,6,7]` (max of each window of size k)

**Naive:** O(nk) — recompute max for each window.
**Optimal:** Monotonic Deque — O(n).

```java
public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    int[] result = new int[n - k + 1];
    // Deque stores INDICES; front = index of current window max
    Deque<Integer> deque = new ArrayDeque<>();

    for (int i = 0; i < n; i++) {
        // 1. Remove indices outside current window
        while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
            deque.pollFirst();
        }
        // 2. Maintain decreasing order: remove smaller elements from back
        while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
            deque.pollLast();
        }
        deque.offerLast(i);

        // 3. Window is full starting from i == k-1
        if (i >= k - 1) {
            result[i - k + 1] = nums[deque.peekFirst()];
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

```java
class RecentCounter {
    private Queue<Integer> queue = new ArrayDeque<>();

    public int ping(int t) {
        queue.offer(t);
        while (queue.peek() < t - 3000) {
            queue.poll();
        }
        return queue.size();
    }
}
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
```java
for (int i = 0; i < n; i++) {
    // Expire out-of-window indices
    while (!dq.isEmpty() && dq.peekFirst() < i - k + 1) dq.pollFirst();
    // Maintain monotonicity (decreasing for max)
    while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast();
    dq.offerLast(i);
    if (i >= k - 1) result[i - k + 1] = nums[dq.peekFirst()];
}
```

---

## Queue vs Deque — Decision Table

| Need | Use |
|------|-----|
| Pure FIFO | `Queue<T>` backed by `ArrayDeque` |
| LIFO (stack) | `Deque<T>` with `push/pop/peek` |
| Both ends (sliding window) | `Deque<T>` with `offerFirst/Last`, `pollFirst/Last` |
| Priority FIFO | `PriorityQueue<T>` |
| Thread-safe queue | `LinkedBlockingQueue<T>` |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Monotonic Stack](./Monotonic%20Stack.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Stack vs Queue Usage](../Interview%20Tips/Stack%20vs%20Queue%20Usage.md)

> **Last Updated:** 2026-06-26
