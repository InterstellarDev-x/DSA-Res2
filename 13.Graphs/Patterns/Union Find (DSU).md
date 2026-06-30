> **Topic:** [Graphs](../README.md) · **Pattern 6 of 7**

# Union Find (DSU)

Union Find — also called Disjoint Set Union (DSU) — is the go-to data structure for **dynamic connectivity** and **grouping** problems. It maintains a collection of disjoint sets and supports two near-instant operations:

- **find(x)** — return the representative ("root") of the set containing `x`.
- **union(x, y)** — merge the two sets containing `x` and `y`.

Two elements are in the same set iff they share the same root. This makes "are these connected?", "how many groups are there?", and "merge these two groups" all answerable in amortized near-constant time.

## Core Concept

Each element points to a parent. Following parents repeatedly leads to the root of the set. The structure starts with every element as its own singleton set (`parent[i] = i`).

Two optimizations make it fast:

1. **Path compression** (in `find`): after locating the root, re-point every node along the path directly to the root. Future lookups become flat.
2. **Union by rank / size**: always attach the smaller (shorter / lighter) tree under the larger one, keeping trees shallow.

With **both** optimizations, the amortized cost per operation is **O(α(n))**, where α is the inverse Ackermann function. For any practical `n` (even far beyond the number of atoms in the universe), α(n) ≤ 4, so each operation is effectively **O(1)**.

| Operation | Without optimization | Path compression only | Compression + union by rank |
|-----------|----------------------|------------------------|-----------------------------|
| `find` | O(n) worst case | O(log n) amortized | **O(α(n)) ≈ O(1)** |
| `union` | O(n) worst case | O(log n) amortized | **O(α(n)) ≈ O(1)** |

When to reach for DSU instead of BFS/DFS:

- Edges/connections arrive **incrementally** (online), and you must answer connectivity as you go.
- You only need to know **grouping / connectivity**, not the actual shortest path.
- You must detect the **first edge that forms a cycle** in an undirected graph.

DSU does **not** support efficient *deletion* of edges (no easy "un-union"). For that, you need different techniques.

## Reusable Template

Use this DSU class throughout. It carries both `rank` (for union by rank) and `size` (handy for problems that need the size of a component), plus a live `count` of disjoint sets.

```java
class UnionFind {
    private final int[] parent;
    private final int[] rank;
    private final int[] size;
    private int count; // number of disjoint sets

    public UnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        size = new int[n];
        count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;
            rank[i] = 0;
            size[i] = 1;
        }
    }

    // find with path compression (iterative, no stack overflow risk)
    public int find(int x) {
        int root = x;
        while (root != parent[root]) {
            root = parent[root];
        }
        while (x != root) {       // compress the path
            int next = parent[x];
            parent[x] = root;
            x = next;
        }
        return root;
    }

    // union by rank; returns true if a real merge happened, false if already connected
    public boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        size[ra] += size[rb];
        if (rank[ra] == rank[rb]) rank[ra]++;
        count--;
        return true;
    }

    public boolean connected(int a, int b) {
        return find(a) == find(b);
    }

    public int size(int x) {
        return size[find(x)];
    }

    public int count() {
        return count;
    }
}
```

> The `union` return value (`true` = merged, `false` = already in same set) is the single most useful signal in DSU problems — it instantly detects cycles and redundant edges.

## Recognition Signals

| Signal in the prompt | Why DSU fits |
|----------------------|--------------|
| "Are these two connected?" asked repeatedly | `connected(a, b)` in O(α) |
| Edges added one at a time; answer changes per addition | Incremental `union`, online queries |
| "Find the edge that creates a cycle" | First `union` returning `false` |
| "How many groups / provinces / islands / components?" | `count()` |
| "Merge accounts / friends / equal things" | Transitive grouping by root |
| "Valid tree?" | n−1 edges **and** every `union` succeeds |
| "Equations a==b, a!=b — consistent?" | Union the `==`, verify the `!=` |
| Grid connectivity where you union by shared row/column | Map labels/cells to DSU indices |

---

## 38. Redundant Connection — Medium (LC684)

**Idea:** A tree with `n` nodes has exactly `n−1` edges; one extra edge creates a cycle. Process edges in order and return the first one whose two endpoints are *already* connected.

```java
class Solution {
    public int[] findRedundantConnection(int[][] edges) {
        int n = edges.length;            // n nodes, n edges (1-indexed)
        UnionFind uf = new UnionFind(n + 1);
        for (int[] e : edges) {
            if (!uf.union(e[0], e[1])) {  // already connected -> cycle edge
                return e;
            }
        }
        return new int[0]; // unreachable per constraints
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

---

## 39. Accounts Merge — Medium (LC721)

**Idea:** Union all emails belonging to the same account (and across accounts that share any email). Then group emails by their DSU root, sort each group, and prepend the owner's name.

```java
import java.util.*;

