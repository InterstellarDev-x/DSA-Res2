> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Mistakes**

# Greedy Algorithms — Common Mistakes (Top 10)

## Mistake 1: Wrong Sort Key — Start vs End

**Bug:** Sorting by start time when you should sort by end time for interval scheduling.

```java
// WRONG — sort by start time for non-overlapping intervals
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));

// CORRECT — sort by end time
Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));
```

**Consequence:** Counter-example: `[[1,10],[2,3],[4,5]]`. Sorted by start, greedy keeps `[1,10]` first, then removes everything. Answer: 2 (wrong). Sorted by end, keeps `[2,3]` and `[4,5]`. Answer: 1 (correct).

**Rule of thumb:**
- "Maximize number of non-overlapping intervals" → sort by **end**
- "Merge overlapping intervals" → sort by **start**
- "Minimum arrows / removals" → sort by **end**

---

## Mistake 2: Missing Gas Station Circular Check

**Bug:** Forgetting to check if total gain is negative before returning `startStation`.

```java
// WRONG — forgets infeasibility check
public int canCompleteCircuit(int[] gas, int[] cost) {
    int currentGain = 0, startStation = 0;
    for (int i = 0; i < gas.length; i++) {
        currentGain += gas[i] - cost[i];
        if (currentGain < 0) { startStation = i + 1; currentGain = 0; }
    }
    return startStation; // BUG: may return invalid answer when no solution exists
}

// CORRECT
public int canCompleteCircuit(int[] gas, int[] cost) {
    int totalGain = 0, currentGain = 0, startStation = 0;
    for (int i = 0; i < gas.length; i++) {
        int gain = gas[i] - cost[i];
        totalGain += gain;
        currentGain += gain;
        if (currentGain < 0) { startStation = i + 1; currentGain = 0; }
    }
    return totalGain >= 0 ? startStation : -1; // CORRECT: check feasibility
}
```

**Test case that reveals bug:** `gas=[2,3,4]`, `cost=[3,4,3]`. Total gain = -1, no solution. Wrong code returns 2; correct code returns -1.

---

## Mistake 3: Candy Single-Pass Failure

**Bug:** Trying to solve Candy in a single left-to-right pass.

```java
// WRONG — single pass misses decreasing sequences
public int candy(int[] ratings) {
    int[] candies = new int[ratings.length];
    Arrays.fill(candies, 1);
    for (int i = 1; i < ratings.length; i++) {
        if (ratings[i] > ratings[i-1]) candies[i] = candies[i-1] + 1;
        else if (ratings[i] < ratings[i-1]) candies[i-1]++; // BUG: too late!
    }
    // ...
}
```

**Why it fails:** `[5, 4, 3, 2, 1]` — when we reach index 1 and increment index 0, we have already committed to index 0's value being used for future comparisons. The retroactive increment does not propagate backward correctly.

**Correct: Two passes.** See [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) for full solution.

---

## Mistake 4: Valid Parenthesis String — Forgetting `*` Can Be Empty

**Bug:** Only treating `*` as `(` or `)`, not as empty string.

```java
// The real bug: not clamping lo = Math.max(lo, 0)

// WRONG — lo goes negative, which is invalid
for (char c : s.toCharArray()) {
    if (c == '(') { lo++; hi++; }
    else if (c == ')') { lo--; hi--; }
    else { lo--; hi++; }
    if (hi < 0) return false;
    // Missing: lo = Math.max(lo, 0);
}
return lo == 0; // BUG: lo might be -2, and this returns false when should be true
```

**Test case:** `s = "*"`. lo=-1, hi=1. Without clamp, lo=-1 ≠ 0, returns false. But `"*"` is valid (treat as empty). Correct answer: true.

---

## Mistake 5: Jump Game II — Off-by-One in Loop Bound

**Bug:** Using `i <= nums.length - 1` instead of `i < nums.length - 1`.

```java
// WRONG — processes the last index unnecessarily, overcounts jumps
for (int i = 0; i <= nums.length - 1; i++) {
    farthest = Math.max(farthest, i + nums[i]);
    if (i == curEnd) { jumps++; curEnd = farthest; }
}

// CORRECT — stop at second-to-last index
for (int i = 0; i < nums.length - 1; i++) {
    farthest = Math.max(farthest, i + nums[i]);
    if (i == curEnd) { jumps++; curEnd = farthest; }
}
```

