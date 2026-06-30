# Monotonic Stack

> **Topic:** [Stacks & Queues](../README.md) · **Pattern 3 of 5**
> **Problems:** NGE I & II · Daily Temperatures · Online Stock Span · 132 Pattern · Asteroid Collision · Remove K Digits · Trapping Rain Water · Sum of Subarray Minimums · Remove Duplicate Letters · Maximum Width Ramp · Number of Visible People

---

## Core Concept

A **monotonic stack** maintains elements in a strictly increasing or decreasing order at all times. When a new element violates the invariant, pop until invariant holds. The moment of popping is when answers are computed.

**Invariant types:**
- **Decreasing stack:** each new element pops all smaller elements → popped element's "Next Greater Element" is the new element
- **Increasing stack:** each new element pops all larger elements → popped element's "Next Smaller Element" is the new element

---

## Template 1: Next Greater Element (NGE)

```java
// For each element, find the first element to its RIGHT that is greater
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);          // default: no NGE found
    Deque<Integer> stack = new ArrayDeque<>();  // stores INDICES

    for (int i = 0; i < n; i++) {
        // Pop all elements smaller than nums[i] — nums[i] is their NGE
        while (!stack.isEmpty() && nums[stack.peek()] < nums[i]) {
            result[stack.pop()] = nums[i];
        }
        stack.push(i);
    }
    return result;
    // Remaining indices in stack have no NGE → stay -1
}
```

**Why store indices?** We need to write the answer back to `result[index]`. Values alone are insufficient.

---

## Problem 1: Next Greater Element I — LC 496

Two arrays: find NGE of each element of `nums1` within `nums2`.

```java
public int[] nextGreaterElement(int[] nums1, int[] nums2) {
    Map<Integer, Integer> nge = new HashMap<>();
    Deque<Integer> stack = new ArrayDeque<>();

    for (int val : nums2) {
        while (!stack.isEmpty() && stack.peek() < val) {
            nge.put(stack.pop(), val);   // store value→NGE mapping
        }
        stack.push(val);
    }

    int[] result = new int[nums1.length];
    for (int i = 0; i < nums1.length; i++) {
        result[i] = nge.getOrDefault(nums1[i], -1);
    }
    return result;
}
```

**Complexity:** O(n + m) time, O(n) space

---

## Problem 2: Daily Temperatures — LC 739

Find, for each day, how many days until a warmer temperature.

```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Deque<Integer> stack = new ArrayDeque<>();  // indices

    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && temperatures[stack.peek()] < temperatures[i]) {
            int prevIdx = stack.pop();
            result[prevIdx] = i - prevIdx;   // days to wait = index difference
        }
        stack.push(i);
    }
    return result;
}
```

**Key insight:** `i - prevIdx` gives the distance (days to wait). This only works because we store indices, not temperatures.

---

## Problem 3: Next Greater Element II (Circular) — LC 503

Array is circular — wrap around once.

```java
public int[] nextGreaterElements(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    Arrays.fill(result, -1);
    Deque<Integer> stack = new ArrayDeque<>();

    // Loop twice to simulate circular array
    for (int i = 0; i < 2 * n; i++) {
        int idx = i % n;
        while (!stack.isEmpty() && nums[stack.peek()] < nums[idx]) {
            result[stack.pop()] = nums[idx];
        }
        if (i < n) stack.push(idx);   // only push indices from first pass
    }
    return result;
}
```

**Why `if (i < n)` for push?** During the second pass, we only need to resolve remaining stack elements — we don't push new indices again (they're already in result or will be resolved).

---

## Problem 4: Online Stock Span — LC 901

Design: `next(price)` returns the number of consecutive days (including today) for which today's price ≥ the price on those days.

```java
class StockSpanner {
    // Stack stores (price, span) pairs
    private Deque<int[]> stack = new ArrayDeque<>();

    public int next(int price) {
        int span = 1;
        // Absorb all previous days where price was ≤ current price
        while (!stack.isEmpty() && stack.peek()[0] <= price) {
            span += stack.pop()[1];
        }
        stack.push(new int[]{price, span});
        return span;
    }
}
```

**Key trick:** Store cumulative span. When absorbing a previous day, we don't need to re-examine what it absorbed — its stored span already accounts for those days. This achieves O(1) amortized.

**Trace:** prices = [100, 80, 60, 70, 60, 75, 85]
```
100: stack=[(100,1)], span=1
80:  stack=[(100,1),(80,1)], span=1
60:  stack=[(100,1),(80,1),(60,1)], span=1
70:  pop (60,1)→span=1+1=2. stack=[(100,1),(80,1),(70,2)], span=2
60:  stack=[(100,1),(80,1),(70,2),(60,1)], span=1
75:  pop (60,1)→3, pop (70,2)→5. stack=[(100,1),(80,1),(75,5)], span=5
85:  pop (75,5)→6, pop (80,1)→7. stack=[(100,1),(85,7)], span=7
```

