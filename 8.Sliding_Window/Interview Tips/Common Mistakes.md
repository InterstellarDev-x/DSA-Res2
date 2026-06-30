# Common Mistakes — Sliding Window & Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## Mistake 1: Using `while` Instead of `if` for Max-Length Windows

```java
// BAD — for CHARACTER REPLACEMENT (max window problem)
while ((right - left + 1) - maxFreq > k) {
    freq[s.charAt(left) - 'A']--;
    left++;
}

// GOOD — use if; window size never needs to decrease below current max
if ((right - left + 1) - maxFreq > k) {
    freq[s.charAt(left) - 'A']--;
    left++;
}
```

For min-length problems (Minimum Window Substring, Minimum Subarray Sum), use `while`. For max-length problems (Char Replacement, LSWORC), use `if` to avoid shrinking below the already-recorded maximum.

---

## Mistake 2: Missing `if (goal < 0) return 0` in atMost

```java
// BAD
private int atMost(int[] nums, int goal) {
    int left = 0, sum = 0, result = 0;
    for (...) {
        sum += nums[right];
        while (sum > goal) sum -= nums[left++];   // if goal=-1 and sum=0: 0 > -1 → loop forever
        result += right - left + 1;
    }
}

// GOOD
private int atMost(int[] nums, int goal) {
    if (goal < 0) return 0;   // ← guard
    // ...
}
```

When `exactly(k) = atMost(k) - atMost(k-1)` and `k=0`, we call `atMost(-1)`. The guard prevents infinite loop.

---

## Mistake 3: Forgetting to Advance Both Pointers After Triplet in 3Sum

```java
// BAD — infinite loop on [0,0,0] since left/right don't advance
while (left < right) {
    if (sum == 0) {
        result.add(...);
        while (left < right && nums[left] == nums[left+1]) left++;
        while (left < right && nums[right] == nums[right-1]) right--;
        // MISSING: left++; right--;
    }
}

// GOOD
if (sum == 0) {
    result.add(...);
    while (left < right && nums[left] == nums[left+1]) left++;
    while (left < right && nums[right] == nums[right-1]) right--;
    left++; right--;   // ← always advance after recording
}
```

---

## Mistake 4: Wrong Duplicate Skip Condition in kSum

```java
// BAD — i=0 should not be skipped (first element is never a duplicate of previous)
if (nums[i] == nums[i-1]) continue;   // ArrayIndexOutOfBoundsException when i=0!

// GOOD
if (i > 0 && nums[i] == nums[i-1]) continue;
```

Also, for inner loops in 4Sum:
```java
// BAD — skips valid j=i+1 if nums[i+1]==nums[i]
if (j > 0 && nums[j] == nums[j-1]) continue;

// GOOD — j must be > i+1 (not > 0) to allow the first j after i
if (j > i + 1 && nums[j] == nums[j-1]) continue;
```

---

## Mistake 5: Not Handling `k <= 1` in Product < K

```java
// BAD — infinite loop for k=1 since product=1 ≥ 1 forever
public int numSubarrayProductLessThanK(int[] nums, int k) {
    int left = 0, product = 1, result = 0;
    for (int right = 0; right < nums.length; right++) {
        product *= nums[right];
        while (product >= k) product /= nums[left++];  // never exits if k=1
        result += right - left + 1;
    }
}

// GOOD
if (k <= 1) return 0;
```

---

## Mistake 6: Off-By-One in Fixed Window Initialization

```java
// BAD — processes first k+1 elements in initial window instead of k
double sum = 0;
for (int i = 0; i <= k; i++) sum += nums[i];   // ← <= should be <

// GOOD
for (int i = 0; i < k; i++) sum += nums[i];
```

---

## Mistake 7: Min Window — Returning Before Checking Last State

```java
// BAD — misses updating minLen if window is valid at end
public String minWindow(...) {
    // ... loop ends ...
    return s.substring(minLeft, minLeft + minLen);  // minLen might still be MAX_VALUE
}

// GOOD
return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
```

If `t = "abc"` and `s = "abc"`, the window becomes valid at the end but the `while` shrink block updates `minLen` correctly (it fires during the loop when `have == required`). Still, the `== MAX_VALUE` guard is needed for cases where `t` has characters not in `s`.

---

## Mistake 8: Longest Subarray After Delete — Wrong Length Formula

```java
// BAD — returns window size instead of window size - 1
maxLen = Math.max(maxLen, right - left + 1);

// GOOD — must delete exactly one element
maxLen = Math.max(maxLen, right - left);   // window_size - 1
```

LC 1493 requires **exactly** one deletion (not "at most one"). The answer is always `windowSize - 1`.

---

## Mistake 9: Two Pointers on Unsorted Array for Sum Problems

```java
// BAD — trying two-pointer approach without sorting
int left = 0, right = nums.length - 1;
while (left < right) {
    if (nums[left] + nums[right] == target) { ... }
    else if (nums[left] + nums[right] < target) left++;
    else right--;
}
// WRONG for unsorted arrays — can't make greedy pointer decisions
```

Two pointers for sum problems **require a sorted array**. Sorting is O(n log n) and must be done first.

---

## Mistake 10: Not Removing Entry from Map When Count Reaches Zero

```java
// BAD — map still has 0-count entries, affecting map.size() check
basket.put(fruits[left], basket.get(fruits[left]) - 1);
left++;

// GOOD — remove 0-count entries to keep map size accurate
basket.merge(fruits[left], -1, Integer::sum);
if (basket.get(fruits[left]) == 0) basket.remove(fruits[left]);
left++;
```

For problems that track `map.size() > k` (distinct element count), stale 0-count entries corrupt the size check.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Window Template](./Window%20Template.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
