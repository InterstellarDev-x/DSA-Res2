# Design: Autocomplete System

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `trie` `hashmap` `autocomplete`
> **LeetCode:** [LC 642](https://leetcode.com/problems/design-search-autocomplete-system/)

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Approaches](#approaches)
4. [C++ Implementation](#c-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design a search autocomplete system. Given a list of historical sentences with their times searched, implement:

- `AutocompleteSystem(string[] sentences, int[] times)` — initialise
- `vector<string> input(char c)` — accepts one character at a time; returns top 3 most searched sentences matching the current prefix. `'#'` ends the current input and saves it.

Top 3 ranked by: **times searched descending**, then **lexicographic order ascending** for ties.

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core structure | Trie for prefix lookup + std::unordered_map for frequency |
| Ranking | Custom comparator: `[&](const string& a, const string& b) { int fa = freq[a], fb = freq[b]; return fa != fb ? fb > fa : a < b; }` |
| Live typing | Accumulate typed characters into `curInput` |
| Save on `#` | Insert `curInput` into trie with updated frequency |
| Trade-off discussion | Trie (fast prefix) vs std::unordered_map (simple but O(n) scan on each char) |

---

## Approaches

| Approach | `input` Time | Space | Notes |
|----------|-------------|-------|-------|
| std::unordered_map + scan | O(n×k) per char | O(n×k) | Scan all sentences for prefix match |
| Trie + DFS | O(p + r) per char | O(n×k) | p = prefix len, r = results |
| Trie + top-k at each node | O(1) per char | O(n×k×3) | Pre-cache top 3 at each node |

---

## C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

class AutocompleteSystem {
    struct TrieNode {
        unordered_map<char, TrieNode*> children;
        // Could pre-cache top-k here for O(1) lookup
    };

    TrieNode* root = new TrieNode();
    unordered_map<string, int> freq;
    TrieNode* cur;
    string inputStr;

    void insert(const string& s) {
        TrieNode* node = root;
        for (char c : s) {
            if (!node->children.count(c))
                node->children[c] = new TrieNode();
            node = node->children[c];
        }
    }

    void dfs(TrieNode* node, string& sb, vector<string>& results) {
        if (freq.count(sb)) results.push_back(sb);
        for (auto& [c, child] : node->children) {
            sb += c;
            dfs(child, sb, results);
            sb.pop_back();
        }
    }

public:
    AutocompleteSystem(vector<string>& sentences, vector<int>& times) {
        for (int i = 0; i < (int)sentences.size(); i++) {
            freq[sentences[i]] = times[i];
            insert(sentences[i]);
        }
        cur = root;
    }

    vector<string> input(char c) {
        if (c == '#') {
            freq[inputStr]++;
            insert(inputStr);
            inputStr.clear();
            cur = root;
            return {};
        }

        inputStr += c;
        if (cur != nullptr) {
            auto it = cur->children.find(c);
            cur = (it != cur->children.end()) ? it->second : nullptr;
        }
        if (cur == nullptr) return {};

        // Collect all sentences under cur node via DFS
        vector<string> candidates;
        string prefix = inputStr;
        dfs(cur, prefix, candidates);

        // Sort: freq desc, then lex asc
        sort(candidates.begin(), candidates.end(), [&](const string& a, const string& b) {
            int fa = freq.count(a) ? freq[a] : 0;
            int fb = freq.count(b) ? freq[b] : 0;
            return fa != fb ? fb > fa : a < b;
        });
        if (candidates.size() > 3) candidates.resize(3);
        return candidates;
    }
};
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Constructor | O(n × k) | O(n × k) |
| `input(c)` — normal char | O(p + r log r) | O(r) — results list |
| `input('#')` | O(k) | O(1) |

Where: `n` = sentences, `k` = avg length, `p` = current prefix length, `r` = matching results.

---

## Edge Cases

- Typing `#` as first character → save empty string (handle gracefully)
- Same sentence typed multiple times → frequency increments correctly
- No matching sentences → return empty list
- Prefix matches sentence exactly → include it in results
- Tie in frequency → lexicographic order (test with identical-freq sentences)

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Implement Trie](https://leetcode.com/problems/implement-trie-prefix-tree/) | LC 208 |
| [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/) | LC 1268 |
| [Word Search II](https://leetcode.com/problems/word-search-ii/) | LC 212 |

---

## Follow-up Questions

1. **Top 3 cached at each Trie node — how?** → Store a `priority_queue<string>` (size 3) at each node; update on insert. `input()` becomes O(1) per char.
2. **Thread-safe for multiple users?** → Use `std::mutex` with `std::lock_guard` around the Trie and `freq` map; or use a per-prefix-subtree lock for finer granularity.
3. **Disk-based for 10 billion sentences?** → LSM-tree or prefix-sorted file with binary search; cache hot prefixes in memory.
4. **Handle typos / fuzzy matching?** → BK-tree or Levenshtein automaton instead of Trie.

---

## Company Tags

`Google` `Amazon` `Microsoft` `Uber` `Airbnb`

---

## Navigation

| Related |
|---------|
| [Strings README](../README.md) |
| [KMP and Z-Algorithm](../Patterns/KMP%20and%20Z%20Algorithm.md) |
| [Pattern Matcher](./Pattern%20Matcher.md) |

> **Last Updated:** 2026-06-26
