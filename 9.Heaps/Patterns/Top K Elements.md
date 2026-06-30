# Top K Elements

> **Topic:** [Heaps](../README.md) · **Pattern 2 of 5**
> **Problems:** Kth Largest in Array/Stream · K Closest Points · Top K Frequent Elements/Words · Kth Largest Integer

---

## Core Concept

**Goal:** Find the k largest (or smallest, or most frequent) elements efficiently.

**Key insight — min-heap of size k for "k largest":**
- Maintain a min-heap of the k largest elements seen so far
- The root of this heap = kth largest element
- When a new element arrives: offer it; if heap size > k, poll (removes the smallest, keeping the k largest)

```
Finding k largest from [3,1,4,1,5,9,2,6], k=3:
Min-heap of size 3:
  After 3:     [3]
  After 1:     [1,3]
  After 4:     [1,3,4]
  After 1 (4th): offer 1 → [1,1,3,4]; size>3 → poll 1 → [1,3,4]
  After 5:     offer 5 → [1,3,4,5]; poll 1 → [3,4,5]
  After 9:     offer 9 → [3,4,5,9]; poll 3 → [4,5,9]
  After 2:     offer 2 → [2,4,5,9]; poll 2 → [4,5,9]
  After 6:     offer 6 → [4,5,6,9]; poll 4 → [5,6,9]
  Root = 5 = kth largest ✓
```

**Template:**
```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();  // min-heap of size k
for (int num : nums) {
    minHeap.offer(num);
    if (minHeap.size() > k) minHeap.poll();   // remove smallest; keep k largest
}
return minHeap.peek();  // root = kth largest
```

---

## Problem 1: Kth Largest Element in a Stream — LC 703

Design class: `KthLargest(k, nums)` + `add(val)` returns current kth largest.

```java
class KthLargest {
    private PriorityQueue<Integer> minHeap;
    private int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        minHeap = new PriorityQueue<>();
        for (int n : nums) add(n);
    }

    public int add(int val) {
        minHeap.offer(val);
        if (minHeap.size() > k) minHeap.poll();
        return minHeap.peek();
    }
}
```

**Complexity:** O(log k) per `add` — heap stays size ≤ k.

---

## Problem 2: Kth Largest Element in an Array — LC 215

**Approach 1: Min-heap of size k — O(n log k) time, O(k) space**

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

**Approach 2: Quickselect — O(n) average, O(n²) worst, O(1) space**

```java
public int findKthLargest(int[] nums, int k) {
    return quickselect(nums, 0, nums.length - 1, nums.length - k);
    // kth largest = (n-k)th smallest (0-indexed)
}

private int quickselect(int[] nums, int left, int right, int kSmallest) {
    if (left == right) return nums[left];

    int pivotIdx = partition(nums, left, right);
    if (pivotIdx == kSmallest) return nums[pivotIdx];
    else if (pivotIdx < kSmallest) return quickselect(nums, pivotIdx + 1, right, kSmallest);
    else return quickselect(nums, left, pivotIdx - 1, kSmallest);
}

private int partition(int[] nums, int left, int right) {
    int pivot = nums[right], i = left;
    for (int j = left; j < right; j++) {
        if (nums[j] <= pivot) swap(nums, i++, j);
    }
    swap(nums, i, right);
    return i;
}

private void swap(int[] nums, int i, int j) {
    int tmp = nums[i]; nums[i] = nums[j]; nums[j] = tmp;
}
```

**Which to use?**
- Heap: guaranteed O(n log k), preferred when k << n or for streaming data
- Quickselect: O(n) average, but O(n²) worst case; preferred when modifying the array is OK and k ≈ n/2

---

## Problem 3: K Closest Points to Origin — LC 973

Distance² = x² + y² (no need to take square root — avoids floating point).

