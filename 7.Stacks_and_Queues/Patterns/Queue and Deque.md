# Queue & Deque

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 2 of 5**
> **Problems:** Implement Queue using Stacks · Implement Stack using Queues · Sliding Window Maximum · Number of Recent Calls

---

## Core Concepts

### Queue — FIFO
```rust
use std::collections::VecDeque;
// VecDeque is the standard Queue implementation in Rust
let mut q: VecDeque<i32> = VecDeque::new();
q.push_back(x);      // enqueue
q.front();           // view front without removing (returns Option<&i32>)
q.pop_front();       // dequeue returns Option<i32> — unwrap or match
q.is_empty();
q.len();
```

### Deque — Double-Ended Queue
```rust
use std::collections::VecDeque;
// Both stack and queue operations available
let mut dq: VecDeque<i32> = VecDeque::new();
// Front operations
dq.push_front(x);   dq.pop_front();   dq.front();
// Back operations
dq.push_back(x);    dq.pop_back();    dq.back();
// Stack-style (front = top)
dq.push_front(x);   dq.pop_front();   dq.front();
```

**Key distinction:** Use `VecDeque<T>` (not `Vec<T>`) when you need both-end access. `VecDeque` is O(1) amortized for all operations. `LinkedList<T>` is O(1) but has more overhead per node.

---

## Problem 1: Implement Queue using Stacks — LC 232

**Constraint:** Use only push, pop, peek, empty operations of standard stack.

```rust
struct MyQueue {
    in_stack: Vec<i32>,   // for push
    out_stack: Vec<i32>,  // for pop/peek
}

impl MyQueue {
    fn new() -> Self {
        MyQueue {
            in_stack: Vec::new(),
            out_stack: Vec::new(),
        }
    }

    fn push(&mut self, x: i32) {
        self.in_stack.push(x);
    }

    fn pop(&mut self) -> i32 {
        self.transfer();
        self.out_stack.pop().unwrap()
    }

    fn peek(&mut self) -> i32 {
        self.transfer();
        *self.out_stack.last().unwrap()
    }

    fn empty(&self) -> bool {
        self.in_stack.is_empty() && self.out_stack.is_empty()
    }

    fn transfer(&mut self) {
        if self.out_stack.is_empty() {
            while let Some(val) = self.in_stack.pop() {
                self.out_stack.push(val);
            }
        }
    }
}
```

**Amortized O(1) — the key insight:**
Each element crosses from in_stack → out_stack exactly **once** total. Even though a single `pop()` might cost O(n) if a transfer occurs, across N operations the total work is O(N). This is amortized O(1) per operation.

**Trace:** push(1), push(2), push(3)
- in_stack: [3,2,1] (3 on top)
- First pop() triggers transfer: out_stack: [1,2,3] (1 on top) → returns 1
- Second pop(): out_stack has [2,3] → returns 2 (no transfer needed)

---

## Problem 2: Implement Stack using Queues — LC 225

### Approach: One Queue — Push is O(n), Pop/Peek is O(1)

**Invariant:** Queue always stores elements with the most recently pushed at front.

```rust
use std::collections::VecDeque;

struct MyStack {
    q: VecDeque<i32>,
}

impl MyStack {
    fn new() -> Self {
        MyStack { q: VecDeque::new() }
    }

    fn push(&mut self, x: i32) {
        self.q.push_back(x);
        // Rotate: move all elements before x to the back
        let len = self.q.len();
        for _ in 0..len - 1 {
            let front = self.q.pop_front().unwrap();
            self.q.push_back(front);
        }
    }

    fn pop(&mut self) -> i32 {
        self.q.pop_front().unwrap()
    }

    fn top(&self) -> i32 {
        *self.q.front().unwrap()
    }

    fn empty(&self) -> bool {
        self.q.is_empty()
    }
}
```

**Why rotate on push?** After adding `x` at the tail, we rotate all previous elements behind it, making `x` the new front. Front always = top of stack.

**Trace:** push(1): q=[1]; push(2): after push [1,2], rotate 1→back: [2,1]; push(3): after push [2,1,3], rotate 2,1→back: [3,2,1]

---

## Problem 3: Sliding Window Maximum — LC 239

**Input:** `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`
**Output:** `[3,3,5,5,6,7]` (max of each window of size k)

**Naive:** O(nk) — recompute max for each window.
**Optimal:** Monotonic Deque — O(n).

```rust
use std::collections::VecDeque;

fn max_sliding_window(nums: &[i32], k: usize) -> Vec<i32> {
    let n = nums.len();
    let mut result = vec![0; n - k + 1];
    // Deque stores INDICES; front = index of current window max
    let mut dq: VecDeque<usize> = VecDeque::new();

    for i in 0..n {
        // 1. Remove indices outside current window
        while !dq.is_empty() && *dq.front().unwrap() + k <= i {
            dq.pop_front();
        }
        // 2. Maintain decreasing order: remove smaller elements from back
        while !dq.is_empty() && nums[*dq.back().unwrap()] <= nums[i] {
            dq.pop_back();
        }
        dq.push_back(i);

        // 3. Window is full starting from i == k-1
        if i >= k - 1 {
            result[i - k + 1] = nums[*dq.front().unwrap()];
        }
    }
    result
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

```rust
use std::collections::VecDeque;

struct RecentCounter {
    q: VecDeque<i32>,
}

impl RecentCounter {
    fn new() -> Self {
        RecentCounter { q: VecDeque::new() }
    }

    fn ping(&mut self, t: i32) -> i32 {
        self.q.push_back(t);
        while *self.q.front().unwrap() < t - 3000 {
            self.q.pop_front();
        }
        self.q.len() as i32
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
```rust
use std::collections::VecDeque;

for i in 0..n {
    // Expire out-of-window indices
    while !dq.is_empty() && *dq.front().unwrap() + k <= i { dq.pop_front(); }
    // Maintain monotonicity (decreasing for max)
    while !dq.is_empty() && nums[*dq.back().unwrap()] <= nums[i] { dq.pop_back(); }
    dq.push_back(i);
    if i >= k - 1 { result[i - k + 1] = nums[*dq.front().unwrap()]; }
}
```

---

## Queue vs Deque — Decision Table

| Need | Use |
|------|-----|
| Pure FIFO | `VecDeque<T>` with `push_back`/`pop_front` |
| LIFO (stack) | `Vec<T>` with `push`/`pop` or `VecDeque<T>` with `push_front`/`pop_front` |
| Both ends (sliding window) | `VecDeque<T>` with `push_front`/`push_back`, `pop_front`/`pop_back` |
| Priority FIFO | `BinaryHeap<T>` |
| Thread-safe queue | No direct std equivalent — use `Mutex<VecDeque<T>>` or a concurrent crate |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Monotonic Stack](./Monotonic%20Stack.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Stack vs Queue Usage](../Interview%20Tips/Stack%20vs%20Queue%20Usage.md)

> **Last Updated:** 2026-06-26
