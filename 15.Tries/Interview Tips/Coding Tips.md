> **Topic:** [Tries](../README.md) · **Interview Tips**

# Coding Tips

A focused set of practical tips for writing clean, correct trie code under interview conditions.

1. **Choose `children[26]` vs `unordered_map` based on alphabet size/sparsity.**
   A fixed array is fastest and simplest for a small, dense alphabet (lowercase `a–z`). Switch to an `unordered_map<char, TrieNode*>` when the alphabet is large or the trie is sparse, so you do not waste 26+ references per node.
   ```cpp
   TrieNode* children[26] = {};      // dense lowercase alphabet
   unordered_map<char, TrieNode*> children; // large/sparse alphabet
   ```

2. **Use `c - 'a'` indexing for a lowercase trie.**
   Map each lowercase character directly to an array slot. It is branch-free and avoids the overhead of hashing.
   ```cpp
   int idx = c - 'a';
   if (node->children[idx] == nullptr) node->children[idx] = new TrieNode();
   node = node->children[idx];
   ```

3. **`isEnd` flag discipline — distinguish a stored word from a mere prefix.**
   A node existing on a path does not mean a word ends there. Only set and check `isEnd` to decide membership, otherwise every prefix looks like a complete word.
   ```cpp
   bool isEnd;
   // search returns: node != nullptr && node->isEnd
   ```

4. **Store the full word at the leaf node (`string word`) for board problems.**
   In Word Search II, save the complete word on the terminal node instead of rebuilding a `string` during every DFS step. You get the answer instantly when you reach an end node.
   ```cpp
   struct TrieNode { TrieNode* children[26] = {}; string word; };
   // on insert: node->word = word;  on match: result.push_back(node->word);
   ```

5. **Use DFS for wildcard (`.`) search, branching to all children.**
   When the query character is a wildcard, recurse into every non-null child rather than a single slot.
   ```cpp
   if (c == '.') {
       for (auto* child : node->children)
           if (child != nullptr && dfs(child, word, i + 1)) return true;
       return false;
   }
   ```

6. **Use MSB-first bit order for XOR/binary tries.**
   Greedy maximum-XOR depends on resolving the most significant bit first, since it dominates the value. Iterate from the high bit down to bit 0.
   ```cpp
   for (int i = 31; i >= 0; i--) {
       int bit = (num >> i) & 1;
       // descend toward the opposite bit when available to maximize XOR
   }
   ```

7. **Prune the trie during board DFS.**
   Once a word is found, clear `node->word` so it cannot be added twice, and return early when a child slot is null. This avoids duplicate results and dead-end exploration.
   ```cpp
   if (!node->word.empty()) { result.push_back(node->word); node->word = ""; }
   if (node->children[board[r][c] - 'a'] == nullptr) return; // dead branch
   ```

8. **Reuse a single trie across many queries instead of rebuilding.**
   Building a trie is O(total chars); amortize that cost by constructing it once and answering all `search`/`startsWith` queries against the same structure.
   ```cpp
   Trie trie;
   for (auto& w : words) trie.insert(w);   // build once
   for (auto& q : queries) trie.startsWith(q); // reuse many times
   ```

> **Last Updated:** 2026-06-26
