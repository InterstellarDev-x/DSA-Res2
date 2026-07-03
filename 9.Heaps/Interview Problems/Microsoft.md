# Microsoft — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Microsoft

---

## Problem 1: Kth Largest Element — Both Approaches

**LC 215** · Medium

**Heap approach — O(n log k):**
```cpp
#include <bits/stdc++.h>
using namespace std;

int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    for (auto n : nums) {
        minHeap.push(n);
        if (minHeap.size() > k) minHeap.pop();
    }
    return minHeap.top();
}
```

**Quickselect — O(n) average:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int partition(vector<int>& nums, int l, int r) {
    int pivot = nums[r], i = l;
    for (int j = l; j < r; j++) if (nums[j] <= pivot) swap(nums[i++], nums[j]);
    swap(nums[i], nums[r]);
    return i;
}

int quickselect(vector<int>& nums, int l, int r, int target) {
    if (l == r) return nums[l];
    int pivot = partition(nums, l, r);
    if (pivot == target) return nums[pivot];
    return pivot < target ? quickselect(nums, pivot + 1, r, target)
                          : quickselect(nums, l, pivot - 1, target);
}

int findKthLargest(vector<int>& nums, int k) {
    return quickselect(nums, 0, nums.size() - 1, nums.size() - k);
}
```

**Q: Microsoft asks — "Which approach would you use in production and why?"**
A: "It depends on context. If I can't modify the input array (immutable data), I'd use the heap approach. If memory is constrained and the array is mutable, quickselect with O(1) extra space. For guaranteed O(n log k) without O(n²) worst case risk, heap is safer. For very large n with k ≈ 1, quickselect is faster in practice."

---

## Problem 2: Merge K Sorted Lists — Microsoft Style

**LC 23** · Hard

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; }; // SAFE — no overflow
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> heap(cmp);
    for (auto n : lists) if (n != nullptr) heap.push(n);

    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (!heap.empty()) {
        ListNode* node = heap.top(); heap.pop();
        tail->next = node;
        tail = tail->next;
        if (tail->next != nullptr) heap.push(tail->next);
    }
    return dummy.next;
}
```

**Microsoft edge cases:**
- `lists = []` → return nullptr (heap never filled, loop never runs, `dummy.next = nullptr`)
- `lists = [nullptr]` → nullptr filtered in initialization; returns nullptr
- Single list with elements → works correctly (heap has one item, processes all)

**Q: Compare to divide-and-conquer approach.**
A: D&C: merge pairs repeatedly → O(n log k) time, O(log k) recursion stack. Heap: O(n log k) time, O(k) heap space. Both are O(n log k) but D&C uses less peak memory when k >> log k.

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Top K Elements](../Patterns/Top%20K%20Elements.md)
- [K-Way Merge](../Patterns/K%20Way%20Merge.md)

> **Last Updated:** 2026-06-26