```java
public int[][] kClosest(int[][] points, int k) {
    // Max-heap of size k — keep k smallest distances; root = kth closest
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>(
        (a, b) -> Integer.compare(b[0]*b[0] + b[1]*b[1], a[0]*a[0] + a[1]*a[1])
    );

    for (int[] p : points) {
        maxHeap.offer(p);
        if (maxHeap.size() > k) maxHeap.poll();   // remove farthest
    }

    int[][] result = new int[k][2];
    for (int i = k - 1; i >= 0; i--) result[i] = maxHeap.poll();
    return result;
}
```

**Why max-heap for k smallest?** To keep the k closest, maintain a max-heap — the root is always the farthest among the k closest. When a new point is closer than the root, we replace the root (poll max, offer new). This keeps only the k closest.

**Simpler alternative — sort:**
```java
Arrays.sort(points, Comparator.comparingInt(p -> p[0]*p[0] + p[1]*p[1]));
return Arrays.copyOfRange(points, 0, k);  // O(n log n) time, O(1) space
```

Use heap for streaming/online variant; sort for offline.

---

## Problem 4: Top K Frequent Elements — LC 347

```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    // Min-heap by frequency: keep k most frequent
    PriorityQueue<Map.Entry<Integer, Integer>> minHeap =
        new PriorityQueue<>(Comparator.comparingInt(Map.Entry::getValue));

    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        minHeap.offer(e);
        if (minHeap.size() > k) minHeap.poll();   // remove least frequent
    }

    int[] result = new int[k];
    for (int i = k - 1; i >= 0; i--) result[i] = minHeap.poll().getKey();
    return result;
}
```

**O(n log k) time, O(n) space** (for freq map + heap of size k)

**Alternative — Bucket Sort — O(n) time:**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);

    // Buckets: bucket[i] = list of elements with frequency i
    List<Integer>[] buckets = new List[nums.length + 1];
    for (int key : freq.keySet()) {
        int f = freq.get(key);
        if (buckets[f] == null) buckets[f] = new ArrayList<>();
        buckets[f].add(key);
    }

    int[] result = new int[k];
    int idx = 0;
    for (int f = buckets.length - 1; f >= 0 && idx < k; f--) {
        if (buckets[f] != null)
            for (int n : buckets[f]) { if (idx < k) result[idx++] = n; }
    }
    return result;
}
```

---

## Problem 5: Top K Frequent Words — LC 692

Same as Top K Frequent Elements but with lexicographic tie-breaking.

```java
public List<String> topKFrequent(String[] words, int k) {
    Map<String, Integer> freq = new HashMap<>();
    for (String w : words) freq.merge(w, 1, Integer::sum);

    // Min-heap: least frequent at top (to be evicted); lexicographically LATER breaks ties
    PriorityQueue<String> minHeap = new PriorityQueue<>((a, b) -> {
        int fa = freq.get(a), fb = freq.get(b);
        if (fa != fb) return fa - fb;         // lower freq → evict first
        return b.compareTo(a);               // lexicographically later → evict first
    });

    for (String w : freq.keySet()) {
        minHeap.offer(w);
        if (minHeap.size() > k) minHeap.poll();
    }

    LinkedList<String> result = new LinkedList<>();
    while (!minHeap.isEmpty()) result.addFirst(minHeap.poll());  // reverse order
    return result;
}
```

**Why `b.compareTo(a)` for tie-breaking?** We want lexicographically smaller words to stay (appear first in answer). The min-heap evicts the "worst" element. For ties in frequency, "worse" = lexicographically larger. So `b.compareTo(a)` means: if b > a, b comes before a in the heap (b gets evicted first).

---

## Top K — Decision Table

| Goal | Heap Type | Size | Root Meaning |
|------|----------|------|--------------|
| k largest elements | Min-heap | k | kth largest |
| k smallest elements | Max-heap | k | kth smallest |
| k most frequent elements | Min-heap (by freq) | k | kth most frequent |
| k closest points | Max-heap (by dist) | k | kth closest |

**Rule:** For "k [extreme]", use the **opposite** heap type and maintain size k. The root is the answer.

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
