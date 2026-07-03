# Design: Pattern Matcher (Multi-Pattern Search)

> **Topic:** [Strings](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `design` `aho-corasick` `multi-pattern` `trie`

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

Design a system that, given a set of `k` patterns, can find **all occurrences of any pattern** in a text string efficiently.

- `PatternMatcher(string[] patterns)` — preprocess patterns
- `vector<pair<int,int>> search(string text)` — return `{start, patternIndex}` for each match

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Know naive approach | Run KMP for each pattern: O(n×k + m×k) |
| Know optimal | Aho-Corasick: build once O(m×k), search O(n + total_matches) |
| Understand failure links | How they mirror KMP's LPS but across a Trie |
| Discuss std::string `find` | Uses Boyer-Moore-Horspool internally, O(n×m) worst |

---

## Approaches

| Approach | Build | Search | Notes |
|----------|-------|--------|-------|
| Run KMP per pattern | O(m×k) | O(n×k) | Simple, good for few patterns |
| Trie + DFS | O(m×k) | O(n × trie_depth) | No failure links |
| Aho-Corasick | O(m×k × Σ) | O(n + matches) | Optimal for multi-pattern |

---

## C++ Implementation — Aho-Corasick

```cpp
#include <bits/stdc++.h>
using namespace std;

struct AhoCorasick {
    struct TrieNode {
        int next[26];
        int fail;                  // failure link
        vector<int> output;       // pattern indices ending here

        TrieNode() {
            fill(next, next + 26, -1);
        }
    };

    vector<TrieNode> trie;
    vector<string> patterns;

    AhoCorasick(vector<string>& pats) : patterns(pats) {
        trie.push_back(TrieNode()); // root

        // Build Trie
        for (int pi = 0; pi < (int)patterns.size(); pi++) {
            int cur = 0;
            for (char c : patterns[pi]) {
                int idx = c - 'a';
                if (trie[cur].next[idx] == -1) {
                    trie[cur].next[idx] = trie.size();
                    trie.push_back(TrieNode());
                }
                cur = trie[cur].next[idx];
            }
            trie[cur].output.push_back(pi);
        }

        // Build failure links (BFS)
        queue<int> q;
        trie[0].fail = 0;
        for (int c = 0; c < 26; c++) {
            int child = trie[0].next[c];
            if (child == -1) {
                trie[0].next[c] = 0; // redirect to root
            } else {
                trie[child].fail = 0;
                q.push(child);
            }
        }
        while (!q.empty()) {
            int u = q.front(); q.pop();
            // Merge output of failure link into current node
            auto& failOut = trie[trie[u].fail].output;
            trie[u].output.insert(trie[u].output.end(), failOut.begin(), failOut.end());
            for (int c = 0; c < 26; c++) {
                int v = trie[u].next[c];
                if (v == -1) {
                    trie[u].next[c] = trie[trie[u].fail].next[c];
                } else {
                    trie[v].fail = trie[trie[u].fail].next[c];
                    q.push(v);
                }
            }
        }
    }

    vector<pair<int,int>> search(const string& text) {
        vector<pair<int,int>> result;
        int cur = 0;
        for (int i = 0; i < (int)text.length(); i++) {
            int c = text[i] - 'a';
            cur = trie[cur].next[c];
            for (int pi : trie[cur].output) {
                int start = i - (int)patterns[pi].length() + 1;
                result.push_back({start, pi});
            }
        }
        return result;
    }
};
// Build: O(total_pattern_len × 26)
// Search: O(n + total_matches)
```

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Build Aho-Corasick | O(Σ × total_pattern_len) | O(Σ × total_pattern_len) |
| Search | O(n + total_matches) | O(n) output |

Where Σ = alphabet size (26 for lowercase).

---

## Edge Cases

- Empty pattern in list → match every position
- Overlapping patterns (`"ab"` and `"abc"`) → both reported independently
- Pattern longer than text → no matches
- Duplicate patterns → report each occurrence multiple times

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Word Search II](https://leetcode.com/problems/word-search-ii/) | LC 212 — Trie + DFS |
| [Stream of Characters](https://leetcode.com/problems/stream-of-characters/) | LC 1032 |
| [Multi Pattern Matching](https://leetcode.com/problems/string-matching-in-an-array/) | LC 1408 |

---

## Follow-up Questions

1. **Aho-Corasick vs running KMP for each pattern?** → AC wins when patterns are many and text is reused; KMP is simpler for a small fixed set.
2. **Case insensitive?** → Lowercase all chars before insertion.
3. **Unicode / large alphabet?** → Use `unordered_map<char, int>` at each node instead of `int[26]`.

---

## Company Tags

`Google` `Amazon` `Palantir` `Competitive Programming`

---

## Navigation

| Related |
|---------|
| [Strings README](../README.md) |
| [KMP and Z-Algorithm](../Patterns/KMP%20and%20Z%20Algorithm.md) |
| [Autocomplete System](./Autocomplete%20System.md) |

> **Last Updated:** 2026-06-26
