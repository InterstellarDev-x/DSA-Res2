# Coding Tips — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `interview-tips` `coding` `arrays` `cpp`

---

## Table of Contents

1. [Before You Code](#before-you-code)
2. [C++-Specific Tips](#c-specific-tips)
3. [Array Manipulation Tricks](#array-manipulation-tricks)
4. [Complexity Shortcuts](#complexity-shortcuts)
5. [Checklist Before Submitting](#checklist-before-submitting)
6. [Related Files](#related-files)

---

## Before You Code

1. **Read the full problem** — Don't start until you know input/output format, constraints, and return type.
2. **Ask clarifying questions:**
   - Are values positive/negative/zero?
   - Can the array be empty? Single element?
   - Can values repeat? Are they sorted?
   - What should be returned if no answer exists?
3. **Write examples by hand** — Pick an example, trace through it before coding.
4. **State your approach aloud** — "I'll use prefix sum + hashmap. Here's why..."
5. **State complexity upfront** — "This will be O(n) time and O(n) space."

---

## C++-Specific Tips

### Sorting

```cpp
#include <bits/stdc++.h>
using namespace std;

sort(arr.begin(), arr.end());                                                              // O(n log n) introsort
sort(intervals.begin(), intervals.end(), [](auto& a, auto& b){ return a[0] < b[0]; });   // sort by first element
sort(arr.begin(), arr.end(), greater<int>());                                              // reverse sort — works on vector<int>
```

### Copy & Fill

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> copy(arr);                                                 // full copy
vector<int> sub(arr.begin() + l, arr.begin() + r + 1);               // [l, r] inclusive
fill(arr.begin(), arr.end(), 0);                                       // fill entire array
fill(arr.begin() + l, arr.begin() + r + 1, -1);                       // fill [l, r]
```

### Two-Pointer Swap

```cpp
void swap_arr(vector<int>& arr, int i, int j) {
    swap(arr[i], arr[j]);
}
```

### Common Initializations

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxVal = INT_MIN;  // not 0! (handles all-negative arrays)
int minVal = INT_MAX;
int sum = 0;           // use long long if values can overflow
```

### 2D Array

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> matrix(m, vector<int>(n, 0));
int rows = matrix.size();
int cols = matrix[0].size();
```

---

## Array Manipulation Tricks

| Operation | Code | Time |
|-----------|------|------|
| Reverse array | Two pointer from ends | O(n) |
| Rotate right by k | Three reverses | O(n) |
| Check palindrome | Two pointer | O(n) |
| Move zeros to end | Slow/fast pointer | O(n) |
| Remove element in-place | Slow/fast pointer | O(n) |
| Prefix sum | Single pass, store in extra array | O(n) |
| Suffix max | Right to left pass | O(n) |

### Rotate Array by K (Three Reversal Trick)

```cpp
// Rotate right by k: [1,2,3,4,5], k=2 → [4,5,1,2,3]
k %= n;
reverse(arr.begin(), arr.end());           // [5,4,3,2,1]
reverse(arr.begin(), arr.begin() + k);     // [4,5,3,2,1]
reverse(arr.begin() + k, arr.end());       // [4,5,1,2,3]
```

### In-Place Matrix Transpose

```cpp
// Transpose square matrix: arr[i][j] ↔ arr[j][i]
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++)
        swap(matrix[i][j], matrix[j][i]);
// Then reverse each row for 90° clockwise rotation
```

---

## Complexity Shortcuts

| Pattern | Time | Space |
|---------|------|-------|
| [Prefix Sum](../Patterns/Prefix%20Sum.md) | O(n) | O(n) |
| [Sliding Window](../Patterns/Sliding%20Window.md) | O(n) | O(1) or O(k) |
| [Two Pointers](../Patterns/Two%20Pointers.md) | O(n) or O(n²) for 3Sum | O(1) |
| [Kadane's](../Patterns/Kadane's%20Algorithm.md) | O(n) | O(1) |
| Sort + scan | O(n log n) | O(1) |
| [Cyclic Sort](../Patterns/Cyclic%20Sort.md) | O(n) | O(1) |
| [Dutch National Flag](../Patterns/Dutch%20National%20Flag.md) | O(n) | O(1) |

---

## Checklist Before Submitting

```
□ Empty array handled?
□ Single element array handled?
□ All same elements handled?
□ Negative values handled (if applicable)?
□ Integer overflow possible? (use long long)
□ Index out of bounds impossible?
□ Time complexity stated?
□ Space complexity stated?
□ All return paths correct?
□ Test with 2–3 examples mentally
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Dry Run Technique](./Dry%20Run.md)
- [Communication Tips](./Communication.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
