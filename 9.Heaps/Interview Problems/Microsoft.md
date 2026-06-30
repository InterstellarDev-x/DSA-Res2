# Microsoft — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Microsoft

---

## Problem 1: Kth Largest Element — Both Approaches

**LC 215** · Medium

**Heap approach — O(n log k):**
```java
public int findKthLargest(int[] nums, int k) {
    PriorityQueue<Integer> minHeap = new PriorityQueue<>();
    for (int n : nums) {
        minHeap.offer(n);
        if (minHeap.size() > k) minHeap.poll();
    }
    return minHeap.peek();
}
```

**Quickselect — O(n) average:**
```java
public int findKthLargest(int[] nums, int k) {
    return quickselect(nums, 0, nums.length - 1, nums.length - k);
}

private int quickselect(int[] nums, int l, int r, int target) {
    if (l == r) return nums[l];
    int pivot = partition(nums, l, r);
    if (pivot == target) return nums[pivot];
    return pivot < target ? quickselect(nums, pivot + 1, r, target)
                          : quickselect(nums, l, pivot - 1, target);
}

private int partition(int[] nums, int l, int r) {
    int pivot = nums[r], i = l;
    for (int j = l; j < r; j++) if (nums[j] <= pivot) swap(nums, i++, j);
    swap(nums, i, r);
    return i;
}
private void swap(int[] a, int i, int j) { int t=a[i]; a[i]=a[j]; a[j]=t; }
```

**Q: Microsoft asks — "Which approach would you use in production and why?"**
A: "It depends on context. If I can't modify the input array (immutable data), I'd use the heap approach. If memory is constrained and the array is mutable, quickselect with O(1) extra space. For guaranteed O(n log k) without O(n²) worst case risk, heap is safer. For very large n with k ≈ 1, quickselect is faster in practice."

---

## Problem 2: Merge K Sorted Lists — Microsoft Style

**LC 23** · Hard

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> heap = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)  // SAFE — no overflow
    );
    for (ListNode n : lists) if (n != null) heap.offer(n);

    ListNode dummy = new ListNode(0), tail = dummy;
    while (!heap.isEmpty()) {
        ListNode node = heap.poll();
        tail.next = node;
        tail = tail.next;
        if (tail.next != null) heap.offer(tail.next);
    }
    return dummy.next;
}
```

**Microsoft edge cases:**
- `lists = []` → return null (heap never filled, loop never runs, `dummy.next = null`)
- `lists = [null]` → null filtered in initialization; returns null
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
