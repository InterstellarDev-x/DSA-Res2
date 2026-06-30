> **Topic:** [Dynamic Programming](../README.md) · **Pattern 6 of 8**

# Longest Increasing Subsequence

The **Longest Increasing Subsequence (LIS)** pattern is one of the most reusable templates in dynamic programming. At its core it answers a single question: *for each element, what is the best chain that ends at this element, given some "can-extend" predicate between two elements?* Once you internalize that the predicate (`nums[j] < nums[i]`) is just a plug-in, a whole family of problems collapses into the same `O(n^2)` template — change the predicate (divisibility, string-chain), or bolt on a `count[]` array, and you have solved four more problems for free.

This pattern also has a beautiful `O(n log n)` variant for the canonical problem (patience sorting with a *tails* array), which is worth mastering both for interviews and for the invariant it teaches.

The five problems below progress from the classic LIS, through reconstruction, into three variations that all reuse the same skeleton.

| # | Problem | Source | Key Twist |
|---|---------|--------|-----------|
| 37 | Longest Increasing Subsequence | LC 300 | `O(n^2)` DP **and** `O(n log n)` tails array |
| 38 | Print LIS | GFG | Parent pointers to reconstruct the sequence |
| 39 | Largest Divisible Subset | LC 368 | Predicate becomes `nums[i] % nums[j] == 0` |
| 40 | Longest String Chain | LC 1048 | Predicate is "is a predecessor word"; DP over a HashMap |
| 41 | Number of LIS | LC 673 | Add a `count[]` array alongside `length[]` |

Sibling patterns: [Partition DP (MCM)](./Partition%20DP%20(MCM).md) · [DP on Subsequences](./DP%20on%20Subsequences.md)

---

## 1. The LIS Template

Before the individual problems, here is the mental model that ties them all together.

**State.** Let `dp[i]` = the length (or size, or chain length) of the best valid subsequence that **ends exactly at index `i`**.

**Recurrence.**

```
dp[i] = 1 + max( dp[j] )   over all j < i such that canExtend(j, i)
      = 1                  if no such j exists
```

**Base case.** Every element alone is a valid subsequence of length 1, so `dp[i]` starts at `1`.

**Answer.** The overall best is `max(dp[i])` over all `i` — *not* `dp[n - 1]`, because the best subsequence can end anywhere.

**The only thing that changes between problems** is `canExtend(j, i)`:

| Problem | `canExtend(j, i)` |
|---------|-------------------|
| LIS (LC 300) | `nums[j] < nums[i]` |
| Largest Divisible Subset (LC 368) | `nums[i] % nums[j] == 0` (after sorting) |
| Longest String Chain (LC 1048) | `isPredecessor(word[j], word[i])` (after sorting by length) |
| Number of LIS (LC 673) | `nums[j] < nums[i]`, but also track a `count[]` |

That is the entire insight. Now let's apply it.

---

## 2. Longest Increasing Subsequence (LC 300)

> Given an integer array `nums`, return the length of the longest **strictly increasing** subsequence.

### 2a. The `O(n^2)` DP

**State.** `dp[i]` = length of the longest strictly increasing subsequence ending at index `i`.

**Recurrence.** `dp[i] = 1 + max(dp[j])` for all `j < i` with `nums[j] < nums[i]`; otherwise `dp[i] = 1`.

**Base case.** `Arrays.fill(dp, 1)`.

**Answer.** `max(dp[i])`.

```java
import java.util.Arrays;

class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        if (n == 0) return 0;

        int[] dp = new int[n];
        Arrays.fill(dp, 1);          // every element is an LIS of length 1

        int best = 1;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {      // canExtend predicate
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            best = Math.max(best, dp[i]);
        }
        return best;
    }
}
```

**Complexity**

| Metric | Value |
|--------|-------|
| Time | `O(n^2)` |
| Space | `O(n)` |

#### Dry run (`O(n^2)`)

`nums = [10, 9, 2, 5, 3, 7, 101, 18]`

