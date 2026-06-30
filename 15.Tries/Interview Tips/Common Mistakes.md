> **Topic:** [Tries](../README.md) · **Interview Tips**

# Common Mistakes

Ten frequent trie bugs, each shown as a Wrong vs Correct pair with a one-line explanation.

1. **Forgetting `isEnd` (a prefix wrongly counted as a complete word).**
   ```java
   // WRONG
   boolean search(String w) { return walk(w) != null; }
   // CORRECT
   boolean search(String w) { TrieNode n = walk(w); return n != null && n.isEnd; }
   ```
   Without `isEnd`, `"ap"` reports as a stored word merely because `"apple"` was inserted.

2. **Accessing a null child without a null check.**
   ```java
   // WRONG
   node = node.children[c - 'a'];        // may already be null next iteration
   // CORRECT
   if (node.children[c - 'a'] == null) return false;
   node = node.children[c - 'a'];
   ```
   Descending into a missing child throws `NullPointerException`.

3. **Not marking cells visited in Word Search II.**
   ```java
   // WRONG
   dfs(board, r + 1, c, node); // same cell can be reused in one path
   // CORRECT
   char t = board[r][c]; board[r][c] = '#';
   dfs(board, r + 1, c, node);
   board[r][c] = t;
   ```
   A cell must not be used twice within the same word path.

4. **Modifying the board without restoring it on backtrack.**
   ```java
   // WRONG
   board[r][c] = '#';
   dfs(...); // never restored
   // CORRECT
   char t = board[r][c]; board[r][c] = '#';
   dfs(...);
   board[r][c] = t; // restore for sibling paths
   ```
   Failing to restore corrupts exploration from other starting cells.

5. **Wildcard search not trying all children.**
   ```java
   // WRONG
   if (c == '.') return dfs(node.children[0], w, i + 1); // only one branch
   // CORRECT
   if (c == '.')
       for (TrieNode ch : node.children)
           if (ch != null && dfs(ch, w, i + 1)) return true;
   ```
   A `.` must branch into every existing child, not just the first.

6. **XOR trie using wrong bit order (LSB instead of MSB).**
   ```java
   // WRONG
   for (int i = 0; i <= 31; i++) { int bit = (num >> i) & 1; ... }
   // CORRECT
   for (int i = 31; i >= 0; i--) { int bit = (num >> i) & 1; ... }
   ```
   Greedy max-XOR must resolve the most significant bit first.

7. **Off-by-one in 31 vs 32 bits in the XOR trie.**
   ```java
   // WRONG
   for (int i = 32; i >= 0; i--) { int bit = (num >> i) & 1; } // bit 32 invalid
   // CORRECT
   for (int i = 31; i >= 0; i--) { int bit = (num >> i) & 1; } // 32 bits: 31..0
   ```
   A 32-bit int has bits indexed `31` down to `0`; index `32` is out of range.

8. **Sharing a single `TrieNode` reference incorrectly.**
   ```java
   // WRONG
   TrieNode shared = new TrieNode();
   for (int i = 0; i < 26; i++) children[i] = shared; // all letters alias one node
   // CORRECT
   children[c - 'a'] = new TrieNode(); // a distinct node per letter
   ```
   Aliasing one child for all letters merges unrelated branches.

9. **`StringBuilder` overhead in board DFS instead of storing word at node.**
   ```java
   // WRONG
   sb.append(board[r][c]); ... if (node.isEnd) result.add(sb.toString());
   // CORRECT
   if (node.word != null) { result.add(node.word); node.word = null; }
   ```
   Storing the full word at the end node avoids per-step string building.

10. **Not pruning found words (duplicates / wasted work).**
    ```java
    // WRONG
    if (node.word != null) result.add(node.word); // can add same word twice
    // CORRECT
    if (node.word != null) { result.add(node.word); node.word = null; }
    ```
    Nulling `node.word` after collecting it prevents duplicates and trims the trie.

> **Last Updated:** 2026-06-26
