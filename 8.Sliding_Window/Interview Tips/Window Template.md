# Window Template Guide

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## The Universal Sliding Window Template

```java
// Step 1: Choose state representation
//   - int counter for simple counts (zeros, odds)
//   - int[] freq array for character frequencies
//   - Map<K,V> for arbitrary key frequencies/counts

// Step 2: Choose goal (maximize or minimize window)
//   - Maximize: use 'if' for shrink, update answer after shrink
//   - Minimize: use 'while' for shrink, update answer before shrink

// Step 3: Choose what to track
//   - Window size (right - left + 1)
//   - Count of valid windows (result += right - left + 1)

int left = 0;
// state variables here

for (int right = 0; right < n; right++) {
    // [EXPAND] Add nums[right] to window state

    // [SHRINK] While/if window is invalid/should shrink
    while (isInvalid()) {  // or 'if' for max-length problems
        // Remove nums[left] from window state
        left++;
    }

    // [RECORD] Update answer
    // For max length: maxLen = Math.max(maxLen, right - left + 1)
    // For min length: minLen = Math.min(minLen, right - left + 1) [inside while]
    // For count: result += right - left + 1
}
```

---

## Template Instantiations

### Template 1: Max Length — At Most k of Property

**Problems:** Max Consecutive Ones III, Fruits Into Baskets, LSWORC

```java
int left = 0, state = 0, maxLen = 0;
for (int right = 0; right < n; right++) {
    state += increase(nums[right]);           // expand
    if (state > k) {                          // violated → if, not while
        state -= decrease(nums[left++]);      // shrink by 1
    }
    maxLen = Math.max(maxLen, right - left + 1);  // window is valid
}
return maxLen;
```

### Template 2: Min Length — At Least Target Sum/Count

**Problems:** Minimum Subarray Sum

```java
int left = 0, state = 0, minLen = Integer.MAX_VALUE;
for (int right = 0; right < n; right++) {
    state += nums[right];                     // expand
    while (state >= target) {                 // valid → while, minimize
        minLen = Math.min(minLen, right - left + 1);
        state -= nums[left++];
    }
}
return minLen == Integer.MAX_VALUE ? 0 : minLen;
```

### Template 3: Min Length — All Chars Present

**Problems:** Minimum Window Substring

```java
int[] need = buildNeedArray(t);
int left = 0, have = 0, required = t.length();
int minLen = Integer.MAX_VALUE, minLeft = 0;

for (int right = 0; right < n; right++) {
    if (need[s.charAt(right)]-- > 0) have++;  // expand with condition
    while (have == required) {                 // valid → while, minimize
        if (right - left + 1 < minLen) { minLen = right - left + 1; minLeft = left; }
        if (++need[s.charAt(left)] > 0) have--;  // shrink with condition
        left++;
    }
}
return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
```

### Template 4: Count Subarrays — At Most k

**Problems:** Subarray Product < K, Binary Subarrays With Sum (combined with subtraction)

```java
int left = 0, state = initialState, result = 0;
for (int right = 0; right < n; right++) {
    state = applyRight(state, nums[right]);    // expand
    while (state > k || isInvalid(state)) {   // shrink until valid
        state = removeLeft(state, nums[left++]);
    }
    result += right - left + 1;               // count all starting positions
}
return result;
```

---

## Decision Flowchart

```
Problem says "subarray/substring"?
    YES →
        Fixed k elements?
            YES → Fixed Window Template
            NO  →
                Count valid subarrays?
                    YES → atMost template + subtraction if "exactly k"
                    NO  →
                        Minimize window?
                            YES → Variable Window, while-shrink, update inside while
                            NO  → Variable Window, if-shrink, update after if
        NO → Two Pointers (if sorted) or different approach
```

---

## State Variable Selection

| Window Property | State Variable | Init | Expand | Shrink | Violated When |
|----------------|---------------|------|--------|--------|---------------|
| Running sum | `int sum` | 0 | `sum += x` | `sum -= x` | `sum >= target` (for minimize) |
| Zero count | `int zeros` | 0 | `if x==0: zeros++` | `if x==0: zeros--` | `zeros > k` |
| Odd count | `int odds` | 0 | `if x%2==1: odds++` | `if x%2==1: odds--` | `odds > k` |
| Running product | `int product` | 1 | `product *= x` | `product /= x` | `product >= k` |
| Distinct count | `Map + size` | empty | `put/merge x` | `remove if 0` | `map.size() > k` |
| Char frequency | `int[] freq` | zeros | `freq[x]++` | `freq[x]--` | `freq[x] > 1` or `freq[x] > allowed` |
| Have/need | `int have` | 0 | conditional `have++` | conditional `have--` | `have < required` |

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