| i | nums[i] | j's with nums[j] < nums[i] | dp[i] | dp array so far |
|---|---------|----------------------------|-------|-----------------|
| 0 | 10  | (none)            | 1 | [1,1,1,1,1,1,1,1] |
| 1 | 9   | (none, 10 > 9)    | 1 | [1,1,1,1,1,1,1,1] |
| 2 | 2   | (none)            | 1 | [1,1,1,1,1,1,1,1] |
| 3 | 5   | j=2 (2<5)         | 2 | [1,1,1,2,1,1,1,1] |
| 4 | 3   | j=2 (2<3)         | 2 | [1,1,1,2,2,1,1,1] |
| 5 | 7   | j=2,3,4 → max dp 2 | 3 | [1,1,1,2,2,3,1,1] |
| 6 | 101 | all → max dp 3    | 4 | [1,1,1,2,2,3,4,1] |
| 7 | 18  | j=2,3,4,5 → max dp 3 | 4 | [1,1,1,2,2,3,4,4] |

`max(dp) = 4` (one such LIS: `2, 3, 7, 18` or `2, 5, 7, 101`).

### 2b. The `O(n log n)` patience-sorting / tails approach

We maintain an array `tails`, where **`tails[len - 1]` holds the smallest possible tail value of any increasing subsequence of length `len`** seen so far.

**The tails invariant.** `tails` is always sorted in strictly increasing order, and at any point `tails.length` equals the length of the current LIS. For each new number `x`:

- If `x` is larger than every tail, it extends the longest chain → **append** it.
- Otherwise, find the **leftmost** tail `>= x` and **overwrite** it with `x`. This does not change the LIS length, but it keeps the smallest tail for that length, leaving more room for future extensions.

Because `tails` is sorted, "find the leftmost tail `>= x`" is a `lower_bound` binary search.

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] tails = new int[n];   // tails[k] = smallest tail of an increasing subseq of length k+1
        int size = 0;               // current length of the LIS / number of valid tails

        for (int x : nums) {
            int pos = lowerBound(tails, size, x);   // first index with tails[pos] >= x
            tails[pos] = x;                          // overwrite, or append if pos == size
            if (pos == size) {
                size++;
            }
        }
        return size;
    }

    // Returns the first index in [0, size) whose value is >= target.
    // If none, returns size (i.e. target is larger than all current tails).
    private int lowerBound(int[] tails, int size, int target) {
        int lo = 0, hi = size;        // search in [lo, hi)
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] < target) {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        return lo;
    }
}
```

**Using `Arrays.binarySearch` instead of a manual search.** `Arrays.binarySearch(tails, 0, size, x)` returns the index if found, or `-(insertionPoint) - 1` if not. For a *strictly* increasing LIS we want the insertion point of `x`, so:

```java
int idx = Arrays.binarySearch(tails, 0, size, x);
int pos = (idx >= 0) ? idx : -(idx + 1);   // lower-bound style insertion point
```

Note `Arrays.binarySearch` finds an exact match arbitrarily among duplicates, so the manual `lowerBound` is preferred when correctness around equal keys matters. For strictly increasing LIS the difference is moot because we always overwrite at the insertion point and equal values map to the same slot.

**Complexity**

| Metric | Value |
|--------|-------|
| Time | `O(n log n)` |
| Space | `O(n)` |

> Note: the `tails` array is **not** itself a valid LIS — it only tracks lengths. Its final *length* is the answer, but its contents may not be a real subsequence of `nums`. To reconstruct an actual LIS, use the parent-pointer technique from Problem 38.

#### Dry run (`O(n log n)`)

`nums = [10, 9, 2, 5, 3, 7, 101, 18]`

| x | lowerBound pos | action | tails (active prefix) | size |
|---|----------------|--------|------------------------|------|
| 10  | 0 | append      | [10]            | 1 |
| 9   | 0 | overwrite 0 | [9]             | 1 |
| 2   | 0 | overwrite 0 | [2]             | 1 |
| 5   | 1 | append      | [2, 5]          | 2 |
| 3   | 1 | overwrite 1 | [2, 3]          | 2 |
| 7   | 2 | append      | [2, 3, 7]       | 3 |
| 101 | 3 | append      | [2, 3, 7, 101]  | 4 |
| 18  | 3 | overwrite 3 | [2, 3, 7, 18]   | 4 |

Final `size = 4`. Matches the `O(n^2)` result. Notice `[2, 3, 7, 18]` happens to be a real LIS here, but in general the array contents are only a length witness.

---

## 3. Print LIS (GFG)

> Return the actual longest increasing subsequence (any one, often the lexicographically meaningful one), not just its length.

We extend the `O(n^2)` DP with a **parent (hash) pointer** array. `parent[i]` stores the index `j` from which `dp[i]` was extended. After filling `dp`, we find the index of the maximum `dp` value, then walk the parent chain backwards and reverse.

**State.** Same `dp[i]` as Problem 37, plus `parent[i]` = predecessor index (or `i` itself / `-1` if the chain starts here).

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

class Solution {
    public List<Integer> printLIS(int[] nums) {
        int n = nums.length;
        List<Integer> result = new ArrayList<>();
        if (n == 0) return result;

        int[] dp = new int[n];
        int[] parent = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 0; i < n; i++) {
            parent[i] = i;          // self-reference marks a chain start
        }

        int bestLen = 1, bestIdx = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i] && dp[j] + 1 > dp[i]) {
                    dp[i] = dp[j] + 1;
                    parent[i] = j;          // remember where we came from
                }
            }
            if (dp[i] > bestLen) {
                bestLen = dp[i];
                bestIdx = i;
            }
        }

        // Walk the parent chain backwards from bestIdx.
        for (int cur = bestIdx; ; cur = parent[cur]) {
            result.add(nums[cur]);
            if (parent[cur] == cur) break;   // reached a chain start
        }
        Collections.reverse(result);
        return result;
    }
}
```