---

## Problem 5: 132 Pattern — LC 456

Find i < j < k such that `nums[i] < nums[k] < nums[j]`.

**Approach:** Traverse from right to left with a decreasing monotonic stack.
- `third` = the best candidate for `nums[k]` (largest value seen below a "mountain top")
- Stack maintains potential `nums[j]` values (decreasing)

```java
public boolean find132pattern(int[] nums) {
    int n = nums.length;
    int third = Integer.MIN_VALUE;       // best candidate for nums[k]
    Deque<Integer> stack = new ArrayDeque<>();  // candidates for nums[j]

    for (int i = n - 1; i >= 0; i--) {
        if (nums[i] < third) return true;    // nums[i] is nums[1] (smallest)
        while (!stack.isEmpty() && stack.peek() < nums[i]) {
            third = stack.pop();   // this is nums[k] — it was below nums[j]=nums[i]
        }
        stack.push(nums[i]);
    }
    return false;
}
```

**Why right-to-left?** We want to find a smaller element to the left of a "valley above the minimum." Going right-to-left, whenever we see a larger element than the stack top, the popped element becomes the best `k` candidate.

---

## Problem 6: Asteroid Collision — LC 735

Asteroids: positive = moving right, negative = moving left. Collision: right-moving hits left-moving. Larger absolute value survives; equal sizes both explode.

```java
public int[] asteroidCollision(int[] asteroids) {
    Deque<Integer> stack = new ArrayDeque<>();
    for (int a : asteroids) {
        boolean exploded = false;
        while (!stack.isEmpty() && a < 0 && stack.peek() > 0) {
            if (stack.peek() < -a) {
                stack.pop();   // right-moving asteroid explodes
                continue;
            } else if (stack.peek() == -a) {
                stack.pop();   // both explode
            }
            // stack.peek() > -a: new asteroid explodes
            exploded = true;
            break;
        }
        if (!exploded) stack.push(a);
    }

    // Convert stack to array (stack is in reverse order)
    int[] result = new int[stack.size()];
    for (int i = result.length - 1; i >= 0; i--) result[i] = stack.pop();
    return result;
}
```

**Collision only when:** current asteroid is negative (`a < 0`) AND stack top is positive (`stack.peek() > 0`). Same direction = no collision.

---

## Problem 7: Remove K Digits — LC 402

Remove `k` digits from `num` string to get the smallest possible number.

**Greedy with monotonic stack:** maintain an increasing stack. Whenever we see a digit smaller than the top, pop (that digit is a larger digit we should remove).

```java
public String removeKdigits(String num, int k) {
    Deque<Character> stack = new ArrayDeque<>();
    for (char c : num.toCharArray()) {
        while (k > 0 && !stack.isEmpty() && stack.peek() > c) {
            stack.pop();
            k--;
        }
        stack.push(c);
    }
    // If k > 0, remove from the end (digits are already in increasing order)
    while (k-- > 0) stack.pop();

    // Build result, skipping leading zeros
    StringBuilder sb = new StringBuilder();
    boolean leadingZero = true;
    // Stack has most recent on top — need to iterate in insertion order
    // Use iteration from bottom of stack
    Deque<Character> temp = new ArrayDeque<>();
    while (!stack.isEmpty()) temp.push(stack.pop());  // reverse
    while (!temp.isEmpty()) {
        char c = temp.pop();
        if (leadingZero && c == '0') continue;
        leadingZero = false;
        sb.append(c);
    }
    return sb.length() == 0 ? "0" : sb.toString();
}
```

**Complexity:** O(n) time, O(n) space

---

## Problem 8: Trapping Rain Water — LC 42

**Input:** `[0,1,0,2,1,0,1,3,2,1,2,1]` → `6`

### Monotonic Stack Approach — O(n) time, O(n) space

```java
public int trap(int[] height) {
    Deque<Integer> stack = new ArrayDeque<>();  // decreasing stack of indices
    int water = 0;

    for (int i = 0; i < height.length; i++) {
        while (!stack.isEmpty() && height[stack.peek()] < height[i]) {
            int bottom = stack.pop();
            if (stack.isEmpty()) break;
            int left = stack.peek();
            int width = i - left - 1;
            int boundedHeight = Math.min(height[left], height[i]) - height[bottom];
            water += width * boundedHeight;
        }
        stack.push(i);
    }
    return water;
}
```

**Layer-by-layer concept:** The stack maintains a valley shape. When a right wall (taller element) is found, we compute the water trapped between the left wall (new stack top after pop) and right wall at the height of the just-popped element.

