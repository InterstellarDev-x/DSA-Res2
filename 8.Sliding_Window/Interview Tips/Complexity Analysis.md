# Complexity Analysis — Sliding Window & Two Pointers

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## Pattern-to-Complexity Quick Reference

| Problem / Operation | Time | Space | Notes |
|---------------------|------|-------|-------|
| [Max Average Subarray I](../Patterns/Fixed%20Size%20Window.md) | O(n) | O(1) | Single pass |
| [Find Anagrams / Permutation in String](../Patterns/Fixed%20Size%20Window.md) | O(n) | O(1) | O(26) array comparison |
| [Min Subarray Sum](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | Each element enters/exits once |
| [LSWORC (freq array)](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | 128-element array |
| [LSWORC (HashMap)](../Patterns/Variable%20Size%20Window.md) | O(n) | O(min(m,n)) | m = charset size |
| [Max Cons Ones III](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | |
| [Fruits Into Baskets](../Patterns/Variable%20Size%20Window.md) | O(n) | O(k) | k = distinct fruit types in window (≤ 3) |
| [Char Replacement](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | 26-char array |
| [Min Window Substring](../Patterns/Variable%20Size%20Window.md) | O(\|s\| + \|t\|) | O(1) | 128-element need array |
| [All Three Characters](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | 3-element count array |
| [Longest Subarray After Delete](../Patterns/Variable%20Size%20Window.md) | O(n) | O(1) | |
| [Binary Subarrays With Sum (window)](../Patterns/At%20Most%20K%20Trick.md) | O(n) | O(1) | atMost called twice |
| [Binary Subarrays With Sum (prefix)](../Patterns/At%20Most%20K%20Trick.md) | O(n) | O(n) | HashMap for prefix counts |
| [Nice Subarrays](../Patterns/At%20Most%20K%20Trick.md) | O(n) | O(1) | atMost called twice |
| [Subarray Product < K](../Patterns/At%20Most%20K%20Trick.md) | O(n) | O(1) | Direct atMost |
| [Container With Most Water](../Patterns/Two%20Pointers.md) | O(n) | O(1) | Two pointers from ends |
| [3Sum](../Patterns/Two%20Pointers.md) | O(n²) | O(log n) | Sort + two-pointer per i |
| [4Sum](../Patterns/Two%20Pointers.md) | O(n³) | O(log n) | Sort + two nested loops + two-pointer |
| [Move Zeroes](../Patterns/Two%20Pointers.md) | O(n) | O(1) | Fast/slow pointer |
| [Sort Colors](../Patterns/Two%20Pointers.md) | O(n) | O(1) | Dutch National Flag, one pass |
| [Remove Duplicates](../Patterns/Two%20Pointers.md) | O(n) | O(1) | Slow/fast pointer |

---

## Why Sliding Window Is O(n), Not O(n²)

The nested `while` loop looks like O(n²) but is O(n):

> **Proof:** Each element is added to the window (when `right` passes it) exactly once and removed (when `left` passes it) at most once. Total add operations = n. Total remove operations ≤ n. Every iteration of the inner `while` loop corresponds to one remove. So across all outer loop iterations, the inner loop runs at most n times total.

**Total work = O(n outer iterations + n inner iterations) = O(n).**

---

## Space Complexity Notes

| State Variable | Space | When |
|---------------|-------|------|
| `int` counter | O(1) | sum, zeros, odds |
| `int[26]` char freq | O(1) | uppercase only |
| `int[128]` char freq | O(1) | ASCII |
| `HashMap` | O(k) | at most k distinct values in window |
| Prefix sum array | O(n) | when prefix sum approach used |

"O(1) space" in sliding window means space doesn't grow with input — the window itself is tracked by two pointers, not stored.

---

## kSum Complexity Analysis

| Problem | Time | Why |
|---------|------|-----|
| 2Sum (sorted) | O(n) | Single two-pointer pass |
| 2Sum (unsorted) | O(n log n) | Sort + two-pointer |
| 3Sum | O(n²) | O(n) outer × O(n) inner two-pointer |
| 4Sum | O(n³) | O(n²) outer × O(n) inner two-pointer |
| kSum | O(n^(k-1)) | Recursive reduction: kSum → (k-1)Sum with one pointer |

---

## How to Explain Sliding Window O(n) in Interviews

> "Even though there's a nested loop, each element is processed at most twice — once when the right pointer passes it, and once when the left pointer passes it. So the total number of operations across the entire outer loop is O(2n) = O(n), not O(n²)."

For minimum window substring specifically:
> "We process each character of `s` at most twice — once when added to the window and once when removed. The `t` processing is O(|t|) for the initial need array. Total: O(|s| + |t|)."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Window Template](./Window%20Template.md)

> **Last Updated:** 2026-06-26
