> **Topic:** [Tries](../README.md) · **Interview Tips**

# Common Mistakes

Ten frequent trie bugs, each shown as a Wrong vs Correct pair with a one-line explanation.

1. **Forgetting `is_end` (a prefix wrongly counted as a complete word).**
   ```rust
   // WRONG
   fn search(w: &str) -> bool { walk(w).is_some() }
   // CORRECT
   fn search(w: &str) -> bool { walk(w).map(|n| n.is_end).unwrap_or(false) }
   ```
   Without `is_end`, `"ap"` reports as a stored word merely because `"apple"` was inserted.

2. **Accessing a None child without a None check.**
   ```rust
   // WRONG
   node = node.children[(c as u8 - b'a') as usize].as_deref().unwrap(); // may panic if None
   // CORRECT
   let idx = (c as u8 - b'a') as usize;
   if node.children[idx].is_none() { return false; }
   node = node.children[idx].as_deref().unwrap();
   ```
   Descending into a missing child causes a panic / undefined behavior.

3. **Not marking cells visited in Word Search II.**
   ```rust
   // WRONG
   dfs(board, r + 1, c, node); // same cell can be reused in one path
   // CORRECT
   let t = board[r][c]; board[r][c] = '#';
   dfs(board, r + 1, c, node);
   board[r][c] = t;
   ```
   A cell must not be used twice within the same word path.

4. **Modifying the board without restoring it on backtrack.**
   ```rust
   // WRONG
   board[r][c] = '#';
   dfs(board, r, c, node); // never restored
   // CORRECT
   let t = board[r][c]; board[r][c] = '#';
   dfs(board, r, c, node);
   board[r][c] = t; // restore for sibling paths
   ```
   Failing to restore corrupts exploration from other starting cells.

5. **Wildcard search not trying all children.**
   ```rust
   // WRONG
   if c == '.' { return dfs(&node.children[0], w, i + 1); } // only one branch
   // CORRECT
   if c == '.' {
       for ch in node.children.iter().flatten() {
           if dfs(ch, w, i + 1) { return true; }
       }
   }
   ```
   A `.` must branch into every existing child, not just the first.

6. **XOR trie using wrong bit order (LSB instead of MSB).**
   ```rust
   // WRONG
   for i in 0..=31 { let bit = (num >> i) & 1; /* ... */ }
   // CORRECT
   for i in (0..=31).rev() { let bit = (num >> i) & 1; /* ... */ }
   ```
   Greedy max-XOR must resolve the most significant bit first.

7. **Off-by-one in 31 vs 32 bits in the XOR trie.**
   ```rust
   // WRONG
   for i in (0..=32).rev() { let bit = (num >> i) & 1; } // bit 32 invalid
   // CORRECT
   for i in (0..=31).rev() { let bit = (num >> i) & 1; } // 32 bits: 31..0
   ```
   A 32-bit integer has bits indexed `31` down to `0`; index `32` is out of range.

8. **Sharing a single `TrieNode` reference incorrectly.**
   ```rust
   // WRONG
   // Conceptually equivalent (requires Rc to alias in Rust):
   // all 26 children pointing to the same node merges unrelated branches
   use std::rc::Rc;
   use std::cell::RefCell;
   let shared = Rc::new(RefCell::new(TrieNode::new()));
   for i in 0..26 { children[i] = Some(Rc::clone(&shared)); } // all letters alias one node
   // CORRECT
   children[c as usize - 'a' as usize] = Some(Box::new(TrieNode::new())); // a distinct node per letter
   ```
   Aliasing one child for all letters merges unrelated branches.

9. **`String` overhead in board DFS instead of storing word at node.**
   ```rust
   // WRONG
   sb.push(board[r][c]); /* ... */ if node.is_end { result.push(sb.clone()); }
   // CORRECT
   if let Some(word) = node.word.take() { result.push(word); }
   ```
   Storing the full word at the end node avoids per-step string building.

10. **Not pruning found words (duplicates / wasted work).**
    ```rust
    // WRONG
    if let Some(ref word) = node.word { result.push(word.clone()); } // can add same word twice
    // CORRECT
    if let Some(word) = node.word.take() { result.push(word); }
    ```
    Using `.take()` on `node.word` after collecting it prevents duplicates and trims the trie.

> **Last Updated:** 2026-06-26
