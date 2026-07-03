# Bit Search & Binary Trie

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Maximum XOR of Two Numbers, XOR queries on array, Prefix XOR with Trie

---

## Core Idea

A **Binary Trie** stores integers bit by bit from the most significant bit (MSB) down. Each node has two children: 0 and 1. Inserting n into the trie takes O(32) time. Querying for the maximum XOR partner of a number x takes O(32) time — at each level, try the opposite bit of x's current bit to maximize XOR.

---

## Binary Trie Structure

```cpp
#include <bits/stdc++.h>
using namespace std;

struct TrieNode {
    TrieNode* children[2] = {nullptr, nullptr}; // children[0] = '0' bit, children[1] = '1' bit
};
```

**Insert n into trie (MSB first, 32 bits):**
```cpp
#include <bits/stdc++.h>
using namespace std;

void insert(TrieNode* root, int n) {
    TrieNode* curr = root;
    for (int bit = 31; bit >= 0; bit--) {
        int b = (n >> bit) & 1;
        if (curr->children[b] == nullptr) curr->children[b] = new TrieNode();
        curr = curr->children[b];
    }
}
```

**Query: find number in trie that maximizes XOR with x:**
```cpp
#include <bits/stdc++.h>
using namespace std;

int maxXOR(TrieNode* root, int x) {
    TrieNode* curr = root;
    int result = 0;
    for (int bit = 31; bit >= 0; bit--) {
        int b = (x >> bit) & 1;
        int want = 1 - b; // prefer opposite bit to maximize XOR
        if (curr->children[want] != nullptr) {
            result |= (1 << bit); // got the XOR bit = 1
            curr = curr->children[want];
        } else {
            curr = curr->children[b]; // forced to take same bit, XOR = 0
        }
    }
    return result;
}
```

---

## Template 1 — Maximum XOR of Two Numbers (LC 421)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximumXOR(vector<int>& nums) {
    TrieNode* root = new TrieNode();
    for (int n : nums) insert(root, n);

    int maxVal = 0;
    for (int n : nums) {
        maxVal = max(maxVal, maxXOR(root, n));
    }
    return maxVal;
}
```

**Time:** O(32n) = O(n). **Space:** O(32n) = O(n) nodes.

---

## Template 2 — Greedy Bit-by-Bit (no Trie, using std::unordered_set)

For Maximum XOR, build the answer greedily from MSB to LSB using an `std::unordered_set` of prefixes:

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximumXOR(vector<int>& nums) {
    int maxVal = 0, mask = 0;
    for (int bit = 31; bit >= 0; bit--) {
        mask |= (1 << bit);
        unordered_set<int> prefixes;
        for (int n : nums) prefixes.insert(n & mask); // prefix up to current bit

        int candidate = maxVal | (1 << bit); // try setting this bit in the answer
        for (int prefix : prefixes) {
            // If candidate ^ prefix is also a prefix, then two numbers XOR to candidate
            if (prefixes.count(candidate ^ prefix)) {
                maxVal = candidate;
                break;
            }
        }
    }
    return maxVal;
}
```

**Time:** O(32n). **Space:** O(n) for `std::unordered_set`.

---

## Template 3 — Maximum XOR with a Number in Range (LC 1707)

Queries: for query `[x, m]`, find max XOR of x with any element ≤ m. Sort queries by m; process in order, inserting elements into trie as their value ≤ current m.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> maximizeXor(vector<int>& nums, vector<vector<int>>& queries) {
    sort(nums.begin(), nums.end());
    int n = queries.size();
    vector<int> idx(n);
    for (int i = 0; i < n; i++) idx[i] = i;
    sort(idx.begin(), idx.end(), [&](int a, int b) { return queries[a][1] < queries[b][1]; }); // sort by m

    vector<int> result(n);
    TrieNode* root = new TrieNode();
    int j = 0;

    for (int i : idx) {
        int x = queries[i][0], m = queries[i][1];
        while (j < (int)nums.size() && nums[j] <= m) {
            insert(root, nums[j++]);
        }
        result[i] = (j == 0) ? -1 : maxXOR(root, x);
    }
    return result;
}
```

---

## Template 4 — Count Pairs with XOR in Range

Count pairs `(i, j)` with `i < j` such that `l ≤ nums[i] XOR nums[j] ≤ r`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int countPairs(vector<int>& nums, int low, int high) {
    return countAtMost(nums, high) - countAtMost(nums, low - 1);
}

// Count pairs with XOR <= limit
int countAtMost(vector<int>& nums, int limit) {
    TrieNode* root = new TrieNode();
    int count = 0;
    for (int n : nums) {
        count += queryAtMost(root, n, limit);
        insert(root, n);
    }
    return count;
}
```

---

## Why MSB-First?

For maximizing XOR, the highest bit contributes `2^31` vs `2^0` — greedy from MSB is correct. A set bit at position 31 is worth more than all lower 31 bits combined.

---

## Trie vs std::unordered_set Approach

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| Binary Trie | O(32n) | O(32n) | When querying multiple times or with range constraints |
| std::unordered_set prefix | O(32n) | O(n) | Single maximum XOR with no range constraint |
| Brute force | O(n²) | O(1) | n ≤ 1000 |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Max XOR Two Numbers (Trie) | O(n) | O(n) |
| Max XOR Two Numbers (std::unordered_set) | O(n) | O(n) |
| Max XOR with Range Query | O((n+q) log n) | O(n) |

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — bit extraction
- [XOR Tricks](./XOR%20Tricks.md) — XOR identities
- [Tries](../../15.Tries/README.md) — character-based Trie (same structure, different alphabet)

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [Bitmask DP](./Bitmask%20DP.md)

> **Last Updated:** 2026-06-26
