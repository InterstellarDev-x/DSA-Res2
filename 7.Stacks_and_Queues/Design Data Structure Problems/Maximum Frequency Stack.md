# Maximum Frequency Stack

> **Topic:** [Stacks & Queues](../README.md) · **Design 2 of 2**
> **Problems:** Maximum Frequency Stack (LC 895)

---

## Problem Statement — LC 895

Design a stack-like data structure that:
- `push(val)`: pushes `val` onto the stack
- `pop()`: removes and returns the most **frequent** element. If there's a tie, return the most **recently pushed** among them.

**Example:**
```
push(5), push(7), push(5), push(7), push(4), push(5)
pop() → 5  (5 appears 3 times, most frequent)
pop() → 7  (now 5→2, 7→2 tied; 7 was pushed more recently at count 2)
pop() → 5  (5→2, 4→1; 5 is most frequent)
pop() → 4  (5→1, 4→1; 4 was pushed more recently)
```

---

## Solution: std::unordered_map + std::unordered_map of Stacks

**Key insight:** Group elements by frequency. Within each frequency group, maintain insertion order (stack semantics for tie-breaking).

```cpp
#include <bits/stdc++.h>
using namespace std;

class FreqStack {
    unordered_map<int, int> freq;          // val → current frequency
    unordered_map<int, stack<int>> group;  // freq → stack of vals at this freq
    int maxFreq;

public:
    FreqStack() : maxFreq(0) {}

    void push(int val) {
        int f = (freq.count(val) ? freq[val] : 0) + 1;
        freq[val] = f;
        maxFreq = max(maxFreq, f);
        group[f].push(val);  // default-constructs stack if key absent
    }

    int pop() {
        auto& stk = group[maxFreq];
        int val = stk.top();
        stk.pop();
        freq[val] = freq[val] - 1;
        if (stk.empty()) {
            group.erase(maxFreq);
            maxFreq--;   // only decrement — can never skip a frequency level
        }
        return val;
    }
};
```

**Complexity:** O(1) for both `push` and `pop`.

---

## How It Works — The Core Invariant

When we push element `x` for the k-th time, we add it to `group[k]`. So `group[k]` contains all elements that have been pushed exactly k or more times, in the order they reached frequency k.

**Critical property:** `maxFreq` can only decrease by 1 on each pop (never skip). If `group[maxFreq]` becomes empty after a pop, the next pop must come from `group[maxFreq - 1]`.

**Why can't maxFreq jump down by more than 1?**
- To have something at frequency k, there must have been something at frequency k-1 first.
- After popping from group[k] (making it empty), group[k-1] is guaranteed to be non-empty because the element we just popped was also in group[k-1] when it was at frequency k-1.

---

## Detailed Trace

**Operations:** push(5), push(7), push(5), push(7), push(4), push(5)

```
push(5): freq={5:1}, group={1:[5]}, maxFreq=1
push(7): freq={5:1,7:1}, group={1:[7,5]}, maxFreq=1
push(5): freq={5:2,7:1}, group={1:[7,5],2:[5]}, maxFreq=2
push(7): freq={5:2,7:2}, group={1:[7,5],2:[7,5]}, maxFreq=2
push(4): freq={5:2,7:2,4:1}, group={1:[4,7,5],2:[7,5]}, maxFreq=2
push(5): freq={5:3,7:2,4:1}, group={1:[4,7,5],2:[7,5],3:[5]}, maxFreq=3

pop(): group[3]=[5] → pop 5. freq[5]=2. group[3] empty → remove, maxFreq=2. Return 5.
  State: freq={5:2,7:2,4:1}, group={1:[4,7,5],2:[7,5]}, maxFreq=2

pop(): group[2]=[7,5] → pop 7 (most recent at freq 2). freq[7]=1. group[2]=[5]. Return 7.
  State: freq={5:2,7:1,4:1}, group={1:[4,7,5],2:[5]}, maxFreq=2

pop(): group[2]=[5] → pop 5. freq[5]=1. group[2] empty → remove, maxFreq=1. Return 5.
  State: freq={5:1,7:1,4:1}, group={1:[4,7,5]}, maxFreq=1

pop(): group[1]=[4,7,5] → pop 4 (most recent at freq 1). freq[4]=0. Return 4.
```

---

## Comparison: MaxFreqStack vs LFU Cache

Both deal with frequency. Key differences:

| Aspect | MaxFreqStack (LC 895) | LFU Cache (LC 460) |
|--------|----------------------|---------------------|
| Purpose | Pop most frequent | Evict least frequent |
| Tie-breaking | Most recently pushed | Most recently used |
| Keys | Any pushed value | key-value pairs with capacity |
| Deletion | Always pop max freq | Only evict min freq when full |
| `minFreq` tracking | `maxFreq` tracks top | `minFreq` tracks bottom |

---

## Follow-up Interview Questions

**Q: What if we also need to peek without popping?**
A: `peek()` = `group[maxFreq].top()`. O(1).

**Q: What if we need the k-th most frequent element's stack?**
A: `group[maxFreq - k + 1].top()` gives the top of the k-th frequency group.

**Q: Thread safety considerations?**
A: Two non-atomic operations in `push` (update freq map, update group map) and `pop` (read group, update freq, possibly decrement maxFreq). Need to synchronize the entire method or use a mutex.

**Q: How would you handle negative frequencies (after many pops)?**
A: The design guarantees freq[val] ≥ 0 (we never pop more than we push). But defensively, we could cap at 0: `freq[val] = max(0, freq[val] - 1)`.

---

## Amazon LP Alignment

| LP Principle | Answer Connection |
|-------------|-------------------|
| Invent and Simplify | The `group` map elegantly encodes both frequency AND recency — no separate timestamp needed |
| Deliver Results | O(1) amortized for both operations meets strict SLA requirements |
| Think Big | Extension: support `pushWithPriority` or `popK` (pop top-k by freq) — group map structure scales to these easily |

---

## Related Files

- [Min Stack](./Min%20Stack.md)
- [LFU Cache (Linked List topic)](../../4.Linked_List/Design%20Data%20Structure%20Problems/LFU%20Cache.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