class Solution {
    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        int n = accounts.size();
        UnionFind uf = new UnionFind(n);

        // Map each email to the index of the first account it appears in.
        Map<String, Integer> emailToAccount = new HashMap<>();
        for (int i = 0; i < n; i++) {
            List<String> acc = accounts.get(i);
            for (int j = 1; j < acc.size(); j++) {
                String email = acc.get(j);
                if (emailToAccount.containsKey(email)) {
                    uf.union(i, emailToAccount.get(email)); // shared email -> same person
                } else {
                    emailToAccount.put(email, i);
                }
            }
        }

        // Collect emails under each account's root.
        Map<Integer, TreeSet<String>> rootToEmails = new HashMap<>();
        for (Map.Entry<String, Integer> e : emailToAccount.entrySet()) {
            int root = uf.find(e.getValue());
            rootToEmails.computeIfAbsent(root, k -> new TreeSet<>()).add(e.getKey());
        }

        // Build result: name + sorted emails.
        List<List<String>> result = new ArrayList<>();
        for (Map.Entry<Integer, TreeSet<String>> e : rootToEmails.entrySet()) {
            String name = accounts.get(e.getKey()).get(0);
            List<String> merged = new ArrayList<>();
            merged.add(name);
            merged.addAll(e.getValue()); // TreeSet keeps them sorted
            result.add(merged);
        }
        return result;
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
        }
    }
}
```

**Dry run.** Suppose:

```
accounts = [
  0: ["John", "a@x.com", "b@x.com"],
  1: ["John", "c@x.com"],
  2: ["John", "a@x.com", "c@x.com"],
  3: ["Mary", "m@x.com"]
]
```

1. **Build `emailToAccount` and union as we go.**
   - Account 0: `a@x.com` -> not seen, map `a@x.com=0`. `b@x.com` -> map `b@x.com=0`.
   - Account 1: `c@x.com` -> map `c@x.com=1`.
   - Account 2: `a@x.com` -> already maps to 0, so `union(2, 0)`. Now roots: parent[2]=0 (say). `c@x.com` -> already maps to 1, so `union(2, 1)`; find(2)=0, find(1)=1, merge -> parent[1]=0. Now accounts {0,1,2} share one root (0).
   - Account 3: `m@x.com` -> map `m@x.com=3`. Stays alone.

2. **Group emails by root via `find`.**
   - `a@x.com` (acct 0) -> root 0
   - `b@x.com` (acct 0) -> root 0
   - `c@x.com` (acct 1) -> find(1)=0 -> root 0
   - `m@x.com` (acct 3) -> root 3
   - `rootToEmails = { 0: {a@x.com, b@x.com, c@x.com}, 3: {m@x.com} }` (TreeSet auto-sorts).

3. **Emit with names.**
   - Root 0 -> name `accounts.get(0).get(0)` = "John" -> `["John","a@x.com","b@x.com","c@x.com"]`.
   - Root 3 -> "Mary" -> `["Mary","m@x.com"]`.

The two "John" accounts that shared `a@x.com`/`c@x.com` collapse into one, while the unrelated "Mary" stays separate. Even though all three Johns happen to share the same name, they merge only because of shared *emails*, not the name.

**Complexity:** Let `N` be the total number of emails. Time O(N·α(N) + N log N) for sorting via TreeSet. Space O(N).

---

## 40. Number of Operations to Make Network Connected — Medium (LC1319)

**Idea:** To connect `c` components you need `c−1` cables. You can only reuse cables you already have, so you must hold at least `n−1` edges total; otherwise it is impossible (return −1). Otherwise the answer is `components − 1`.

```java
class Solution {
    public int makeConnected(int n, int[][] connections) {
        if (connections.length < n - 1) return -1; // not enough cables to ever connect

        UnionFind uf = new UnionFind(n);
        for (int[] c : connections) {
            uf.union(c[0], c[1]);
        }
        return uf.count() - 1; // cables needed = components - 1
    }

