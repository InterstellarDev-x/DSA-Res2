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

Use this DSU struct throughout. It carries both `rnk` (for union by rank) and `sz` (handy for problems that need the size of a component), plus a live `count` of disjoint sets.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
    sz: Vec<usize>,
    count: usize, // number of disjoint sets
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
            sz: vec![1; n],
            count: n,
        }
    }

    // find with path compression (iterative, no stack overflow risk)
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] {
            root = self.parent[root];
        }
        let mut x = x;
        while x != root {       // compress the path
            let next = self.parent[x];
            self.parent[x] = root;
            x = next;
        }
        root
    }

    // unite by rank; returns true if a real merge happened, false if already connected
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        self.sz[ra] += self.sz[rb];
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        self.count -= 1;
        true
    }

    fn connected(&mut self, a: usize, b: usize) -> bool {
        self.find(a) == self.find(b)
    }

    fn size(&mut self, x: usize) -> usize {
        let root = self.find(x);
        self.sz[root]
    }

    fn get_count(&self) -> usize {
        self.count
    }
}
```

> The `unite` return value (`true` = merged, `false` = already in same set) is the single most useful signal in DSU problems — it instantly detects cycles and redundant edges.

## Recognition Signals

| Signal in the prompt | Why DSU fits |
|----------------------|--------------|
| "Are these two connected?" asked repeatedly | `connected(a, b)` in O(α) |
| Edges added one at a time; answer changes per addition | Incremental `unite`, online queries |
| "Find the edge that creates a cycle" | First `unite` returning `false` |
| "How many groups / provinces / islands / components?" | `get_count()` |
| "Merge accounts / friends / equal things" | Transitive grouping by root |
| "Valid tree?" | n−1 edges **and** every `unite` succeeds |
| "Equations a==b, a!=b — consistent?" | Unite the `==`, verify the `!=` |
| Grid connectivity where you union by shared row/column | Map labels/cells to DSU indices |

---

## 38. Redundant Connection — Medium (LC684)

**Idea:** A tree with `n` nodes has exactly `n−1` edges; one extra edge creates a cycle. Process edges in order and return the first one whose two endpoints are *already* connected.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    pub fn find_redundant_connection(edges: Vec<Vec<i32>>) -> Vec<i32> {
        let n = edges.len();            // n nodes, n edges (1-indexed)
        let mut uf = UnionFind::new(n + 1);
        for e in &edges {
            if !uf.unite(e[0] as usize, e[1] as usize) {  // already connected -> cycle edge
                return e.clone();
            }
        }
        vec![] // unreachable per constraints
    }
}
```

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

---

## 39. Accounts Merge — Medium (LC721)

**Idea:** Union all emails belonging to the same account (and across accounts that share any email). Then group emails by their DSU root, sort each group, and prepend the owner's name.