### Two-Pointer Approach — O(n) time, O(1) space

```java
public int trap(int[] height) {
    int left = 0, right = height.length - 1;
    int maxLeft = 0, maxRight = 0, water = 0;
    while (left < right) {
        if (height[left] <= height[right]) {
            if (height[left] >= maxLeft) maxLeft = height[left];
            else water += maxLeft - height[left];
            left++;
        } else {
            if (height[right] >= maxRight) maxRight = height[right];
            else water += maxRight - height[right];
            right--;
        }
    }
    return water;
}
```

**Which to use in interview?** Two-pointer is O(1) space and simpler to code. Monotonic stack approach generalizes more easily to follow-ups.

---

## Problem 9: Sum of Subarray Minimums — LC 907

For each subarray, find its minimum and sum all minimums. Answer modulo 10^9 + 7.

**Key insight:** For each element `A[i]`, count how many subarrays have `A[i]` as their minimum. Use monotonic increasing stack to find:
- `left[i]` = number of subarrays ending at `i` where `A[i]` is minimum (distance to previous smaller element)
- `right[i]` = number of subarrays starting at `i` where `A[i]` is minimum (distance to next smaller or equal element)

```java
public int sumSubarrayMins(int[] arr) {
    int MOD = 1_000_000_007;
    int n = arr.length;
    int[] left = new int[n];    // left[i] = # subarrays where arr[i] is min, ending at i
    int[] right = new int[n];   // right[i] = # subarrays where arr[i] is min, starting at i
    Deque<Integer> stack = new ArrayDeque<>();

    // Compute left[i]: distance to previous STRICTLY smaller element
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) stack.pop();
        left[i] = stack.isEmpty() ? i + 1 : i - stack.peek();
        stack.push(i);
    }
    stack.clear();
    // Compute right[i]: distance to next SMALLER OR EQUAL element
    // Use <= for right to avoid double-counting equal elements
    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && arr[stack.peek()] > arr[i]) stack.pop();
        right[i] = stack.isEmpty() ? n - i : stack.peek() - i;
        stack.push(i);
    }

    long ans = 0;
    for (int i = 0; i < n; i++) {
        ans = (ans + (long) arr[i] * left[i] * right[i]) % MOD;
    }
    return (int) ans;
}
```

**Why strict `>=` for left but `>` for right?** Prevents double-counting subarrays with equal minimums. The convention: equal elements are counted on the left side.

---

## Problem 10: Remove Duplicate Letters — LC 316

Return the smallest lexicographic subsequence containing all unique characters exactly once.

```java
public String removeDuplicateLetters(String s) {
    int[] count = new int[26];
    boolean[] inStack = new boolean[26];
    for (char c : s.toCharArray()) count[c - 'a']++;

    Deque<Character> stack = new ArrayDeque<>();
    for (char c : s.toCharArray()) {
        count[c - 'a']--;
        if (inStack[c - 'a']) continue;    // already in result
        // Pop if current char is smaller AND the top char appears later
        while (!stack.isEmpty() && stack.peek() > c && count[stack.peek() - 'a'] > 0) {
            inStack[stack.pop() - 'a'] = false;
        }
        stack.push(c);
        inStack[c - 'a'] = true;
    }
    StringBuilder sb = new StringBuilder();
    Deque<Character> tmp = new ArrayDeque<>();
    while (!stack.isEmpty()) tmp.push(stack.pop());
    while (!tmp.isEmpty()) sb.append(tmp.pop());
    return sb.toString();
}
```

**Three conditions to pop:** (1) stack not empty, (2) top > current (can improve lexicographic order), (3) top's count > 0 (top will appear again later, so safe to remove now).

---

## Monotonic Stack — Pattern Summary

| Problem Type | Stack Order | Pop When | Answer At |
|-------------|-------------|----------|-----------|
| Next Greater Element | Decreasing | new > top | pop time |
| Next Smaller Element | Increasing | new < top | pop time |
| Stock Span (prev consecutive ≤) | Decreasing | new > top | pop time (accumulate span) |
| 132 Pattern | Decreasing | new > top | maintains `third` |
| Trapping Rain Water | Decreasing | new > top | compute layer on pop |
| Remove K Digits | Increasing | new < top AND k>0 | build from stack |
| Sum of Subarray Mins | Increasing | new < top | left/right distances |
| Remove Duplicate Letters | Stack (lexicographic) | top > new AND will recur | build from stack |

---

## Related Files

- [Stack Basics](./Stack%20Basics.md)
- [Histogram & Rectangle](./Histogram%20and%20Rectangle.md)
- [Queue & Deque](./Queue%20and%20Deque.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