    class UnionFind {
        int[] parent, rank;
        int count;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            count = n;
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            count--;
        }
        int count() { return count; }
    }
}
```

**Complexity:** Time O(E·α(n) + n). Space O(n).

---

## 41. Satisfiability of Equality Equations — Medium (LC990)

**Idea:** Two-pass. First union every variable joined by `==`. Then for every `!=`, if the two variables are already in the same set, it is a contradiction.

```java
class Solution {
    public boolean equationsPossible(String[] equations) {
        UnionFind uf = new UnionFind(26); // variables are single lowercase letters

        // Pass 1: union all equalities.
        for (String eq : equations) {
            if (eq.charAt(1) == '=') {
                uf.union(eq.charAt(0) - 'a', eq.charAt(3) - 'a');
            }
        }
        // Pass 2: verify inequalities don't conflict.
        for (String eq : equations) {
            if (eq.charAt(1) == '!') {
                if (uf.connected(eq.charAt(0) - 'a', eq.charAt(3) - 'a')) {
                    return false;
                }
            }
        }
        return true;
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
        }
        boolean connected(int a, int b) { return find(a) == find(b); }
    }
}
```

**Complexity:** Time O(n·α(26)) ≈ O(n) where n = number of equations. Space O(1) (fixed 26 variables).

---

## 42. Most Stones Removed with Same Row or Column — Medium (LC947)

**Idea:** A stone can be removed if it shares a row or column with another stone — i.e., stones in the same connected component can be reduced to one. Union stones by shared row and column; the answer is `total stones − number of components`. To union by row/column cheaply, map columns into a separate index space.

```java
import java.util.*;

class Solution {
    public int removeStones(int[][] stones) {
        int n = stones.length;
        UnionFind uf = new UnionFind(20000); // rows 0..9999, cols 10000..19999

        for (int[] s : stones) {
            int row = s[0];
            int col = s[1] + 10000; // shift columns into disjoint index range
            uf.union(row, col);
        }

        // Count distinct roots among the indices actually used.
        Set<Integer> roots = new HashSet<>();
        for (int[] s : stones) {
            roots.add(uf.find(s[0]));
        }
        return n - roots.size(); // stones - components
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
        }
    }
}
```

> **Why count roots from the row index only?** Each stone unions its row with its column, so a stone's row index already belongs to the same component as its column. Collecting `find(row)` over all stones therefore yields exactly the distinct components.

**Complexity:** Time O(n·α). Space O(R + C) for the index space (≈ 20000).

---

## 43. Number of Islands II — Hard (LC305)

**Idea:** Process land additions online. Start with 0 islands. Each `addLand(r, c)`: if the cell is new land, increment the count, then union it with any of its 4 neighbors that are already land — each successful merge of two distinct islands decrements the count. Record the count after every operation.

```java
import java.util.*;

class Solution {
    public List<Integer> numIslands2(int m, int n, int[][] positions) {
        UnionFind uf = new UnionFind(m * n);
        boolean[] isLand = new boolean[m * n];
        List<Integer> result = new ArrayList<>();
        int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};
        int count = 0;