**Complexity**

| Metric | Value |
|--------|-------|
| Time | `O(n^2)` |
| Space | `O(n)` (`dp` + `parent`) |

#### Dry run

`nums = [10, 9, 2, 5, 3, 7, 101, 18]` → `dp = [1,1,1,2,2,3,4,4]`, with `parent`:

| i | nums[i] | dp[i] | parent[i] |
|---|---------|-------|-----------|
| 0 | 10  | 1 | 0 |
| 1 | 9   | 1 | 1 |
| 2 | 2   | 1 | 2 |
| 3 | 5   | 2 | 2 |
| 4 | 3   | 2 | 2 |
| 5 | 7   | 3 | 3 |
| 6 | 101 | 4 | 5 |
| 7 | 18  | 4 | 5 |

`bestLen = 4`, `bestIdx = 6`. Walk: `6 → parent 5 → parent 3 → parent 2 → parent 2 (start)`.
Collected `[101, 7, 5, 2]`, reversed → **`[2, 5, 7, 101]`**.

---

## 4. Largest Divisible Subset (LC 368)

> Given a set of distinct positive integers, return the largest subset such that every pair `(a, b)` satisfies `a % b == 0` or `b % a == 0`.

**The variation.** This is LIS with two changes:

1. **Sort first.** If `nums` is sorted ascending and `a < b`, then divisibility in one direction is the only one possible: we only need `nums[i] % nums[j] == 0`. Sorting guarantees that a divisibility chain is also an increasing chain, and the pairwise condition reduces to consecutive-in-chain divisibility (divisibility is transitive: if `c % b == 0` and `b % a == 0`, then `c % a == 0`).
2. **Predicate.** `canExtend(j, i)` becomes `nums[i] % nums[j] == 0` instead of `nums[j] < nums[i]`.

Reconstruction uses parent pointers exactly like Problem 38.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

class Solution {
    public List<Integer> largestDivisibleSubset(int[] nums) {
        int n = nums.length;
        List<Integer> result = new ArrayList<>();
        if (n == 0) return result;

        Arrays.sort(nums);                 // ascending; enables single-direction divisibility

        int[] dp = new int[n];
        int[] parent = new int[n];
        Arrays.fill(dp, 1);
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }

        int bestLen = 1, bestIdx = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[i] % nums[j] == 0 && dp[j] + 1 > dp[i]) {   // changed predicate
                    dp[i] = dp[j] + 1;
                    parent[i] = j;
                }
            }
            if (dp[i] > bestLen) {
                bestLen = dp[i];
                bestIdx = i;
            }
        }

        for (int cur = bestIdx; ; cur = parent[cur]) {
            result.add(nums[cur]);
            if (parent[cur] == cur) break;
        }
        Collections.reverse(result);
        return result;
    }
}
```

**Complexity**

| Metric | Value |
|--------|-------|
| Time | `O(n^2)` (plus `O(n log n)` sort) |
| Space | `O(n)` |

#### Dry run

`nums = [1, 2, 4, 8]` (already sorted).

| i | nums[i] | j with nums[i]%nums[j]==0 | dp[i] | parent[i] |
|---|---------|---------------------------|-------|-----------|
| 0 | 1 | (none) | 1 | 0 |
| 1 | 2 | j=0 (2%1==0) | 2 | 0 |
| 2 | 4 | j=0,1 → max dp 2 | 3 | 1 |
| 3 | 8 | j=0,1,2 → max dp 3 | 4 | 2 |

`bestIdx = 3`. Walk `3 → 2 → 1 → 0`, collect `[8, 4, 2, 1]`, reverse → **`[1, 2, 4, 8]`**.

---

## 5. Longest String Chain (LC 1048)

> A word `A` is a **predecessor** of `B` if you can insert exactly one letter into `A` (anywhere) to get `B`. Return the length of the longest possible word chain `w1 → w2 → ... → wk`.

**The variation.** Same template, but:

1. **Sort by length.** A chain only ever goes from a shorter word to a word exactly one longer, so process words in increasing length order — every potential predecessor is seen before the word that extends it.
2. **Predicate via "remove one char."** Instead of comparing all pairs (`O(n^2 * L)`), for each word we generate its `L` possible predecessors (drop one character at a time) and look them up in a `HashMap`. This is the "can extend" predicate, computed by deletion rather than comparison.
3. **DP over a HashMap.** `dp.get(word)` = longest chain ending at `word`.

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int longestStrChain(String[] words) {
        // Sort by ascending length so predecessors are processed first.
        Arrays.sort(words, Comparator.comparingInt(String::length));

        Map<String, Integer> dp = new HashMap<>();   // word -> longest chain ending here
        int best = 1;

        for (String word : words) {
            int curBest = 1;                          // the word by itself
            StringBuilder sb = new StringBuilder(word);
            for (int i = 0; i < word.length(); i++) {
                sb.deleteCharAt(i);                   // candidate predecessor
                String predecessor = sb.toString();
                if (dp.containsKey(predecessor)) {
                    curBest = Math.max(curBest, dp.get(predecessor) + 1);
                }
                sb.insert(i, word.charAt(i));         // restore for next deletion
            }
            dp.put(word, curBest);
            best = Math.max(best, curBest);
        }
        return best;
    }
}
```

**Complexity** — let `n` = number of words, `L` = max word length.

| Metric | Value |
|--------|-------|
| Time | `O(n * L^2)` (for each word, `L` deletions, each building an `O(L)` string) |
| Space | `O(n * L)` (the HashMap of words) |

#### Dry run

`words = ["a", "b", "ba", "bca", "bda", "bdca"]` (already length-sorted).

| word | predecessors checked | dp value | dp map |
|------|----------------------|----------|--------|
| "a"    | (none, len 1)         | 1 | {a:1} |
| "b"    | (none, len 1)         | 1 | {a:1, b:1} |
| "ba"   | "a", "b" → both present, max 1 | 2 | {..., ba:2} |
| "bca"  | "ca", "ba", "bc" → "ba" present (2) | 3 | {..., bca:3} |
| "bda"  | "da", "ba", "bd" → "ba" present (2) | 3 | {..., bda:3} |
| "bdca" | "dca","bca","bda","bdc" → "bca"=3, "bda"=3 | 4 | {..., bdca:4} |

`best = 4` (chain `a → ba → bca → bdca` or `a → ba → bda → bdca`).

---

## 6. Number of LIS (LC 673)

> Return the **number** of longest increasing subsequences in `nums`.

**The variation.** Add a parallel `count[]` array. `length[i]` is the usual LIS length ending at `i`; `count[i]` is how many LISs of that length end at `i`.

**The combine rule** is the subtle part:

- When we find a `j` such that `nums[j] < nums[i]` and `length[j] + 1 > length[i]`: we found a strictly longer chain → **reset**: `length[i] = length[j] + 1`, `count[i] = count[j]`.
- When `length[j] + 1 == length[i]`: we found *another* way to reach the same length → **accumulate**: `count[i] += count[j]`.

Finally, sum `count[i]` over all `i` where `length[i]` equals the global maximum length.

