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

Use this DSU class throughout. It carries both `rnk` (for union by rank) and `sz` (handy for problems that need the size of a component), plus a live `count` of disjoint sets.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct UnionFind {
    vector<int> parent, rnk, sz;
    int count; // number of disjoint sets

    UnionFind(int n) : parent(n), rnk(n, 0), sz(n, 1), count(n) {
        iota(parent.begin(), parent.end(), 0);
    }

    // find with path compression (iterative, no stack overflow risk)
    int find(int x) {
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

    // unite by rank; returns true if a real merge happened, false if already connected
    bool unite(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (rnk[ra] < rnk[rb]) swap(ra, rb);
        parent[rb] = ra;
        sz[ra] += sz[rb];
        if (rnk[ra] == rnk[rb]) rnk[ra]++;
        count--;
        return true;
    }

    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    int size(int x) {
        return sz[find(x)];
    }

    int getCount() {
        return count;
    }
};
```

> The `unite` return value (`true` = merged, `false` = already in same set) is the single most useful signal in DSU problems — it instantly detects cycles and redundant edges.

## Recognition Signals

| Signal in the prompt | Why DSU fits |
|----------------------|--------------|
| "Are these two connected?" asked repeatedly | `connected(a, b)` in O(α) |
| Edges added one at a time; answer changes per addition | Incremental `unite`, online queries |
| "Find the edge that creates a cycle" | First `unite` returning `false` |
| "How many groups / provinces / islands / components?" | `getCount()` |
| "Merge accounts / friends / equal things" | Transitive grouping by root |
| "Valid tree?" | n−1 edges **and** every `unite` succeeds |
| "Equations a==b, a!=b — consistent?" | Unite the `==`, verify the `!=` |
| Grid connectivity where you union by shared row/column | Map labels/cells to DSU indices |

---

## 38. Redundant Connection — Medium (LC684)

**Idea:** A tree with `n` nodes has exactly `n−1` edges; one extra edge creates a cycle. Process edges in order and return the first one whose two endpoints are *already* connected.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        bool unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
            return true;
        }
    };
public:
    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();            // n nodes, n edges (1-indexed)
        UnionFind uf(n + 1);
        for (auto& e : edges) {
            if (!uf.unite(e[0], e[1])) {  // already connected -> cycle edge
                return e;
            }
        }
        return {}; // unreachable per constraints
    }
};
```

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

---

## 39. Accounts Merge — Medium (LC721)

**Idea:** Union all emails belonging to the same account (and across accounts that share any email). Then group emails by their DSU root, sort each group, and prepend the owner's name.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
        }
    };
public:
    vector<vector<string>> accountsMerge(vector<vector<string>>& accounts) {
        int n = accounts.size();
        UnionFind uf(n);

        // Map each email to the index of the first account it appears in.
        unordered_map<string, int> emailToAccount;
        for (int i = 0; i < n; i++) {
            auto& acc = accounts[i];
            for (int j = 1; j < (int)acc.size(); j++) {
                string email = acc[j];
                if (emailToAccount.count(email)) {
                    uf.unite(i, emailToAccount[email]); // shared email -> same person
                } else {
                    emailToAccount[email] = i;
                }
            }
        }

        // Collect emails under each account's root.
        unordered_map<int, set<string>> rootToEmails;
        for (auto& [email, acctIdx] : emailToAccount) {
            int root = uf.find(acctIdx);
            rootToEmails[root].insert(email);
        }

        // Build result: name + sorted emails.
        vector<vector<string>> result;
        for (auto& [root, emails] : rootToEmails) {
            string name = accounts[root][0];
            vector<string> merged = {name};
            merged.insert(merged.end(), emails.begin(), emails.end()); // set keeps them sorted
            result.push_back(merged);
        }
        return result;
    }
};
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

1. **Build `emailToAccount` and unite as we go.**
   - Account 0: `a@x.com` -> not seen, map `a@x.com=0`. `b@x.com` -> map `b@x.com=0`.
   - Account 1: `c@x.com` -> map `c@x.com=1`.
   - Account 2: `a@x.com` -> already maps to 0, so `unite(2, 0)`. Now roots: parent[2]=0 (say). `c@x.com` -> already maps to 1, so `unite(2, 1)`; find(2)=0, find(1)=1, merge -> parent[1]=0. Now accounts {0,1,2} share one root (0).
   - Account 3: `m@x.com` -> map `m@x.com=3`. Stays alone.

2. **Group emails by root via `find`.**
   - `a@x.com` (acct 0) -> root 0
   - `b@x.com` (acct 0) -> root 0
   - `c@x.com` (acct 1) -> find(1)=0 -> root 0
   - `m@x.com` (acct 3) -> root 3
   - `rootToEmails = { 0: {a@x.com, b@x.com, c@x.com}, 3: {m@x.com} }` (std::set auto-sorts).