**Why:** When `i == nums.length - 1`, we are already AT the destination. We do not need to "jump from" it. Processing it causes a spurious increment of `jumps`.

**Test case:** `[0]`. Single element, already at destination. Correct: 0 jumps. Wrong: 1 jump.

---

## Mistake 6: Meeting Rooms II — Wrong Heap Type

**Bug:** Using max-heap instead of min-heap.

```java
// WRONG — max-heap gives farthest ending room, not earliest ending room
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());

// CORRECT — min-heap gives earliest ending room
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
```

**Why it matters:** We want to check if ANY room is free (has end ≤ new start). The room most likely to be free is the one that ends earliest. Max-heap gives the room that ends latest — useless for this check.

---

## Mistake 7: Insert Interval — Wrong Overlap Condition

**Bug:** Using `<` instead of `<=` in the overlap merge condition.

```java
// WRONG — misses touching intervals [1,3] and [3,5] should merge to [1,5]
while (i < n && intervals[i][0] < newInterval[1]) { ... }

// CORRECT — intervals touching at a single point ARE overlapping
while (i < n && intervals[i][0] <= newInterval[1]) { ... }
```

**Test case:** `intervals=[[1,3],[3,5]]`, `newInterval=[2,3]`. Wrong code stops before merging `[3,5]` because `3 < 3` is false. Correct code merges both.

---

## Mistake 8: Hand of Straights — Not Cleaning Up Zero-Count Keys

**Bug:** Not removing keys with count 0 from TreeMap, causing `firstKey()` to return completed cards.

```java
// WRONG — zero-count keys remain in map
count.merge(i, -1, Integer::sum);
// If count.get(i) == 0, the key stays in the TreeMap

// CORRECT — remove exhausted keys
count.merge(i, -1, Integer::sum);
if (count.get(i) == 0) count.remove(i);
```

**Consequence:** `firstKey()` may return a card with count 0 on the next iteration, causing an infinite loop or incorrect group formation.

---

## Mistake 9: Non-overlapping Intervals — Wrong Comparison Operator

**Bug:** Using `>` instead of `>=` for the non-overlap check.

```java
// WRONG — treats touching intervals [1,2] and [2,3] as overlapping
if (interval[0] > prevEnd) { kept++; prevEnd = interval[1]; }

// CORRECT — touching intervals do NOT overlap
if (interval[0] >= prevEnd) { kept++; prevEnd = interval[1]; }
```

**Test case:** `[[1,2],[2,3]]`. These intervals touch at point 2 but do not overlap. Wrong code removes one; correct code keeps both (result = 0 removals).

---

## Mistake 10: Partition Labels — Using Frequency Count Instead of Last Occurrence

**Bug:** Building a frequency count instead of a last-occurrence map.

```java
// WRONG — frequency count does not tell us WHERE the last occurrence is
int[] freq = new int[26];
for (char c : s) freq[c - 'a']++;

// CORRECT — last occurrence index is what we need for window stretching
int[] last = new int[26];
for (int i = 0; i < s.length(); i++) last[s.charAt(i) - 'a'] = i;
```

**Why:** The window must extend to the POSITION of the last occurrence of each character (so the character is fully contained), not just count how many times it occurs.

---

## Quick Reference: Bug → Fix Table

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong sort key | Wrong answer on interval problems | Sort by end (maximize) or start (merge) |
| Gas Station no feasibility check | Returns wrong station when -1 expected | Track `totalGain`, return -1 if negative |
| Candy single pass | Wrong candy count on peaks/valleys | Two passes: L→R then R→L |
| Valid Paren missing lo clamp | Returns false on valid `"*"` strings | Add `lo = Math.max(lo, 0)` |
| Jump Game II loop bound | Overcounts jumps | Use `i < nums.length - 1` |
| Meeting Rooms max-heap | Allocates too many rooms | Use `new PriorityQueue<>()` (min) |
| Insert Interval `<` vs `<=` | Misses touching interval merges | Use `<=` in merge condition |
| Hand of Straights no cleanup | Infinite loop or wrong groups | Remove 0-count keys from TreeMap |
| Non-overlapping `>` vs `>=` | Removes touching intervals | Use `>=` for non-overlap |
| Partition Labels frequency vs last | Wrong partition boundaries | Build last-occurrence array, not freq |

> **Last Updated:** 2026-06-26