        for (int[] p : positions) {
            int r = p[0], c = p[1];
            int id = r * n + c;
            if (isLand[id]) {                 // duplicate add -> count unchanged
                result.add(count);
                continue;
            }
            isLand[id] = true;
            count++;                          // new island for now
            for (int[] d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                int nid = nr * n + nc;
                if (isLand[nid] && uf.union(id, nid)) {
                    count--;                  // merged two distinct islands
                }
            }
            result.add(count);
        }
        return result;
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

**Dry run.** `m = 3, n = 3`, `positions = [[0,0],[0,1],[1,2],[2,1],[1,1]]`.

| Step | Add | id | New land? | Neighbor lands merged | count after | result |
|------|-----|----|-----------|------------------------|-------------|--------|
| 1 | (0,0) | 0 | yes -> count 1 | none | 1 | [1] |
| 2 | (0,1) | 1 | yes -> count 2 | (0,0) is land -> merge, count 1 | 1 | [1,1] |
| 3 | (1,2) | 5 | yes -> count 2 | none adjacent | 2 | [1,1,2] |
| 4 | (2,1) | 7 | yes -> count 3 | none adjacent (yet) | 3 | [1,1,2,3] |
| 5 | (1,1) | 4 | yes -> count 4 | up=(0,1) land -> merge (count 3); down=(2,1) land -> merge (count 2); right=(1,2) land -> merge (count 1) | 1 | [1,1,2,3,1] |

At step 5 the new land at (1,1) bridges three previously separate islands at once: each successful `union` drops the count, taking it from 4 down to 1. Final answer: `[1, 1, 2, 3, 1]`.

**Complexity:** Let `k = positions.length`. Time O(k·α(m·n)). Space O(m·n).

---

## 44. Regions Cut By Slashes — Medium (LC959)

**Idea:** Subdivide each `1×1` cell into 4 triangles indexed **0 = top, 1 = right, 2 = bottom, 3 = left**. Within a cell: `'/'` separates {top,left} from {bottom,right}; `'\\'` separates {top,right} from {bottom,left}; `' '` unions all four. Across cells: a cell's right triangle (1) unions the right neighbor's left (3); a cell's bottom triangle (2) unions the lower neighbor's top (0). The number of regions equals the final component count.

```java
class Solution {
    public int regionsBySlashes(String[] grid) {
        int n = grid.length;
        UnionFind uf = new UnionFind(4 * n * n);

        for (int r = 0; r < n; r++) {
            String row = grid[r];
            for (int c = 0; c < n; c++) {
                int base = 4 * (r * n + c); // base index of this cell's 4 triangles
                char ch = row.charAt(c);
                // 0=top, 1=right, 2=bottom, 3=left
                if (ch == '/') {
                    uf.union(base + 0, base + 3); // top + left
                    uf.union(base + 1, base + 2); // right + bottom
                } else if (ch == '\\') {
                    uf.union(base + 0, base + 1); // top + right
                    uf.union(base + 2, base + 3); // bottom + left
                } else { // ' '
                    uf.union(base + 0, base + 1);
                    uf.union(base + 1, base + 2);
                    uf.union(base + 2, base + 3);
                }
                // Union across cell boundaries.
                if (c + 1 < n) { // my right (1) with right-neighbor's left (3)
                    int rightBase = 4 * (r * n + (c + 1));
                    uf.union(base + 1, rightBase + 3);
                }
                if (r + 1 < n) { // my bottom (2) with lower-neighbor's top (0)
                    int downBase = 4 * ((r + 1) * n + c);
                    uf.union(base + 2, downBase + 0);
                }
            }
        }
        return uf.count();
    }

    class UnionFind {
        int[] parent, rank;
        int count;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            count = n;
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            count--;
        }
        int count() { return count; }
    }
}
```

**Complexity:** Time O(n²·α). Space O(n²).

---

## 45. Smallest String With Swaps — Medium (LC1202) 

**Idea:** Each pair `[i, j]` means indices `i` and `j` can be swapped any number of times; transitively, all indices in one connected component can be permuted freely. So group indices by DSU root, sort the characters within each group, and place the smallest characters at the smallest indices.

```java
import java.util.*;

class Solution {
    public String smallestStringWithSwaps(String s, List<List<Integer>> pairs) {
        int n = s.length();
        UnionFind uf = new UnionFind(n);
        for (List<Integer> p : pairs) {
            uf.union(p.get(0), p.get(1));
        }

        // Bucket indices by their root.
        Map<Integer, List<Integer>> groups = new HashMap<>();
        for (int i = 0; i < n; i++) {
            groups.computeIfAbsent(uf.find(i), k -> new ArrayList<>()).add(i);
        }

        char[] result = s.toCharArray();
        for (List<Integer> indices : groups.values()) {
            // Collect the characters at these indices and sort them.
            List<Character> chars = new ArrayList<>();
            for (int idx : indices) chars.add(s.charAt(idx));
            Collections.sort(chars);
            // indices is already ascending (built by increasing i); assign smallest chars first.
            for (int k = 0; k < indices.size(); k++) {
                result[indices.get(k)] = chars.get(k);
            }
        }
        return new String(result);
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
        }
    }
}
```

**Complexity:** Time O(n log n + P·α(n)) where P = number of pairs. Space O(n).

---

## 46. Graph Valid Tree — Medium (LC261)

**Idea:** A graph on `n` nodes is a tree iff it is fully connected **and** acyclic. With DSU: it must have exactly `n−1` edges, and every `union` must succeed (a `union` that returns `false` means both endpoints are already connected — a cycle).

```java
class Solution {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n - 1) return false; // tree must have exactly n-1 edges

        UnionFind uf = new UnionFind(n);
        for (int[] e : edges) {
            if (!uf.union(e[0], e[1])) {
                return false; // cycle: endpoints already connected
            }
        }
        // n-1 successful unions guarantee a single connected acyclic component.
        return true;
    }

    class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            rank = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        boolean union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
            parent[rb] = ra;
            if (rank[ra] == rank[rb]) rank[ra]++;
            return true;
        }
    }
}
```

> Why both checks? `n−1` edges with no cycle ⇒ connected (a forest with `n−1` edges and no cycle is a single tree). Either condition alone is insufficient: `n−1` edges could still form a cycle plus a disconnected node, and "all unions succeed" alone doesn't guarantee full connectivity if there are too few edges.

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

> **Last Updated:** 2026-06-26