3. **Emit with names.**
   - Root 0 -> name `accounts[0][0]` = "John" -> `["John","a@x.com","b@x.com","c@x.com"]`.
   - Root 3 -> "Mary" -> `["Mary","m@x.com"]`.

The two "John" accounts that shared `a@x.com`/`c@x.com` collapse into one, while the unrelated "Mary" stays separate. Even though all three Johns happen to share the same name, they merge only because of shared *emails*, not the name.

**Complexity:** Let `N` be the total number of emails. Time O(N·α(N) + N log N) for sorting via std::set. Space O(N).

---

## 40. Number of Operations to Make Network Connected — Medium (LC1319)

**Idea:** To connect `c` components you need `c−1` cables. You can only reuse cables you already have, so you must hold at least `n−1` edges total; otherwise it is impossible (return −1). Otherwise the answer is `components − 1`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        int count;
        UnionFind(int n) : parent(n), rnk(n, 0), count(n) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
            count--;
        }
        int getCount() { return count; }
    };
public:
    int makeConnected(int n, vector<vector<int>>& connections) {
        if ((int)connections.size() < n - 1) return -1; // not enough cables to ever connect

        UnionFind uf(n);
        for (auto& c : connections) {
            uf.unite(c[0], c[1]);
        }
        return uf.getCount() - 1; // cables needed = components - 1
    }
};
```

**Complexity:** Time O(E·α(n) + n). Space O(n).

---

## 41. Satisfiability of Equality Equations — Medium (LC990)

**Idea:** Two-pass. First union every variable joined by `==`. Then for every `!=`, if the two variables are already in the same set, it is a contradiction.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
        }
        bool connected(int a, int b) { return find(a) == find(b); }
    };
public:
    bool equationsPossible(vector<string>& equations) {
        UnionFind uf(26); // variables are single lowercase letters

        // Pass 1: unite all equalities.
        for (auto& eq : equations) {
            if (eq[1] == '=') {
                uf.unite(eq[0] - 'a', eq[3] - 'a');
            }
        }
        // Pass 2: verify inequalities don't conflict.
        for (auto& eq : equations) {
            if (eq[1] == '!') {
                if (uf.connected(eq[0] - 'a', eq[3] - 'a')) {
                    return false;
                }
            }
        }
        return true;
    }
};
```

**Complexity:** Time O(n·α(26)) ≈ O(n) where n = number of equations. Space O(1) (fixed 26 variables).

---

## 42. Most Stones Removed with Same Row or Column — Medium (LC947)

**Idea:** A stone can be removed if it shares a row or column with another stone — i.e., stones in the same connected component can be reduced to one. Union stones by shared row and column; the answer is `total stones − number of components`. To union by row/column cheaply, map columns into a separate index space.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
        }
    };
public:
    int removeStones(vector<vector<int>>& stones) {
        int n = stones.size();
        UnionFind uf(20000); // rows 0..9999, cols 10000..19999

        for (auto& s : stones) {
            int row = s[0];
            int col = s[1] + 10000; // shift columns into disjoint index range
            uf.unite(row, col);
        }

        // Count distinct roots among the indices actually used.
        unordered_set<int> roots;
        for (auto& s : stones) {
            roots.insert(uf.find(s[0]));
        }
        return n - (int)roots.size(); // stones - components
    }
};
```

> **Why count roots from the row index only?** Each stone unites its row with its column, so a stone's row index already belongs to the same component as its column. Collecting `find(row)` over all stones therefore yields exactly the distinct components.

**Complexity:** Time O(n·α). Space O(R + C) for the index space (≈ 20000).

---

## 43. Number of Islands II — Hard (LC305)

**Idea:** Process land additions online. Start with 0 islands. Each `addLand(r, c)`: if the cell is new land, increment the count, then union it with any of its 4 neighbors that are already land — each successful merge of two distinct islands decrements the count. Record the count after every operation.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        bool unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
            return true;
        }
    };
public:
    vector<int> numIslands2(int m, int n, vector<vector<int>>& positions) {
        UnionFind uf(m * n);
        vector<bool> isLand(m * n, false);
        vector<int> result;
        int dirs[4][2] = {{-1,0},{1,0},{0,-1},{0,1}};
        int count = 0;

        for (auto& p : positions) {
            int r = p[0], c = p[1];
            int id = r * n + c;
            if (isLand[id]) {                 // duplicate add -> count unchanged
                result.push_back(count);
                continue;
            }
            isLand[id] = true;
            count++;                          // new island for now
            for (auto& d : dirs) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                int nid = nr * n + nc;
                if (isLand[nid] && uf.unite(id, nid)) {
                    count--;                  // merged two distinct islands
                }
            }
            result.push_back(count);
        }
        return result;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        int count;
        UnionFind(int n) : parent(n), rnk(n, 0), count(n) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
            count--;
        }
        int getCount() { return count; }
    };
public:
    int regionsBySlashes(vector<string>& grid) {
        int n = grid.size();
        UnionFind uf(4 * n * n);

        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                int base = 4 * (r * n + c); // base index of this cell's 4 triangles
                char ch = grid[r][c];
                // 0=top, 1=right, 2=bottom, 3=left
                if (ch == '/') {
                    uf.unite(base + 0, base + 3); // top + left
                    uf.unite(base + 1, base + 2); // right + bottom
                } else if (ch == '\\') {
                    uf.unite(base + 0, base + 1); // top + right
                    uf.unite(base + 2, base + 3); // bottom + left
                } else { // ' '
                    uf.unite(base + 0, base + 1);
                    uf.unite(base + 1, base + 2);
                    uf.unite(base + 2, base + 3);
                }
                // Unite across cell boundaries.
                if (c + 1 < n) { // my right (1) with right-neighbor's left (3)
                    int rightBase = 4 * (r * n + (c + 1));
                    uf.unite(base + 1, rightBase + 3);
                }
                if (r + 1 < n) { // my bottom (2) with lower-neighbor's top (0)
                    int downBase = 4 * ((r + 1) * n + c);
                    uf.unite(base + 2, downBase + 0);
                }
            }
        }
        return uf.getCount();
    }
};
```

