# Coding Tips — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `interview-tips` `coding` `arrays` `java`

---

## Table of Contents

1. [Before You Code](#before-you-code)
2. [Java-Specific Tips](#java-specific-tips)
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

## Java-Specific Tips

### Sorting

```java
Arrays.sort(arr);                                  // primitives: O(n log n) dual-pivot quicksort
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);    // sort by first element
Arrays.sort(arr, Collections.reverseOrder());      // won't work on int[] — use Integer[]
Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new); // convert for reverse sort
```

### Copy & Fill

```java
int[] copy = Arrays.copyOf(arr, arr.length);       // full copy
int[] sub  = Arrays.copyOfRange(arr, l, r + 1);   // [l, r] inclusive
Arrays.fill(arr, 0);                               // fill entire array
Arrays.fill(arr, l, r + 1, -1);                   // fill [l, r]
```

### Two-Pointer Swap

```java
private void swap(int[] arr, int i, int j) {
    int tmp = arr[i]; arr[i] = arr[j]; arr[j] = tmp;
}
```

### Common Initializations

```java
int max = Integer.MIN_VALUE;  // not 0! (handles all-negative arrays)
int min = Integer.MAX_VALUE;
int sum = 0;                  // use long if values can overflow
```

### 2D Array

```java
int[][] matrix = new int[m][n];
int rows = matrix.length;
int cols = matrix[0].length;
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

```java
// Rotate right by k: [1,2,3,4,5], k=2 → [4,5,1,2,3]
k %= n;
reverse(arr, 0, n-1);     // [5,4,3,2,1]
reverse(arr, 0, k-1);     // [4,5,3,2,1]
reverse(arr, k, n-1);     // [4,5,1,2,3]
```

### In-Place Matrix Transpose

```java
// Transpose square matrix: arr[i][j] ↔ arr[j][i]
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++) {
        int tmp = matrix[i][j];
        matrix[i][j] = matrix[j][i];
        matrix[j][i] = tmp;
    }
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
□ Integer overflow possible? (use long)
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
