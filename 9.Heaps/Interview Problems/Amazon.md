# Amazon — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Amazon

---

## Problem 1: Top K Frequent Elements — Deep Dive

**LC 347** · Medium

### Two Approaches

**Approach 1: Min-heap of size k — O(n log k)**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    PriorityQueue<Map.Entry<Integer, Integer>> minHeap =
        new PriorityQueue<>(Comparator.comparingInt(Map.Entry::getValue));

    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        minHeap.offer(e);
        if (minHeap.size() > k) minHeap.poll();
    }

    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) result[i] = minHeap.poll().getKey();
    return result;
}
```

**Approach 2: Bucket sort — O(n)**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    @SuppressWarnings("unchecked")
    List<Integer>[] buckets = new List[nums.length + 1];
    for (int key : freq.keySet()) {
        int f = freq.get(key);
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(key);
    }

    int[] result = new int[k];
    int idx = 0;
    for (int f = buckets.length - 1; f >= 0 && idx < k; f--)
        if (buckets[f] != null)
            for (int n : buckets[f]) if (idx < k) result[idx++] = n;
    return result;
}
```

**Q: When would you use bucket sort over heap?**
A: When the input values are bounded (nums are integers in a known range). Bucket sort is O(n) but uses O(n) space for the bucket array. For floating-point frequencies or very sparse data, heap is more appropriate.

**Q: Amazon follow-up — Top K Frequent Words with lexicographic tie-breaking?**
A: Add a second comparator condition: for equal frequencies, lexicographically later strings should be evicted first from the min-heap.

---

## Problem 2: Task Scheduler — Amazon Deep Dive

**LC 621** · Medium

**Both approaches must be known:**

```java
// Formula — O(26 log 26) = O(1) effectively
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    Arrays.sort(freq);
    int maxFreq = freq[25];
    int maxCount = 0;
    for (int f : freq) if (f == maxFreq) maxCount++;
    return Math.max((maxFreq - 1) * (n + 1) + maxCount, tasks.length);
}
```

**Q: Explain the formula.**
A: Build "frames" of size `n+1`. The most frequent task (`maxFreq`) anchors each frame. There are `maxFreq - 1` full frames, each of size `n+1`. The final slot holds all tasks with max frequency (`maxCount`). Total = `(maxFreq-1)*(n+1) + maxCount`. But if tasks fill all slots with no idle time, the answer is just `tasks.length`.

**Q: Amazon variant — what if n is very large?**
A: For large n, idle time dominates. The formula still handles this correctly — `(maxFreq-1)*(n+1) + maxCount` grows with n.

**Q: Amazon variant — different cooldown per task type?**
A: The heap simulation approach generalizes to this. The cooldown queue can track per-task cooldowns. The formula approach doesn't generalize.

---

## Problem 3: Merge K Sorted Lists — Amazon Phone Screen

**LC 23** · Hard · O(n log k)

```java
public ListNode mergeKLists(ListNode[] lists) {
    PriorityQueue<ListNode> heap = new PriorityQueue<>(
        (a, b) -> Integer.compare(a.val, b.val)
    );
    for (ListNode node : lists) if (node != null) heap.offer(node);

    ListNode dummy = new ListNode(0), tail = dummy;
    while (!heap.isEmpty()) {
        tail.next = heap.poll();
        tail = tail.next;
        if (tail.next != null) heap.offer(tail.next);
    }
    return dummy.next;
}
```

**Q: Why not just concatenate and sort?**
A: O(n log n) where n = total nodes. K-way merge with heap is O(n log k) — better when k << n (many short lists).

**Q: What if k is very large (k = n lists, each length 1)?**
A: Building the heap costs O(k) = O(n). Each poll is O(log k) = O(log n). Total O(n log n) — same as sort. For this degenerate case, sorting is simpler.

---

## Amazon LP Alignment

| LP | Connection |
|----|-----------|
| Dive Deep | Know both heap and bucket sort for Top K; formula and simulation for Task Scheduler |
| Deliver Results | O(n log k) beats O(n log n) for Merge K Lists when k is small |
| Invent & Simplify | Bucket sort O(n) simplifies Top K when input range is bounded |

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Top K Elements](../Patterns/Top%20K%20Elements.md)
- [Heap + Frequency](../Patterns/Heap%20and%20Frequency.md)

> **Last Updated:** 2026-06-26