**Complexity:** Time O(n²·α). Space O(n²).

---

## 45. Smallest String With Swaps — Medium (LC1202) 

**Idea:** Each pair `[i, j]` means indices `i` and `j` can be swapped any number of times; transitively, all indices in one connected component can be permuted freely. So group indices by DSU root, sort the characters within each group, and place the smallest characters at the smallest indices.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        void unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
        }
    };
public:
    string smallestStringWithSwaps(string s, vector<vector<int>>& pairs) {
        int n = s.length();
        UnionFind uf(n);
        for (auto& p : pairs) {
            uf.unite(p[0], p[1]);
        }

        // Bucket indices by their root.
        unordered_map<int, vector<int>> groups;
        for (int i = 0; i < n; i++) {
            groups[uf.find(i)].push_back(i);
        }

        string result = s;
        for (auto& [root, indices] : groups) {
            // Collect the characters at these indices and sort them.
            vector<char> chars;
            for (int idx : indices) chars.push_back(s[idx]);
            sort(chars.begin(), chars.end());
            // indices is already ascending (built by increasing i); assign smallest chars first.
            for (int k = 0; k < (int)indices.size(); k++) {
                result[indices[k]] = chars[k];
            }
        }
        return result;
    }
};
```

**Complexity:** Time O(n log n + P·α(n)) where P = number of pairs. Space O(n).

---

## 46. Graph Valid Tree — Medium (LC261)

**Idea:** A graph on `n` nodes is a tree iff it is fully connected **and** acyclic. With DSU: it must have exactly `n−1` edges, and every `unite` must succeed (a `unite` that returns `false` means both endpoints are already connected — a cycle).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    struct UnionFind {
        vector<int> parent, rnk;
        UnionFind(int n) : parent(n), rnk(n, 0) {
            iota(parent.begin(), parent.end(), 0);
        }
        int find(int x) {
            int root = x;
            while (root != parent[root]) root = parent[root];
            while (x != root) { int nx = parent[x]; parent[x] = root; x = nx; }
            return root;
        }
        bool unite(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return false;
            if (rnk[ra] < rnk[rb]) swap(ra, rb);
            parent[rb] = ra;
            if (rnk[ra] == rnk[rb]) rnk[ra]++;
            return true;
        }
    };
public:
    bool validTree(int n, vector<vector<int>>& edges) {
        if ((int)edges.size() != n - 1) return false; // tree must have exactly n-1 edges

        UnionFind uf(n);
        for (auto& e : edges) {
            if (!uf.unite(e[0], e[1])) {
                return false; // cycle: endpoints already connected
            }
        }
        // n-1 successful unites guarantee a single connected acyclic component.
        return true;
    }
};
```

> Why both checks? `n−1` edges with no cycle ⇒ connected (a forest with `n−1` edges and no cycle is a single tree). Either condition alone is insufficient: `n−1` edges could still form a cycle plus a disconnected node, and "all unites succeed" alone doesn't guarantee full connectivity if there are too few edges.

**Complexity:** Time O(n·α(n)) ≈ O(n). Space O(n).

> **Last Updated:** 2026-06-26