```java
import java.util.Arrays;

class Solution {
    public int findNumberOfLIS(int[] nums) {
        int n = nums.length;
        if (n == 0) return 0;

        int[] length = new int[n];   // length of LIS ending at i
        int[] count  = new int[n];   // number of LIS of that length ending at i
        Arrays.fill(length, 1);
        Arrays.fill(count, 1);

        int maxLen = 1;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    if (length[j] + 1 > length[i]) {
                        length[i] = length[j] + 1;   // found a longer chain: reset
                        count[i] = count[j];
                    } else if (length[j] + 1 == length[i]) {
                        count[i] += count[j];        // same length: accumulate
                    }
                }
            }
            maxLen = Math.max(maxLen, length[i]);
        }

        int total = 0;
        for (int i = 0; i < n; i++) {
            if (length[i] == maxLen) {
                total += count[i];
            }
        }
        return total;
    }
}
```

**Complexity**

| Metric | Value |
|--------|-------|
| Time | `O(n^2)` |
| Space | `O(n)` (`length` + `count`) |

#### Dry run

`nums = [1, 3, 5, 4, 7]`

| i | nums[i] | processing | length[i] | count[i] |
|---|---------|------------|-----------|----------|
| 0 | 1 | start | 1 | 1 |
| 1 | 3 | j=0: 1+1>1 reset | 2 | 1 |
| 2 | 5 | j=0 → len2 cnt1; j=1: 2+1>2 reset | 3 | 1 |
| 3 | 4 | j=0 → len2 cnt1; j=1: 2+1==2? no, 2+1>2 reset to 3 cnt1 (from 3) | 3 | 1 |
| 4 | 7 | j=2(len3)→ 3+1>3 reset len4 cnt1; j=3(len3)→ 3+1==4 accumulate cnt+=1 | 4 | 2 |

`maxLen = 4`. Sum of `count[i]` where `length[i]==4` → only `i=4`, `count=2`.
Answer **2** (the two LISs: `1,3,5,7` and `1,3,4,7`).

> The classic pitfall: using a single counter or `count[i] += count[j]` on the reset branch. You must **replace** the count when the length strictly increases, and **add** only when the length ties.

---

## 7. Recognition Signals

| Signal in the problem statement | Likely LIS variant |
|---------------------------------|--------------------|
| "longest **subsequence**" (not substring) with an order/inequality condition | LIS core (LC 300) |
| "strictly increasing" / "monotonic" chain of values | LIS `O(n^2)` or `O(n log n)` |
| Need the **actual** sequence, not just its length | Add parent pointers (Problem 38) |
| Pairwise **divisibility** of a subset | Largest Divisible Subset (sort + `% == 0` predicate) |
| Build longer **words/strings** by adding/removing one element | Longest String Chain (sort by length + HashMap DP) |
| "**how many** longest ..." / count of optimal subsequences | Number of LIS (`length[]` + `count[]`) |
| `n <= ~2500` and an `O(n^2)` chain DP is fast enough | `O(n^2)` template |
| `n` up to `~10^5`, only the **length** is needed | `O(n log n)` tails / patience sorting |

---

## 8. Summary

The LIS pattern is fundamentally **"best chain ending at `i`, over a pluggable predicate."** Master the three moving parts and every problem here is a recombination:

- **The `O(n^2)` skeleton.** `dp[i] = 1 + max(dp[j])` for valid `j < i`; answer is `max(dp)`. Memorize this; it is the backbone of Problems 37–41.
- **The predicate is the lever.** `nums[j] < nums[i]` → strict increase (37). `nums[i] % nums[j] == 0` after sorting → divisible subset (39). "remove one char to reach a predecessor" after sorting by length → string chain (40). Same loop, different `if`.
- **Reconstruction = parent pointers.** Store where each `dp[i]` came from, walk backwards from the best index, reverse (38, 39).
- **Counting = a parallel `count[]` array.** Reset on a strictly longer chain, accumulate on a tie (41). Getting the reset-vs-accumulate distinction right is the whole problem.
- **The `O(n log n)` upgrade.** Maintain a sorted `tails` array where `tails[k]` is the smallest tail of any increasing subsequence of length `k+1`; binary-search (`lowerBound`) the insertion point of each element. The array length is the LIS length, but its contents are only a length witness — reconstruct with parent pointers if you need the actual sequence.

If you can write the `O(n^2)` template from memory and articulate the tails invariant, you own this entire pattern.

---

> **Last Updated:** 2026-06-26
