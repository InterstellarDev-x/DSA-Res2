# Coding Tips — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## 1. Never Use Subtraction in Heap Comparators

```cpp
#include <bits/stdc++.h>
using namespace std;

// DANGEROUS — subtraction overflows for INT_MIN/INT_MAX
auto bad_cmp = [](int a, int b) { return (b - a) > 0; }; // overflow-prone
priority_queue<int, vector<int>, decltype(bad_cmp)> maxHeapBad(bad_cmp);

// SAFE — default priority_queue is already a max-heap in C++
priority_queue<int> maxHeap; // max-heap (largest on top)
// or with explicit safe lambda comparator
auto safe_cmp = [](int a, int b) { return a < b; }; // max-heap
priority_queue<int, vector<int>, decltype(safe_cmp)> maxHeap2(safe_cmp);
// or use greater<int> for min-heap
priority_queue<int, vector<int>, greater<int>> minHeap;
```

For `vector<int> arrays` in the heap (common for K-way merge entries):
```cpp
#include <bits/stdc++.h>
using namespace std;

// SAFE — compare array elements explicitly (no subtraction)
auto cmp = [](const vector<int>& a, const vector<int>& b) { return a[0] > b[0]; }; // min-heap by first element
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> heap(cmp);
```

---

## 2. Top K Pattern — Min-Heap for K Largest, Max-Heap for K Smallest

```cpp
#include <bits/stdc++.h>
using namespace std;

// k LARGEST elements → min-heap size k (root = kth largest)
priority_queue<int, vector<int>, greater<int>> minHeap;
for (int n : nums) {
    minHeap.push(n);
    if ((int)minHeap.size() > k) minHeap.pop();
}
// minHeap.top() = kth largest

// k SMALLEST elements → max-heap size k (root = kth smallest)
priority_queue<int> maxHeap; // default max-heap
for (int n : nums) {
    maxHeap.push(n);
    if ((int)maxHeap.size() > k) maxHeap.pop();
}
// maxHeap.top() = kth smallest
```

Mnemonic: "Opposite heap, size k, root is the answer."

---

## 3. K-Way Merge — Always Store (Value, ListIndex, ElementIndex)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Standard K-way merge heap entry
heap.push({value, listIndex, elementIndex});
// When popping: advance elementIndex in the same list
auto curr = heap.top(); heap.pop();
if (curr[2] + 1 < (int)lists[curr[1]].size()) {
    heap.push({lists[curr[1]][curr[2]+1], curr[1], curr[2]+1});
}
```

The three-element entry is the canonical pattern — never store value alone.

---

## 4. Two Heaps — Always Rebalance After Every Operation

```cpp
#include <bits/stdc++.h>
using namespace std;

// CORRECT — rebalance every time
void addNum(int num) {
    if (lo.empty() || num <= lo.top()) lo.push(num);
    else hi.push(num);
    // ALWAYS rebalance
    if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
    else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
}
```

It's easy to forget the rebalance in a time-pressured interview. Write it as a `rebalance()` helper function to make it obvious and testable.

---

## 5. Task Scheduler Formula — Memorize the Derivation

```
Frames:  [A _ _] [A _ _] [A]
         ↑n+1↑   ↑n+1↑   ↑maxCount tasks
Number of full frames: maxFreq - 1
Total slots in frames: (maxFreq - 1) * (n + 1)
Final tail tasks: maxCount (tasks tied for max frequency)
Formula: max((maxFreq - 1) * (n + 1) + maxCount, tasks.length)
```

`tasks.length` handles the case where tasks fill all slots with no idle time needed.

---

## 6. Median Two Heaps — `lo.top() / 2.0 + hi.top() / 2.0` for Overflow Safety

```cpp
#include <bits/stdc++.h>
using namespace std;

// Potential overflow if lo.top() and hi.top() are both near INT_MAX
return (lo.top() + hi.top()) / 2.0;   // can overflow before the cast

// Safe version
return lo.top() / 2.0 + hi.top() / 2.0;
```

Both are fine for typical LeetCode inputs, but the safe version is correct for all 32-bit integers.

---

## 7. Element Removal from `priority_queue` Is O(n) — Avoid in Loops

```cpp
#include <bits/stdc++.h>
using namespace std;

// SLOW — std::priority_queue has no direct remove(x); a workaround is O(n)
// Avoid manual element removal inside loops

// Use std::multiset when O(log n) removal is needed (acts as ordered multiset)
multiset<int> ms;
ms.erase(ms.find(outgoingElement)); // O(log n)

// Or use lazy deletion: mark elements as deleted, skip on pop
```

For sliding window median, use `std::map<int, int>` instead of `priority_queue` to get O(log n) removal.

---

## 8. Reorganize String — Early Exit Check

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxFreq = 0;
for (int f : freq) maxFreq = max(maxFreq, f);
if (maxFreq > ((int)s.length() + 1) / 2) return "";
```

Always check feasibility before attempting construction. Integer division: `(n+1)/2` correctly computes the ceiling of `n/2`.

---

## 9. Pop from Empty Heap Is Undefined Behavior — Check Before Using

```cpp
#include <bits/stdc++.h>
using namespace std;

// SAFE pattern
if (!heap.empty()) {
    int val = heap.top(); heap.pop();  // guaranteed non-empty
}

// Or use ternary (read only; still pop separately if needed)
int val = heap.empty() ? -1 : heap.top();
```

Unlike Java's `priority_queue::top()/pop()` on an empty container is undefined behavior in C++ — always guard with `.empty()`.

---

## Quick Reference

| Goal | Heap Type | Size Constraint | Root After |
|------|----------|----------------|-----------|
| k largest | Min-heap | k | kth largest |
| k smallest | Max-heap | k | kth smallest |
| Median — lower half | Max-heap | ⌈n/2⌉ | lower median |
| Median — upper half | Min-heap | ⌊n/2⌋ | upper median |
| k-way merge | Min-heap | k | current minimum |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Heap Patterns](./Heap%20Patterns.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
