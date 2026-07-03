> **Topic:** [Tries](../README.md) · **Interview Tips**

# Coding Tips

A focused set of practical tips for writing clean, correct trie code under interview conditions.

1. **Choose `children[26]` vs `HashMap` based on alphabet size/sparsity.**
   A fixed array is fastest and simplest for a small, dense alphabet (lowercase `a–z`). Switch to a `HashMap<char, Box<TrieNode>>` when the alphabet is large or the trie is sparse, so you do not waste 26+ references per node.
   ```rust
   children: [Option<Box<TrieNode>>; 26],           // dense lowercase alphabet
   children: HashMap<char, Box<TrieNode>>,          // large/sparse alphabet
   ```

2. **Use `c as u8 - b'a'` indexing for a lowercase trie.**
   Map each lowercase character directly to an array slot. It is branch-free and avoids the overhead of hashing.
   ```rust
   let idx = (c as u8 - b'a') as usize;
   if node.children[idx].is_none() {
       node.children[idx] = Some(Box::new(TrieNode::new()));
   }
   node = node.children[idx].as_mut().unwrap();
   ```

3. **`is_end` flag discipline — distinguish a stored word from a mere prefix.**
   A node existing on a path does not mean a word ends there. Only set and check `is_end` to decide membership, otherwise every prefix looks like a complete word.
   ```rust
   is_end: bool,
   // search returns: node is Some && node.is_end
   ```

4. **Store the full word at the leaf node (`String word`) for board problems.**
   In Word Search II, save the complete word on the terminal node instead of rebuilding a `String` during every DFS step. You get the answer instantly when you reach an end node.
   ```rust
   struct TrieNode { children: [Option<Box<TrieNode>>; 26], word: String }
   // on insert: node.word = word.to_string();  on match: result.push(node.word.clone());
   ```

5. **Use DFS for wildcard (`.`) search, branching to all children.**
   When the query character is a wildcard, recurse into every non-null child rather than a single slot.
   ```rust
   if c == '.' {
       for child in node.children.iter().flatten() {
           if dfs(child, word, i + 1) { return true; }
       }
       return false;
   }
   ```

6. **Use MSB-first bit order for XOR/binary tries.**
   Greedy maximum-XOR depends on resolving the most significant bit first, since it dominates the value. Iterate from the high bit down to bit 0.
   ```rust
   for i in (0..32).rev() {
       let bit = (num >> i) & 1;
       // descend toward the opposite bit when available to maximize XOR
   }
   ```

7. **Prune the trie during board DFS.**
   Once a word is found, clear `node.word` so it cannot be added twice, and return early when a child slot is None. This avoids duplicate results and dead-end exploration.
   ```rust
   if !node.word.is_empty() { result.push(node.word.clone()); node.word.clear(); }
   let idx = (board[r][c] as u8 - b'a') as usize;
   if node.children[idx].is_none() { return; } // dead branch
   ```

8. **Reuse a single trie across many queries instead of rebuilding.**
   Building a trie is O(total chars); amortize that cost by constructing it once and answering all `search`/`starts_with` queries against the same structure.
   ```rust
   let mut trie = Trie::new();
   for w in &words { trie.insert(w); }        // build once
   for q in &queries { trie.starts_with(q); } // reuse many times
   ```

> **Last Updated:** 2026-06-26