```rust
use std::collections::{HashMap, BTreeSet};

struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
    }
}

struct Solution;

impl Solution {
    pub fn accounts_merge(accounts: Vec<Vec<String>>) -> Vec<Vec<String>> {
        let n = accounts.len();
        let mut uf = UnionFind::new(n);

        // Map each email to the index of the first account it appears in.
        let mut email_to_account: HashMap<String, usize> = HashMap::new();
        for i in 0..n {
            let acc = &accounts[i];
            for j in 1..acc.len() {
                let email = acc[j].clone();
                if let Some(&acct_idx) = email_to_account.get(&email) {
                    uf.unite(i, acct_idx); // shared email -> same person
                } else {
                    email_to_account.insert(email, i);
                }
            }
        }

        // Collect emails under each account's root.
        let mut root_to_emails: HashMap<usize, BTreeSet<String>> = HashMap::new();
        for (email, acct_idx) in &email_to_account {
            let root = uf.find(*acct_idx);
            root_to_emails.entry(root).or_default().insert(email.clone());
        }

        // Build result: name + sorted emails.
        let mut result: Vec<Vec<String>> = Vec::new();
        for (root, emails) in &root_to_emails {
            let name = accounts[*root][0].clone();
            let mut merged = vec![name];
            merged.extend(emails.iter().cloned()); // BTreeSet keeps them sorted
            result.push(merged);
        }
        result
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

1. **Build `email_to_account` and unite as we go.**
   - Account 0: `a@x.com` -> not seen, map `a@x.com=0`. `b@x.com` -> map `b@x.com=0`.
   - Account 1: `c@x.com` -> map `c@x.com=1`.
   - Account 2: `a@x.com` -> already maps to 0, so `unite(2, 0)`. Now roots: parent[2]=0 (say). `c@x.com` -> already maps to 1, so `unite(2, 1)`; find(2)=0, find(1)=1, merge -> parent[1]=0. Now accounts {0,1,2} share one root (0).
   - Account 3: `m@x.com` -> map `m@x.com=3`. Stays alone.

2. **Group emails by root via `find`.**
   - `a@x.com` (acct 0) -> root 0
   - `b@x.com` (acct 0) -> root 0
   - `c@x.com` (acct 1) -> find(1)=0 -> root 0
   - `m@x.com` (acct 3) -> root 3
   - `root_to_emails = { 0: {a@x.com, b@x.com, c@x.com}, 3: {m@x.com} }` (BTreeSet auto-sorts).

3. **Emit with names.**
   - Root 0 -> name `accounts[0][0]` = "John" -> `["John","a@x.com","b@x.com","c@x.com"]`.
   - Root 3 -> "Mary" -> `["Mary","m@x.com"]`.

The two "John" accounts that shared `a@x.com`/`c@x.com` collapse into one, while the unrelated "Mary" stays separate. Even though all three Johns happen to share the same name, they merge only because of shared *emails*, not the name.

**Complexity:** Let `N` be the total number of emails. Time O(N·α(N) + N log N) for sorting via BTreeSet. Space O(N).

---

## 40. Number of Operations to Make Network Connected — Medium (LC1319)

**Idea:** To connect `c` components you need `c−1` cables. You can only reuse cables you already have, so you must hold at least `n−1` edges total; otherwise it is impossible (return −1). Otherwise the answer is `components − 1`.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
    count: usize,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
            count: n,
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        self.count -= 1;
    }
    fn get_count(&self) -> usize { self.count }
}

struct Solution;

impl Solution {
    pub fn make_connected(n: i32, connections: Vec<Vec<i32>>) -> i32 {
        let n = n as usize;
        if connections.len() < n - 1 { return -1; } // not enough cables to ever connect

        let mut uf = UnionFind::new(n);
        for c in &connections {
            uf.unite(c[0] as usize, c[1] as usize);
        }
        (uf.get_count() - 1) as i32 // cables needed = components - 1
    }
}
```

**Complexity:** Time O(E·α(n) + n). Space O(n).

---

## 41. Satisfiability of Equality Equations — Medium (LC990)

**Idea:** Two-pass. First union every variable joined by `==`. Then for every `!=`, if the two variables are already in the same set, it is a contradiction.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
    }
    fn connected(&mut self, a: usize, b: usize) -> bool { self.find(a) == self.find(b) }
}

struct Solution;

impl Solution {
    pub fn equations_possible(equations: Vec<String>) -> bool {
        let mut uf = UnionFind::new(26); // variables are single lowercase letters

        // Pass 1: unite all equalities.
        for eq in &equations {
            let bytes = eq.as_bytes();
            if bytes[1] == b'=' {
                uf.unite((bytes[0] - b'a') as usize, (bytes[3] - b'a') as usize);
            }
        }
        // Pass 2: verify inequalities don't conflict.
        for eq in &equations {
            let bytes = eq.as_bytes();
            if bytes[1] == b'!' {
                if uf.connected((bytes[0] - b'a') as usize, (bytes[3] - b'a') as usize) {
                    return false;
                }
            }
        }
        true
    }
}
```

**Complexity:** Time O(n·α(26)) ≈ O(n) where n = number of equations. Space O(1) (fixed 26 variables).

---

## 42. Most Stones Removed with Same Row or Column — Medium (LC947)

**Idea:** A stone can be removed if it shares a row or column with another stone — i.e., stones in the same connected component can be reduced to one. Union stones by shared row and column; the answer is `total stones − number of components`. To union by row/column cheaply, map columns into a separate index space.

```rust
use std::collections::HashSet;

struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
    }
}

struct Solution;

