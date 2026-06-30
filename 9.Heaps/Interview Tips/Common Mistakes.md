# Common Mistakes — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Mistake 1: Subtraction in Comparator — Integer Overflow

```java
// WRONG — overflows for Integer.MIN_VALUE / MAX_VALUE
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);
// Example: a = Integer.MIN_VALUE, b = 1 → b - a = 1 - (-2147483648) overflows to negative

// CORRECT
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> Integer.compare(b, a));
```

---

## Mistake 2: Wrong Heap Type for Top K

```java
// WRONG — max-heap for "k largest": root gives largest, not kth largest
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
for (int n : nums) {
    maxHeap.offer(n);
    if (maxHeap.size() > k) maxHeap.poll();  // removes largest — wrong!
}

// CORRECT — min-heap for "k largest": root = kth largest (smallest of k largest)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int n : nums) {
    minHeap.offer(n);
    if (minHeap.size() > k) minHeap.poll();  // removes smallest — keeps k largest
}
return minHeap.peek();  // kth largest
```

---

## Mistake 3: Forgetting Rebalance in Two Heaps

```java
// WRONG — no rebalance; sizes diverge
public void addNum(int num) {
    if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
    else hi.offer(num);
    // MISSING: rebalance!
}

// CORRECT
public void addNum(int num) {
    if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
    else hi.offer(num);
    if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
    else if (hi.size() > lo.size()) lo.offer(hi.poll());
}
```

Without rebalance, `findMedian` returns wrong results after several inserts.

---

## Mistake 4: Median Heap — Accessing Empty `hi` When Sizes Equal

```java
// WRONG — hi might be empty initially
public double findMedian() {
    if (lo.size() == hi.size()) return (lo.peek() + hi.peek()) / 2.0;  // hi.peek() NullPointerException if hi empty
    return lo.peek();
}

// CORRECT — guard isEmpty
public double findMedian() {
    if (lo.size() > hi.size()) return lo.peek();
    return (lo.peek() + hi.peek()) / 2.0;  // lo.size == hi.size, both non-empty
}
```

After adding the first element, `lo.size()=1, hi.size()=0` — they're not equal, so this path is safe. But if the size-balance invariant breaks, calling `.peek()` on empty heap returns null, causing NPE on auto-unboxing.

---

## Mistake 5: K-Way Merge — Storing Value Only (Missing Index)

```java
// WRONG — can't advance to next element in the list
heap.offer(new int[]{value});

// CORRECT — store (value, listIndex, elementIndex)
heap.offer(new int[]{value, listIndex, elementIndex});
// On poll: advance element from same list
int[] curr = heap.poll();
if (curr[2] + 1 < lists[curr[1]].size())
    heap.offer(new int[]{lists[curr[1]].get(curr[2]+1), curr[1], curr[2]+1});
```

---

## Mistake 6: Merge K Lists — Using `a.val - b.val` in Comparator

```java
// WRONG — overflow risk
PriorityQueue<ListNode> heap = new PriorityQueue<>((a, b) -> a.val - b.val);

// CORRECT
PriorityQueue<ListNode> heap = new PriorityQueue<>((a, b) -> Integer.compare(a.val, b.val));
```

Even though ListNode values are usually small, the safe habit prevents bugs with large values.

---

## Mistake 7: Task Scheduler — Forgetting `tasks.length` in `Math.max`

```java
// WRONG — when no idle time is needed, answer = tasks.length
return (maxFreq - 1) * (n + 1) + maxCount;

// CORRECT
return Math.max((maxFreq - 1) * (n + 1) + maxCount, tasks.length);
```

For input `tasks = [A,A,B,B,C,C]`, `n = 0`: formula = `(2-1)*(1)+2 = 3`, but `tasks.length = 6`. The `Math.max` ensures we never return fewer slots than actual tasks.

---

## Mistake 8: Reorganize String — Missing Feasibility Check

```java
// WRONG — constructs invalid string without checking
// If 'a' appears 5 times in "aaabc" (n=5), no valid arrangement exists

// CORRECT — check early
int maxFreq = 0;
for (int f : freq) maxFreq = Math.max(maxFreq, f);
if (maxFreq > (s.length() + 1) / 2) return "";
```

Without this check, the heap approach may produce an invalid string or loop incorrectly when one element dominates.

---

## Mistake 9: K Closest — Using `Math.sqrt` for Distance

```java
// WRONG — floating point imprecision; Math.sqrt is unnecessary
double dist = Math.sqrt(x*x + y*y);

// CORRECT — compare squared distances (always integer, no precision loss)
int distSq = x*x + y*y;
// Use in comparator: (a, b) -> Integer.compare(a[0]*a[0]+a[1]*a[1], b[0]*b[0]+b[1]*b[1])
```

For ranking, `sqrt` is monotonic — if `d1 < d2` then `sqrt(d1) < sqrt(d2)`. Skip the sqrt; compare squared distances directly.

---

## Mistake 10: IPO — Not Breaking When No Affordable Project

```java
// WRONG — always tries to poll from available, even if empty
for (int i = 0; i < k; i++) {
    while (!locked.isEmpty() && locked.peek()[0] <= w) available.offer(locked.poll());
    w += available.poll()[1];  // NPE if available is empty!
}

// CORRECT
for (int i = 0; i < k; i++) {
    while (!locked.isEmpty() && locked.peek()[0] <= w) available.offer(locked.poll());
    if (available.isEmpty()) break;  // no affordable projects
    w += available.poll()[1];
}
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Heap Patterns](./Heap%20Patterns.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
