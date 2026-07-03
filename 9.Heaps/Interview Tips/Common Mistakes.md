# Common Mistakes — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Mistake 1: Subtraction in Comparator — Integer Overflow

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — overflow for INT_MIN / INT_MAX with subtraction-based comparator
priority_queue<int, vector<int>, function<bool(int,int)>> maxHeapWrong([](int a, int b) { return b - a > 0; });
// Example: a = INT_MIN, b = 1 → b - a = 1 - (-2147483648) overflows

// CORRECT
priority_queue<int, vector<int>, function<bool(int,int)>> maxHeap([](int a, int b) { return a < b; });
// Or simply use default max-heap:
priority_queue<int> maxHeap2; // default max-heap in C++
```

---

## Mistake 2: Wrong Heap Type for Top K

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — max-heap for "k largest": top gives largest, not kth largest
priority_queue<int> maxHeap; // default max-heap
for (auto& n : nums) {
    maxHeap.push(n);
    if (maxHeap.size() > k) maxHeap.pop();  // removes largest — wrong!
}

// CORRECT — min-heap for "k largest": top = kth largest (smallest of k largest)
priority_queue<int, vector<int>, greater<int>> minHeap;
for (auto& n : nums) {
    minHeap.push(n);
    if (minHeap.size() > k) minHeap.pop();  // removes smallest — keeps k largest
}
return minHeap.top();  // kth largest
```

---

## Mistake 3: Forgetting Rebalance in Two Heaps

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — no rebalance; sizes diverge
void addNum(int num) {
    if (lo.empty() || num <= lo.top()) lo.push(num);
    else hi.push(num);
    // MISSING: rebalance!
}

// CORRECT
void addNum(int num) {
    if (lo.empty() || num <= lo.top()) lo.push(num);
    else hi.push(num);
    if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
    else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
}
```

Without rebalance, `findMedian` returns wrong results after several inserts.

---

## Mistake 4: Median Heap — Accessing Empty `hi` When Sizes Equal

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — hi might be empty initially
double findMedian() {
    if (lo.size() == hi.size()) return (lo.top() + hi.top()) / 2.0;  // undefined behavior if hi is empty
    return lo.top();
}

// CORRECT — guard empty
double findMedian() {
    if (lo.size() > hi.size()) return lo.top();
    return (lo.top() + hi.top()) / 2.0;  // lo.size == hi.size, both non-empty
}
```

After adding the first element, `lo.size()=1, hi.size()=0` — they're not equal, so this path is safe. But if the size-balance invariant breaks, calling `.top()` on an empty heap causes undefined behavior.

---

## Mistake 5: K-Way Merge — Storing Value Only (Missing Index)

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — can't advance to next element in the list
heap.push({value});

// CORRECT — store (value, listIndex, elementIndex)
heap.push({value, listIndex, elementIndex});
// On pop: advance element from same list
auto curr = heap.top(); heap.pop();
if (curr[2] + 1 < (int)lists[curr[1]].size())
    heap.push({lists[curr[1]][curr[2]+1], curr[1], curr[2]+1});
```

---

## Mistake 6: Merge K Lists — Using `a.val - b.val` in Comparator

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode { int val; ListNode* next; ListNode(int x): val(x), next(nullptr){} };

// WRONG — overflow risk with subtraction
auto cmpWrong = [](ListNode* a, ListNode* b) { return a->val - b->val > 0; };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmpWrong)> heapWrong(cmpWrong);

// CORRECT
auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> heap(cmp);
```

Even though `ListNode` values are usually small, the safe habit prevents bugs with large values.

---

## Mistake 7: Task Scheduler — Forgetting `tasks.size()` in `max`

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — when no idle time is needed, answer = tasks.size()
return (maxFreq - 1) * (n + 1) + maxCount;

// CORRECT
return max((maxFreq - 1) * (n + 1) + maxCount, (int)tasks.size());
```

For input `tasks = [A,A,B,B,C,C]`, `n = 0`: formula = `(2-1)*(1)+2 = 3`, but `tasks.size() = 6`. The `max` ensures we never return fewer slots than actual tasks.

---

## Mistake 8: Reorganize String — Missing Feasibility Check

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — constructs invalid string without checking
// If 'a' appears 5 times in "aaabc" (n=5), no valid arrangement exists

// CORRECT — check early
int maxFreq = 0;
for (auto& f : freq) maxFreq = max(maxFreq, f);
if (maxFreq > ((int)s.length() + 1) / 2) return "";
```

Without this check, the heap approach may produce an invalid string or loop incorrectly when one element dominates.

---

## Mistake 9: K Closest — Using `sqrt` for Distance

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — floating point imprecision; sqrt is unnecessary
double dist = sqrt(x*x + y*y);

// CORRECT — compare squared distances (always integer, no precision loss)
int distSq = x*x + y*y;
// Use in comparator: [](auto& a, auto& b) { return a[0]*a[0]+a[1]*a[1] > b[0]*b[0]+b[1]*b[1]; }
```

For ranking, `sqrt` is monotonic — if `d1 < d2` then `sqrt(d1) < sqrt(d2)`. Skip the sqrt; compare squared distances directly.

---

## Mistake 10: IPO — Not Breaking When No Affordable Project

```cpp
#include <bits/stdc++.h>
using namespace std;

// WRONG — always tries to pop from available, even if empty
for (int i = 0; i < k; i++) {
    while (!locked.empty() && locked.top()[0] <= w) { available.push(locked.top()); locked.pop(); }
    w += available.top()[1]; available.pop();  // undefined behavior if available is empty!
}

// CORRECT
for (int i = 0; i < k; i++) {
    while (!locked.empty() && locked.top()[0] <= w) { available.push(locked.top()); locked.pop(); }
    if (available.empty()) break;  // no affordable projects
    w += available.top()[1]; available.pop();
}
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Heap Patterns](./Heap%20Patterns.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