impl Solution {
    pub fn remove_stones(stones: Vec<Vec<i32>>) -> i32 {
        let n = stones.len();
        let mut uf = UnionFind::new(20000); // rows 0..9999, cols 10000..19999

        for s in &stones {
            let row = s[0] as usize;
            let col = s[1] as usize + 10000; // shift columns into disjoint index range
            uf.unite(row, col);
        }

        // Count distinct roots among the indices actually used.
        let mut roots: HashSet<usize> = HashSet::new();
        for s in &stones {
            roots.insert(uf.find(s[0] as usize));
        }
        (n - roots.len()) as i32 // stones - components
    }
}
```

> **Why count roots from the row index only?** Each stone unites its row with its column, so a stone's row index already belongs to the same component as its column. Collecting `find(row)` over all stones therefore yields exactly the distinct components.

**Complexity:** Time O(n·α). Space O(R + C) for the index space (≈ 20000).

---

## 43. Number of Islands II — Hard (LC305)

**Idea:** Process land additions online. Start with 0 islands. Each `addLand(r, c)`: if the cell is new land, increment the count, then union it with any of its 4 neighbors that are already land — each successful merge of two distinct islands decrements the count. Record the count after every operation.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    pub fn num_islands2(m: i32, n: i32, positions: Vec<Vec<i32>>) -> Vec<i32> {
        let (m, n) = (m as usize, n as usize);
        let mut uf = UnionFind::new(m * n);
        let mut is_land = vec![false; m * n];
        let mut result = Vec::new();
        let dirs: [(i32, i32); 4] = [(-1, 0), (1, 0), (0, -1), (0, 1)];
        let mut count = 0i32;

        for p in &positions {
            let r = p[0] as usize;
            let c = p[1] as usize;
            let id = r * n + c;
            if is_land[id] {                 // duplicate add -> count unchanged
                result.push(count);
                continue;
            }
            is_land[id] = true;
            count += 1;                      // new island for now
            for (dr, dc) in &dirs {
                let nr = p[0] + dr;
                let nc = p[1] + dc;
                if nr < 0 || nr >= m as i32 || nc < 0 || nc >= n as i32 { continue; }
                let nid = nr as usize * n + nc as usize;
                if is_land[nid] && uf.unite(id, nid) {
                    count -= 1;              // merged two distinct islands
                }
            }
            result.push(count);
        }
        result
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

At step 5 the new land at (1,1) bridges three previously separate islands at once: each successful `unite` drops the count, taking it from 4 down to 1. Final answer: `[1, 1, 2, 3, 1]`.

**Complexity:** Let `k = positions.length`. Time O(k·α(m·n)). Space O(m·n).

---

## 44. Regions Cut By Slashes — Medium (LC959)

**Idea:** Subdivide each `1×1` cell into 4 triangles indexed **0 = top, 1 = right, 2 = bottom, 3 = left**. Within a cell: `'/'` separates {top,left} from {bottom,right}; `'\\'` separates {top,right} from {bottom,left}; `' '` unions all four. Across cells: a cell's right triangle (1) unions the right neighbor's left (3); a cell's bottom triangle (2) unions the lower neighbor's top (0). The number of regions equals the final component count.

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
    count: usize,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
            count: n,
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        self.count -= 1;
    }
    fn get_count(&self) -> usize { self.count }
}

struct Solution;

impl Solution {
    pub fn regions_by_slashes(grid: Vec<String>) -> i32 {
        let n = grid.len();
        let mut uf = UnionFind::new(4 * n * n);

        for r in 0..n {
            let row_bytes = grid[r].as_bytes();
            for c in 0..n {
                let base = 4 * (r * n + c); // base index of this cell's 4 triangles
                let ch = row_bytes[c] as char;
                // 0=top, 1=right, 2=bottom, 3=left
                if ch == '/' {
                    uf.unite(base + 0, base + 3); // top + left
                    uf.unite(base + 1, base + 2); // right + bottom
                } else if ch == '\\' {
                    uf.unite(base + 0, base + 1); // top + right
                    uf.unite(base + 2, base + 3); // bottom + left
                } else { // ' '
                    uf.unite(base + 0, base + 1);
                    uf.unite(base + 1, base + 2);
                    uf.unite(base + 2, base + 3);
                }
                // Unite across cell boundaries.
                if c + 1 < n { // my right (1) with right-neighbor's left (3)
                    let right_base = 4 * (r * n + (c + 1));
                    uf.unite(base + 1, right_base + 3);
                }
                if r + 1 < n { // my bottom (2) with lower-neighbor's top (0)
                    let down_base = 4 * ((r + 1) * n + c);
                    uf.unite(base + 2, down_base + 0);
                }
            }
        }
        uf.get_count() as i32
    }
}
```

