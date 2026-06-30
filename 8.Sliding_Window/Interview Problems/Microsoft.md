# Microsoft — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Microsoft

---

## Problem 1: Container With Most Water — Deep Dive

**LC 11** · Medium

```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1, maxWater = 0;
    while (left < right) {
        maxWater = Math.max(maxWater, Math.min(height[left], height[right]) * (right - left));
        if (height[left] <= height[right]) left++;
        else right--;
    }
    return maxWater;
}
```

**Q: Prove we never miss the optimal pair.**
A: Suppose the optimal pair is `(i, j)` with `i < j`. WLOG, `height[i] ≤ height[j]`. When our pointers are at `left = i`, we move left (since `height[left] ≤ height[right]`). This is safe because any pair `(i, r)` for `r < j` (i.e., before we've reached `j`) has width `< j - i` and height `≤ height[i]`, so area `< height[i] × (j-i)`. Moving left is the only way to potentially find a taller left wall.

**Q: What if there are multiple optimal pairs?**
A: The algorithm finds at least one — it doesn't need to enumerate all. For a minimum answer or counting follow-up, a different approach would be needed.

---

## Problem 2: 3Sum — Deep Dive

**LC 15** · Medium

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;
        int left = i + 1, right = nums.length - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                while (left < right && nums[left] == nums[left+1]) left++;
                while (left < right && nums[right] == nums[right-1]) right--;
                left++; right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return result;
}
```

**Microsoft edge case focus:**
- All zeros: `[0,0,0]` → `[[0,0,0]]` — valid, one triplet
- All same positive: `[1,1,1]` → `[]` — no triplet sums to 0
- Empty/one/two elements → return `[]`

**Q: Why `i < nums.length - 2` not `i < nums.length`?**
A: We need at least 2 more elements for `left` and `right`. If `i = n-1`, `left = i+1 = n` which is out of bounds.

---

## Problem 3: Minimum Window Substring — Microsoft Style

**LC 76** · Hard

Microsoft asks this with emphasis on clean code and edge case handling:

```java
public String minWindow(String s, String t) {
    if (s == null || t == null || s.length() < t.length()) return "";

    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    int required = t.length(), have = 0, left = 0;
    int minLen = Integer.MAX_VALUE, minLeft = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (need[c]-- > 0) have++;  // need[c] > 0 before decrement → satisfying need

        while (have == required) {
            if (right - left + 1 < minLen) { minLen = right - left + 1; minLeft = left; }
            if (++need[s.charAt(left)] > 0) have--;  // restore and check
            left++;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
```

**Microsoft interview emphasis:**
- Null checks (` == null`)
- Length check early return
- Clear variable naming (`required`, `have`, `need`)
- Correct `substring(start, start + len)` call

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Two Pointers](../Patterns/Two%20Pointers.md)

> **Last Updated:** 2026-06-26
