# Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Pattern 4 of 4**
> **Problems:** Container with Most Water · 3Sum · Move Zeroes · Sort Colors · Remove Duplicates · Trapping Rain Water (see Stacks topic)

---

## Core Concept

Two-pointer problems use two indices that move toward each other or in the same direction to reduce O(n²) brute force to O(n).

**Two variants:**

| Variant | Pointer Direction | Typical Use |
|---------|-----------------|-------------|
| **Opposite ends** | `left` starts at 0, `right` at n-1, move toward each other | Sum problems, max area, palindrome |
| **Same direction** | Both start at 0, `fast` moves ahead of `slow` | Remove elements, partition, detect cycle |

---

## Template 1: Opposite Ends

```cpp
#include <bits/stdc++.h>
using namespace std;

int left = 0, right = (int)nums.size() - 1;
while (left < right) {
    if (condition) {
        // process or record answer
        left++;
    } else {
        right--;
    }
}
```

---

## Template 2: Same Direction (Fast/Slow)

```cpp
#include <bits/stdc++.h>
using namespace std;

int slow = 0;
for (int fast = 0; fast < (int)nums.size(); fast++) {
    if (shouldKeep(nums[fast])) {
        nums[slow++] = nums[fast];
    }
}
// slow = new length
```

---

## Problem 1: Container With Most Water — LC 11

Find two lines forming a container that holds the most water.

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxArea(vector<int>& height) {
    int left = 0, right = (int)height.size() - 1;
    int maxWater = 0;

    while (left < right) {
        int water = min(height[left], height[right]) * (right - left);
        maxWater = max(maxWater, water);

        // Move the shorter line — moving the taller one can only decrease width
        // and can't increase the bounded height (still limited by the shorter)
        if (height[left] <= height[right]) left++;
        else right--;
    }
    return maxWater;
}
```

**Proof of greedy choice:** When `height[left] ≤ height[right]`, the current container is bounded by `height[left]`. Moving `right` inward decreases width AND keeps the bound at `height[left]` or worse. We can only potentially improve by moving `left` to find a taller left wall.

**Complexity:** O(n) time, O(1) space

---

## Problem 2: 3Sum — LC 15

Find all unique triplets summing to zero.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());   // sort first!
    vector<vector<int>> result;

    for (int i = 0; i < (int)nums.size() - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;  // skip duplicate i

        int left = i + 1, right = (int)nums.size() - 1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                result.push_back({nums[i], nums[left], nums[right]});
                while (left < right && nums[left] == nums[left+1]) left++;   // skip dup left
                while (left < right && nums[right] == nums[right-1]) right--; // skip dup right
                left++; right--;
            } else if (sum < 0) {
                left++;
            } else {
                right--;
            }
        }
    }
    return result;
}
```

**Complexity:** O(n²) time — outer loop O(n), inner two-pointer O(n). O(log n) space for sort.

**Duplicate skipping logic:**
- Outer: `i > 0 && nums[i] == nums[i-1]` — skip if same as previous `i` (not `i > 0` alone — we need to allow `i=0`)
- Inner: After finding a valid triplet, skip duplicates for both left and right before advancing

---

## Problem 3: Move Zeroes — LC 283

Move all zeroes to the end while maintaining order of non-zero elements.

```cpp
#include <bits/stdc++.h>
using namespace std;

void moveZeroes(vector<int>& nums) {
    int slow = 0;   // position to write next non-zero
    for (int fast = 0; fast < (int)nums.size(); fast++) {
        if (nums[fast] != 0) {
            nums[slow++] = nums[fast];
        }
    }
    // Fill rest with zeroes
    while (slow < (int)nums.size()) nums[slow++] = 0;
}
```

**In-place swap variant (fewer writes):**
```cpp
#include <bits/stdc++.h>
using namespace std;

void moveZeroes(vector<int>& nums) {
    int slow = 0;
    for (int fast = 0; fast < (int)nums.size(); fast++) {
        if (nums[fast] != 0) {
            swap(nums[slow], nums[fast]);
            slow++;
        }
    }
}
```

---

## Problem 4: Sort Colors (Dutch National Flag) — LC 75

Sort array of 0s, 1s, 2s in one pass.

```cpp
#include <bits/stdc++.h>
using namespace std;

void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = (int)nums.size() - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low++], nums[mid++]);
        } else if (nums[mid] == 1) {
            mid++;
        } else {  // nums[mid] == 2
            swap(nums[mid], nums[high--]);
            // Don't increment mid — swapped element needs to be examined
        }
    }
}
```

**Three regions:** `[0..low-1]` = 0s, `[low..mid-1]` = 1s, `[mid..high]` = unexamined, `[high+1..n-1]` = 2s.

**Why not `mid++` when swapping with `high`?** The element swapped from `high` to `mid` is unexamined — we don't know if it's a 0, 1, or 2. We must examine it before advancing.

---

## Problem 5: Remove Duplicates from Sorted Array — LC 26

```cpp
#include <bits/stdc++.h>
using namespace std;

int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    int slow = 1;
    for (int fast = 1; fast < (int)nums.size(); fast++) {
        if (nums[fast] != nums[fast - 1]) {
            nums[slow++] = nums[fast];
        }
    }
    return slow;
}
```

**Variant — keep at most 2 duplicates (LC 80):**
```cpp
#include <bits/stdc++.h>
using namespace std;

int removeDuplicates(vector<int>& nums) {
    int slow = 2;
    for (int fast = 2; fast < (int)nums.size(); fast++) {
        if (nums[fast] != nums[slow - 2]) {  // compare with position 2 back
            nums[slow++] = nums[fast];
        }
    }
    return slow;
}
```

---

## Problem 6: 4Sum — LC 18

Extension of 3Sum — add an outer loop.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> fourSum(vector<int>& nums, int target) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    int n = nums.size();

    for (int i = 0; i < n - 3; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;
        for (int j = i + 1; j < n - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j-1]) continue;
            int left = j + 1, right = n - 1;
            while (left < right) {
                long sum = (long)nums[i] + nums[j] + nums[left] + nums[right];
                if (sum == target) {
                    result.push_back({nums[i], nums[j], nums[left], nums[right]});
                    while (left < right && nums[left] == nums[left+1]) left++;
                    while (left < right && nums[right] == nums[right-1]) right--;
                    left++; right--;
                } else if (sum < target) left++;
                else right--;
            }
        }
    }
    return result;
}
```

**Use `long` for sum** — 4 ints can overflow.

---

## Two Pointers vs Sliding Window

| Aspect | Two Pointers | Sliding Window |
|--------|-------------|---------------|
| Pointers | 2 independent | left + right (window boundaries) |
| Movement | Toward each other OR same direction | Always right expands, left shrinks |
| Array requirement | Often sorted (for opposite ends) | Usually unsorted |
| Goal | Find elements satisfying condition | Optimize subarray/substring |
| Count result | Usually pairs/triplets | Usually length or count of windows |

---

## Related Files

- [Variable Size Window](./Variable%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Two Pointers on Linked List](../../4.Linked_List/Patterns/Two%20Pointers%20on%20LL.md)
- [Dutch National Flag (Arrays)](../../1.Arrays/Patterns/Dutch%20National%20Flag.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
