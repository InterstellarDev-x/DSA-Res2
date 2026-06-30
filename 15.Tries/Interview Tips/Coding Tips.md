> **Topic:** [Tries](../README.md) · **Interview Tips**

# Coding Tips

A focused set of practical tips for writing clean, correct trie code under interview conditions.

1. **Choose `children[26]` vs `HashMap` based on alphabet size/sparsity.**
   A fixed array is fastest and simplest for a small, dense alphabet (lowercase `a–z`). Switch to a `HashMap<Character, TrieNode>` when the alphabet is large or the trie is sparse, so you do not waste 26+ references per node.
   ```java
   TrieNode[] children = new TrieNode[26];      // dense lowercase alphabet
   Map<Character, TrieNode> children = new HashMap<>(); // large/sparse alphabet
   ```

2. **Use `c - 'a'` indexing for a lowercase trie.**
   Map each lowercase character directly to an array slot. It is branch-free and avoids the overhead of hashing.
   ```java
   int idx = c - 'a';
   if (node.children[idx] == null) node.children[idx] = new TrieNode();
   node = node.children[idx];
   ```

3. **`isEnd` flag discipline — distinguish a stored word from a mere prefix.**
   A node existing on a path does not mean a word ends there. Only set and check `isEnd` to decide membership, otherwise every prefix looks like a complete word.
   ```java
   boolean isEnd;
   // search returns: node != null && node.isEnd
   ```

4. **Store the full word at the leaf node (`String word`) for board problems.**
   In Word Search II, save the complete word on the terminal node instead of rebuilding a `StringBuilder` during every DFS step. You get the answer instantly when you reach an end node.
   ```java
   class TrieNode { TrieNode[] children = new TrieNode[26]; String word; }
   // on insert: node.word = word;  on match: result.add(node.word);
   ```

5. **Use DFS for wildcard (`.`) search, branching to all children.**
   When the query character is a wildcard, recurse into every non-null child rather than a single slot.
   ```java
   if (c == '.') {
       for (TrieNode child : node.children)
           if (child != null && dfs(child, word, i + 1)) return true;
       return false;
   }
   ```

6. **Use MSB-first bit order for XOR/binary tries.**
   Greedy maximum-XOR depends on resolving the most significant bit first, since it dominates the value. Iterate from the high bit down to bit 0.
   ```java
   for (int i = 31; i >= 0; i--) {
       int bit = (num >> i) & 1;
       // descend toward the opposite bit when available to maximize XOR
   }
   ```

7. **Prune the trie during board DFS.**
   Once a word is found, null out `node.word` so it cannot be added twice, and return early when a child slot is null. This avoids duplicate results and dead-end exploration.
   ```java
   if (node.word != null) { result.add(node.word); node.word = null; }
   if (node.children[board[r][c] - 'a'] == null) return; // dead branch
   ```

8. **Reuse a single trie across many queries instead of rebuilding.**
   Building a trie is O(total chars); amortize that cost by constructing it once and answering all `search`/`startsWith` queries against the same structure.
   ```java
   Trie trie = new Trie();
   for (String w : words) trie.insert(w);   // build once
   for (String q : queries) trie.startsWith(q); // reuse many times
   ```

> **Last Updated:** 2026-06-26