**Complexity:** Time O(n²·α). Space O(n²).

---

## 45. Smallest String With Swaps — Medium (LC1202) 

**Idea:** Each pair `[i, j]` means indices `i` and `j` can be swapped any number of times; transitively, all indices in one connected component can be permuted freely. So group indices by DSU root, sort the characters within each group, and place the smallest characters at the smallest indices.

```rust
use std::collections::HashMap;

struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
    }
}

struct Solution;

impl Solution {
    pub fn smallest_string_with_swaps(s: String, pairs: Vec<Vec<i32>>) -> String {
        let n = s.len();
        let mut uf = UnionFind::new(n);
        for p in &pairs {
            uf.unite(p[0] as usize, p[1] as usize);
        }

        // Bucket indices by their root.
        let mut groups: HashMap<usize, Vec<usize>> = HashMap::new();
        for i in 0..n {
            let root = uf.find(i);
            groups.entry(root).or_default().push(i);
        }

        let s_bytes: Vec<u8> = s.into_bytes();
        let mut result = s_bytes.clone();
        for (_root, indices) in &groups {
            // Collect the characters at these indices and sort them.
            let mut chars: Vec<u8> = indices.iter().map(|&idx| s_bytes[idx]).collect();
            chars.sort();
            // indices is already ascending (built by increasing i); assign smallest chars first.
            for (k, &idx) in indices.iter().enumerate() {
                result[idx] = chars[k];
            }
        }
        String::from_utf8(result).unwrap()
    }
}
```

**Complexity:** Time O(n log n + P·α(n)) where P = number of pairs. Space O(n).

---

## 46. Graph Valid Tree — Medium (LC261)

**Idea:** A graph on `n` nodes is a tree iff it is fully connected **and** acyclic. With DSU: it must have exactly `n−1` edges, and every `unite` must succeed (a `unite` that returns `false` means both endpoints are already connected — a cycle).

```rust
struct UnionFind {
    parent: Vec<usize>,
    rnk: Vec<usize>,
}

impl UnionFind {
    fn new(n: usize) -> Self {
        let parent: Vec<usize> = (0..n).collect();
        UnionFind {
            parent,
            rnk: vec![0; n],
        }
    }
    fn find(&mut self, x: usize) -> usize {
        let mut root = x;
        while root != self.parent[root] { root = self.parent[root]; }
        let mut x = x;
        while x != root { let nx = self.parent[x]; self.parent[x] = root; x = nx; }
        root
    }
    fn unite(&mut self, a: usize, b: usize) -> bool {
        let ra = self.find(a);
        let rb = self.find(b);
        if ra == rb { return false; }
        let (ra, rb) = if self.rnk[ra] < self.rnk[rb] { (rb, ra) } else { (ra, rb) };
        self.parent[rb] = ra;
        if self.rnk[ra] == self.rnk[rb] { self.rnk[ra] += 1; }
        true
    }
}

struct Solution;

impl Solution {
    pub fn valid_tree(n: i32, edges: Vec<Vec<i32>>) -> bool {
        let n = n as usize;
        if edges.len() != n - 1 { return false; } // tree must have exactly n-1 edges

        let mut uf = UnionFind::new(n);
        for e in &edges {
            if !uf.unite(e[0] as usize, e[1] as usize) {
                return false; // cycle: endpoints already connected
            }
        }
        // n-1 successful unites guarantee a single connected acyclic component.
        true
    }
}
```

> Why both checks? `n−1` edges with no cycle ⇒ connected (a forest with `n−1` edges and no cycle is a single tree). Either condition alone is insufficient: `n−1` edges could still form a cycle plus a disconnected node, and "all unites succeed" alone doesn't guarantee full connectivity if there are too few edges.

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

> **Last Updated:** 2026-06-26
