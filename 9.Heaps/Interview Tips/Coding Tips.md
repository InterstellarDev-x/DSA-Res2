# Coding Tips — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## 1. Never Use Subtraction in Heap Comparators

```java
// DANGEROUS — subtraction overflows for MIN_VALUE/MAX_VALUE
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);

// SAFE
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
// or
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
```

For `int[] arrays` in the heap (common for K-way merge entries):
```java
// SAFE — compare array elements explicitly
PriorityQueue<int[]> heap = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
```

---

## 2. Top K Pattern — Min-Heap for K Largest, Max-Heap for K Smallest

```java
// k LARGEST elements → min-heap size k (root = kth largest)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int n : nums) {
    minHeap.offer(n);
    if (minHeap.size() > k) minHeap.poll();
}
// minHeap.peek() = kth largest

// k SMALLEST elements → max-heap size k (root = kth smallest)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
for (int n : nums) {
    maxHeap.offer(n);
    if (maxHeap.size() > k) maxHeap.poll();
}
// maxHeap.peek() = kth smallest
```

Mnemonic: "Opposite heap, size k, root is the answer."

---

## 3. K-Way Merge — Always Store (Value, ListIndex, ElementIndex)

```java
// Standard K-way merge heap entry
heap.offer(new int[]{value, listIndex, elementIndex});
// When popping: advance elementIndex in the same list
int[] curr = heap.poll();
if (curr[2] + 1 < lists[curr[1]].length) {
    heap.offer(new int[]{lists[curr[1]][curr[2]+1], curr[1], curr[2]+1});
}
```

The three-element entry is the canonical pattern — never store value alone.

---

## 4. Two Heaps — Always Rebalance After Every Operation

```java
// CORRECT — rebalance every time
public void addNum(int num) {
    if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
    else hi.offer(num);
    // ALWAYS rebalance
    if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
    else if (hi.size() > lo.size())  lo.offer(hi.poll());
}
```

It's easy to forget the rebalance in a time-pressured interview. Write it as a `rebalance()` helper method to make it obvious and testable.

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

## 6. Median Two Heaps — `lo.peek() / 2.0 + hi.peek() / 2.0` for Overflow Safety

```java
// Potential overflow if lo.peek() and hi.peek() are both near Integer.MAX_VALUE
return (lo.peek() + hi.peek()) / 2.0;   // can overflow before the cast

// Safe version
return lo.peek() / 2.0 + hi.peek() / 2.0;
```

Both are fine for typical LeetCode inputs, but the safe version is correct for all 32-bit integers.

---

## 7. `PriorityQueue.remove(x)` Is O(n) — Avoid in Loops

```java
// SLOW — remove(x) is O(n) linear scan
while (!pq.isEmpty()) {
    pq.remove(outgoingElement);   // O(n) every time
}

// Use TreeMap (as ordered multiset) when O(log n) removal is needed
// Or use lazy deletion: mark elements as deleted, skip on poll
```

For sliding window median, use `TreeMap<Integer, Integer>` instead of `PriorityQueue` to get O(log n) removal.

---

## 8. Reorganize String — Early Exit Check

```java
int maxFreq = 0;
for (int f : freq) maxFreq = Math.max(maxFreq, f);
if (maxFreq > (s.length() + 1) / 2) return "";
```

Always check feasibility before attempting construction. Integer division: `(n+1)/2` correctly computes the ceiling of `n/2`.

---

## 9. Poll from Empty Heap Returns Null — Check Before Using

```java
// SAFE pattern
if (!heap.isEmpty()) {
    int val = heap.poll();  // guaranteed non-null
}

// Or use ternary
int val = heap.isEmpty() ? -1 : heap.poll();
```

`PriorityQueue.poll()` returns `null` if empty (unlike `remove()` which throws). For `int` primitive, auto-unboxing `null` throws `NullPointerException`.

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
