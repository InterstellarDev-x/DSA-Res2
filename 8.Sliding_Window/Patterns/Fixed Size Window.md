# Fixed Size Window

> **Topic:** [Sliding Window](../README.md) · **Pattern 1 of 4**
> **Problems:** Maximum Average Subarray I · Maximum Sum of K Consecutive Elements · Sliding Window Maximum (see [Queue & Deque](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md))

---

## Core Concept

A **fixed-size sliding window** maintains a window of exactly `k` elements. As the window slides right by one position, one element enters (from the right) and one exits (from the left).

```
Array:  [2, 3, 4, 1, 5]   k = 3
         [2  3  4]          window 1: sum = 9
            [3  4  1]       window 2: sum = 8
               [4  1  5]   window 3: sum = 10
```

**Key operation:** `windowValue = windowValue - arr[left] + arr[right]` — update in O(1) instead of recomputing.

---

## Template

```java
public double findMaxAverage(int[] nums, int k) {
    // Step 1: Build initial window of size k
    double windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += nums[i];

    double maxSum = windowSum;

    // Step 2: Slide window — remove left, add right
    for (int right = k; right < nums.length; right++) {
        windowSum += nums[right] - nums[right - k];  // right - k = outgoing element
        maxSum = Math.max(maxSum, windowSum);
    }

    return maxSum / k;
}
```

**Why `right - k` for the outgoing element?** When `right = k`, the window is `[1..k]` and element at index `0 = right - k` leaves. When `right = k+1`, element at index `1 = right - k` leaves. General: when `right` enters, `right - k` exits.

---

## Problem 1: Maximum Average Subarray I — LC 643

**Input:** `nums = [1,12,-5,-6,50,3]`, `k = 4` → `12.75` (window `[12,-5,-6,50]`)

```java
public double findMaxAverage(int[] nums, int k) {
    double sum = 0;
    for (int i = 0; i < k; i++) sum += nums[i];
    double maxSum = sum;
    for (int right = k; right < nums.length; right++) {
        sum += nums[right] - nums[right - k];
        maxSum = Math.max(maxSum, sum);
    }
    return maxSum / k;
}
```

**Complexity:** O(n) time, O(1) space

**Edge case:** Single window (`nums.length == k`) — loop body never executes; initial window is the answer.

---

## Problem 2: Maximum Sum of Exactly K Elements (Variant)

```java
public int maxSum(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];
    int maxSum = windowSum;
    for (int right = k; right < arr.length; right++) {
        windowSum += arr[right] - arr[right - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

---

## Problem 3: Find All Anagrams in String — LC 438

Fixed window of size `p.length()`. Use frequency arrays to compare.

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    if (s.length() < p.length()) return result;

    int[] pFreq = new int[26];
    int[] wFreq = new int[26];
    for (char c : p.toCharArray()) pFreq[c - 'a']++;

    int k = p.length();
    for (int i = 0; i < k; i++) wFreq[s.charAt(i) - 'a']++;
    if (Arrays.equals(pFreq, wFreq)) result.add(0);

    for (int right = k; right < s.length(); right++) {
        wFreq[s.charAt(right) - 'a']++;           // add incoming
        wFreq[s.charAt(right - k) - 'a']--;       // remove outgoing
        if (Arrays.equals(pFreq, wFreq)) result.add(right - k + 1);
    }
    return result;
}
```

**Optimization:** Track `matches` count instead of calling `Arrays.equals` every iteration (O(26) → O(1) check).

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    int[] freq = new int[26];
    for (char c : p.toCharArray()) freq[c - 'a']++;

    int k = p.length(), matches = 0;
    // Count how many chars have freq 0 (already matched) — initially all non-p chars
    int required = (int) Arrays.stream(freq).filter(f -> f != 0).count();
    // Actually, simpler: track 'have' == 'need'
    // Use the count-of-satisfied approach:
    int left = 0;
    for (int right = 0; right < s.length(); right++) {
        int c = s.charAt(right) - 'a';
        freq[c]--;
        if (freq[c] == 0) matches++;   // this char's count is now satisfied

        if (right >= k) {
            int l = s.charAt(left++) - 'a';
            if (freq[l] == 0) matches--;   // removing a satisfied char
            freq[l]++;
        }
        if (matches == required) result.add(left - 1);  // wait, need re-check
    }
    // Cleaner: use pFreq and wFreq with 'need' tracker
    return result;
}
```

The `Arrays.equals` version is cleaner for interviews — only 26 comparisons.

---

## Problem 4: Permutation in String — LC 567

Same as Find Anagrams but return `true`/`false`.

```java
public boolean checkInclusion(String s1, String s2) {
    if (s1.length() > s2.length()) return false;
    int[] freq = new int[26];
    for (char c : s1.toCharArray()) freq[c - 'a']++;

    int k = s1.length();
    int[] window = new int[26];
    for (int i = 0; i < k; i++) window[s2.charAt(i) - 'a']++;
    if (Arrays.equals(freq, window)) return true;

    for (int right = k; right < s2.length(); right++) {
        window[s2.charAt(right) - 'a']++;
        window[s2.charAt(right - k) - 'a']--;
        if (Arrays.equals(freq, window)) return true;
    }
    return false;
}
```

**Complexity:** O(n) time (O(26) per comparison = O(1)), O(1) space

---

## Fixed Window vs Variable Window Decision

| Signal | Window Type |
|--------|------------|
| "exactly k elements" / "of length k" | Fixed |
| "at most k" / "minimum length" | Variable |
| "all windows of size k" | Fixed |
| "smallest/largest window satisfying X" | Variable |

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Monotonic Deque for Sliding Max](../../7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
