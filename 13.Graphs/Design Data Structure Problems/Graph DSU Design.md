> **Topic:** [Graphs](../README.md) · **Design Data Structure**

# Graph DSU Design

## Overview

A **Disjoint Set Union (DSU)**, also known as **Union-Find**, is one of the
most reusable data structures in graph problem solving. It maintains a
partition of a universe of elements into disjoint sets and answers two
questions extremely fast:

- **Which set does element `x` belong to?** (`find`)
- **Are `a` and `b` in the same set?** (`connected`)

while also supporting the mutation:

- **Merge the sets containing `a` and `b`.** (`unite`)

### Why it matters for graphs

1. **Dynamic connectivity.** As edges are added one at a time, DSU tells you
   in near constant time whether two vertices are already connected. This is
   the backbone of **Kruskal's MST** (skip an edge if its endpoints are
   already connected, otherwise unite them) and of detecting cycles in an
   undirected graph.
2. **Grouping / clustering.** Problems that ask for the *number of connected
   components*, *number of provinces*, *number of islands*, *accounts merge*,
   *friend circles*, or *redundant connection* are all DSU in disguise. You
   unite related items and read off `count()` or group by representative.
3. **Online vs. offline.** Plain DSU is an *incremental* (online) structure:
   it handles unions and queries interleaved. With a **rollback** variant you
   can also support *offline dynamic connectivity* where edges are added and
   removed (segment-tree-on-time + rollback DSU).
4. **Relational constraints.** A **weighted DSU** stores the relationship of
   each node to its representative (an offset or a ratio), which lets you
   answer "what is the relation between `a` and `b`?" — used for *Evaluate
   Division*, *Satisfiability of Equality Equations*, and parity/2-coloring
   problems.

Two optimizations make DSU practically constant time:

- **Path compression** in `find`: flatten the tree so future lookups are
  direct.
- **Union by rank** (or by size): always attach the shorter tree under the
  taller one to keep trees flat.

Together they give an amortized cost of **O(α(n))** per operation, where
`α` is the inverse Ackermann function — effectively a small constant (≤ 4)
for any input that fits in the universe.

---

## 1. Production `UnionFind` for integer nodes `[0, n)`

```cpp
#include <bits/stdc++.h>
using namespace std;

// Disjoint Set Union (Union-Find) over the integer universe [0, n).
//
// Supports near-constant-time unite and find via
// path compression and union by rank. Also tracks the current number of
// disjoint components.
class UnionFind {

    // parent[x] is the parent of x; a root r satisfies parent[r] == r.
    vector<int> parent;

    // rnk[x] is an upper bound on the height of the tree rooted at x.
    vector<int> rnk;

    // Current number of disjoint sets (components).
    int cnt;

public:
    // Creates a structure with n singleton sets {0}, {1}, ..., {n-1}.
    // n: the number of elements; must be non-negative
    UnionFind(int n) {
        if (n < 0) {
            throw invalid_argument("n must be non-negative: " + to_string(n));
        }
        parent.resize(n);
        rnk.resize(n, 0);
        cnt = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;   // each element is its own root initially
        }
    }

    // Returns the representative (root) of the set containing x,
    // applying iterative path compression so that every node on the path
    // points directly at the root afterwards.
    int find(int x) {
        // First pass: locate the root.
        int root = x;
        while (root != parent[root]) {
            root = parent[root];
        }
        // Second pass: compress the path by repointing each node to root.
        while (x != root) {
            int next = parent[x];
            parent[x] = root;
            x = next;
        }
        return root;
    }

    // Merges the sets containing a and b using union by rank.
    // Returns true if a and b were in different sets and a merge
    // occurred; false if they were already connected.
    bool unite(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false;   // already in the same set, nothing to do
        }
        // Attach the lower-rank tree under the higher-rank tree.
        if (rnk[rootA] < rnk[rootB]) {
            parent[rootA] = rootB;
        } else if (rnk[rootA] > rnk[rootB]) {
            parent[rootB] = rootA;
        } else {
            parent[rootB] = rootA;
            rnk[rootA]++;   // equal ranks: pick one as root and bump its rank
        }
        cnt--;
        return true;
    }

    // Reports whether a and b belong to the same set.
    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    // Returns the current number of disjoint components.
    int count() {
        return cnt;
    }
};
```

### Example: counting connected components

```cpp
#include <bits/stdc++.h>
using namespace std;

// Graph with 5 vertices and edges (0-1), (1-2), (3-4).
UnionFind uf(5);
uf.unite(0, 1);
uf.unite(1, 2);
uf.unite(3, 4);

cout << uf.count() << "\n";          // 2  -> {0,1,2} and {3,4}
cout << uf.connected(0, 2) << "\n";  // true
cout << uf.connected(0, 3) << "\n";  // false
```

---

## 2. Generic `UnionFind<T>` for arbitrary nodes

When nodes are strings, longs, or custom objects (e.g. account emails,
city names, coordinate pairs), back the structure with hash maps instead of
arrays. Elements are created lazily on first reference.

```cpp
#include <bits/stdc++.h>
using namespace std;

// Generic Disjoint Set Union over an arbitrary element type T.
//
// Elements are introduced lazily: any element passed to makeSet,
// find, or unite that is not yet known becomes a new
// singleton set. T must have well-behaved == and std::hash support.
//
// @tparam T the element type
template<typename T>
class UnionFind {

    // Maps each element to its parent; a root maps to itself.
    unordered_map<T, T> parent;

    // Maps each root candidate to its rank (tree height upper bound).
    unordered_map<T, int> rnk;

    // Current number of disjoint sets.
    int cnt = 0;

public:
    // Ensures x exists as a singleton set if it is not already known.
    void makeSet(const T& x) {
        if (!parent.count(x)) {
            parent[x] = x;
            rnk[x] = 0;
            cnt++;
        }
    }

    // Returns the representative of x's set, registering x
    // first if needed, with full path compression.
    T find(const T& x) {
        makeSet(x);
        // Locate the root.
        T root = x;
        while (!(root == parent[root])) {
            root = parent[root];
        }
        // Compress the path.
        T cur = x;
        while (!(cur == root)) {
            T next = parent[cur];
            parent[cur] = root;
            cur = next;
        }
        return root;
    }

    // Merges the sets containing a and b by rank.
    // Returns true if a merge occurred, false if already joined.
    bool unite(const T& a, const T& b) {
        T rootA = find(a);
        T rootB = find(b);
        if (rootA == rootB) {
            return false;
        }
        int rankA = rnk[rootA];
        int rankB = rnk[rootB];
        if (rankA < rankB) {
            parent[rootA] = rootB;
        } else if (rankA > rankB) {
            parent[rootB] = rootA;
        } else {
            parent[rootB] = rootA;
            rnk[rootA] = rankA + 1;
        }
        cnt--;
        return true;
    }

    // Reports whether a and b are in the same set.
    bool connected(const T& a, const T& b) {
        return find(a) == find(b);
    }

    // Returns the current number of disjoint sets.
    int count() {
        return cnt;
    }
};
```

### Example: grouping strings

```cpp
#include <bits/stdc++.h>
using namespace std;

UnionFind<string> uf;
uf.unite("alice@a.com", "alice@b.com");
uf.unite("bob@x.com",   "bob@y.com");

cout << uf.connected("alice@a.com", "alice@b.com") << "\n"; // true
cout << uf.connected("alice@a.com", "bob@x.com") << "\n";   // false
cout << uf.count() << "\n";                                  // 2
```

---

## 3. Complexity analysis

With both **path compression** and **union by rank**, any sequence of `m`
operations on `n` elements runs in **O(m · α(n))** total, i.e. essentially
**O(1) amortized** per operation. Here `α(n)` is the **inverse Ackermann
function**, which grows so slowly that `α(n) ≤ 4` for every `n` that could
ever be stored in practice (well beyond the number of atoms in the universe).

| Operation                | Array `UnionFind` | Generic `UnionFind<T>` | Notes |
|--------------------------|-------------------|------------------------|-------|
| `find(x)`                | O(α(n)) amortized | O(α(n)) amortized*     | `*` plus hash-map constant per node touched |
| `unite(a, b)`            | O(α(n)) amortized | O(α(n)) amortized*     | two finds + a constant merge |
| `connected(a, b)`        | O(α(n)) amortized | O(α(n)) amortized*     | two finds |
| `count()`                | O(1)              | O(1)                   | maintained incrementally |
| Construction / `makeSet` | O(n) / O(1)       | O(1) per element       | array init vs. lazy insert |
| **Space**                | **O(n)**          | **O(n)**               | arrays vs. two hash maps |

> Without path compression *or* union by rank, a single `find` can degrade to
> **O(n)** in the worst case (a degenerate linked-list-shaped tree). Using only
> union by rank gives **O(log n)** per operation; adding path compression drops
> it to the inverse-Ackermann bound.

---

## 4. Follow-ups

### 4a. Weighted / relational DSU

Sometimes you do not just want to know *whether* `a` and `b` are related, but
*how*. Store a **weight (offset) from each node to its parent**. The classic
applications are:

- **Evaluate Division** (LeetCode 399): `weight[x]` = ratio `value(x) / value(parent(x))`, combined multiplicatively. `a / b` is the product of weights along the path.
- **Equation / parity problems**: `weight[x]` = additive offset (e.g. `value(x) - value(parent(x))`).

Below is a multiplicative-ratio version (Evaluate Division). `find` accumulates
the product of edge weights while compressing, and `unite(a, b, ratio)` records
`a / b == ratio`.

```cpp
#include <bits/stdc++.h>
using namespace std;

// Weighted (relational) Disjoint Set Union for ratio constraints of the form
// value(a) / value(b) == ratio.
//
// weight[x] stores value(x) / value(parent[x]).
// Path compression multiplies weights along the path so each node ends up
// storing its ratio directly to the root.
class WeightedUnionFind {

    unordered_map<string, string> parent;
    unordered_map<string, double> weight;

public:
    // Registers x as a singleton with ratio 1.0 to itself.
    void makeSet(const string& x) {
        if (!parent.count(x)) {
            parent[x] = x;
            weight[x] = 1.0;
        }
    }

    // Finds the root of x while accumulating the multiplicative weight
    // from x up to the root, and compresses the path so that
    // weight[x] becomes value(x) / value(root).
    string find(const string& x) {
        makeSet(x);
        if (parent[x] == x) {
            return x;
        }
        string p = parent[x];
        string root = find(p);                     // recurse: now weight[p] = p/root
        weight[x] = weight[x] * weight[p];         // x/root = x/p * p/root
        parent[x] = root;                          // point directly at root
        return root;
    }

    // Records the relation value(a) / value(b) == ratio by merging the
    // two sets. If they are already connected this is treated as consistent
    // input and ignored.
    // a: numerator element, b: denominator element, ratio: value(a) / value(b)
    void unite(const string& a, const string& b, double ratio) {
        makeSet(a);
        makeSet(b);
        string rootA = find(a);
        string rootB = find(b);
        if (rootA == rootB) {
            return; // already related (assumes consistent input)
        }
        // We need: value(a)/value(b) = ratio.
        // weight[a] = a/rootA, weight[b] = b/rootB.
        // Attach rootA under rootB. Set weight[rootA] = rootA/rootB so that
        //   a/b = (a/rootA) * (rootA/rootB) * (rootB/b) = ratio
        // => rootA/rootB = ratio * (b/rootB) / (a/rootA)
        //               = ratio * weight[b] / weight[a]
        weight[rootA] = ratio * weight[b] / weight[a];
        parent[rootA] = rootB;
    }

    // Returns value(a) / value(b) if both are known and connected,
    // otherwise -1.0 (the LeetCode "Evaluate Division" sentinel).
    double query(const string& a, const string& b) {
        if (!parent.count(a) || !parent.count(b)) {
            return -1.0;
        }
        string rootA = find(a);
        string rootB = find(b);
        if (rootA != rootB) {
            return -1.0;
        }
        // a/b = (a/root) / (b/root) = weight[a] / weight[b]
        return weight[a] / weight[b];
    }
};
```

```cpp
#include <bits/stdc++.h>
using namespace std;

// a/b = 2, b/c = 3  =>  a/c = 6, c/a = 1/6, x unknown.
WeightedUnionFind wuf;
wuf.unite("a", "b", 2.0);
wuf.unite("b", "c", 3.0);

cout << wuf.query("a", "c") << "\n";  // 6.0
cout << wuf.query("c", "a") << "\n";  // 0.1666...
cout << wuf.query("a", "x") << "\n";  // -1.0  (x not present)
```

> For **additive** problems (parity, "differences between variables") replace
> the multiplications with additions and the ratio with an offset, and `query`
> returns `weight[a] - weight[b]`.

### 4b. Rollback / persistent DSU

For **offline dynamic connectivity** (edges appear and disappear over time) or
**DSU on tree / small-to-large with undo**, you need to *undo* unions in
reverse order. This requires:

- **Union by rank/size only — no path compression.** Path compression mutates
  arbitrarily many nodes per `find`, which cannot be undone in O(1). Without it,
  each `unite` changes at most one `parent` entry and one `rank` entry, so it is
  cheaply reversible. The trade-off is **O(log n)** per `find` instead of
  inverse Ackermann.
- A **history stack** recording exactly what each `unite` changed.

```cpp
#include <bits/stdc++.h>
using namespace std;

// Rollback (undoable) Disjoint Set Union using union by rank WITHOUT path
// compression. Every successful unite can be reversed in O(1) via
// rollback(), in last-in-first-out order.
//
// find is O(log n) because path compression is intentionally
// omitted to keep unions undoable.
class RollbackUnionFind {

    vector<int> parent;
    vector<int> rnk;
    int cnt;

    // One reversible change: the root that was attached, plus whether the
    // surviving root's rank was incremented during this unite.
    struct Change {
        int attachedRoot;       // root whose parent was changed
        bool rankIncreased;     // did the surviving root's rank go up?
        int survivingRoot;      // root that remained a root

        Change(int attachedRoot, int survivingRoot, bool rankIncreased)
            : attachedRoot(attachedRoot), survivingRoot(survivingRoot),
              rankIncreased(rankIncreased) {}
    };

    stack<Change> history;

public:
    // Creates n singleton sets {0}, ..., {n-1}.
    RollbackUnionFind(int n) : parent(n), rnk(n, 0), cnt(n) {
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    // Returns the root of x WITHOUT path compression.
    int find(int x) {
        while (x != parent[x]) {
            x = parent[x];
        }
        return x;
    }

    // Merges the sets of a and b by rank, pushing the change
    // onto the history stack so it can be undone.
    // Returns true if a merge happened (a rollback point was pushed),
    // false if already connected (nothing pushed).
    bool unite(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false; // no change recorded
        }
        // Ensure rootA is the higher (or equal) rank root.
        if (rnk[rootA] < rnk[rootB]) {
            swap(rootA, rootB);
        }
        // Attach rootB under rootA.
        bool rankIncreased = (rnk[rootA] == rnk[rootB]);
        parent[rootB] = rootA;
        if (rankIncreased) {
            rnk[rootA]++;
        }
        cnt--;
        history.push(Change(rootB, rootA, rankIncreased));
        return true;
    }

    // Undoes the most recent successful unite. Call only when at least
    // one unite is on the history stack.
    void rollback() {
        Change c = history.top();
        history.pop();
        parent[c.attachedRoot] = c.attachedRoot; // detach: make it a root again
        if (c.rankIncreased) {
            rnk[c.survivingRoot]--;              // undo the rank bump
        }
        cnt++;
    }

    // Reports whether a and b are currently connected.
    bool connected(int a, int b) {
        return find(a) == find(b);
    }

    // Returns the current number of components.
    int count() {
        return cnt;
    }

    // Returns the number of undoable unions currently on the history stack.
    int historySize() {
        return (int)history.size();
    }
};
```

```cpp
#include <bits/stdc++.h>
using namespace std;

RollbackUnionFind ruf(4);
ruf.unite(0, 1);
ruf.unite(2, 3);
cout << ruf.count() << "\n";          // 2
ruf.unite(1, 2);
cout << ruf.connected(0, 3) << "\n";  // true,  count == 1
ruf.rollback();                        // undo unite(1, 2)
cout << ruf.connected(0, 3) << "\n";  // false, count == 2
```

> A persistent/rollback DSU plugged into a **segment tree over the timeline**
> answers offline dynamic-connectivity queries (edges with [start, end] life
> spans) in **O((n + q) log q · log n)**.

---

## 5. When to pick which variant

| Variant                  | Node type        | `find` cost     | Path compression | Undo support | Typical problems |
|--------------------------|------------------|-----------------|------------------|--------------|------------------|
| `UnionFind` (array)      | ints `[0, n)`    | O(α(n)) amort.  | Yes              | No           | Kruskal MST, #provinces, cycle detection, redundant connection |
| `UnionFind<T>` (generic) | any hashable `T` | O(α(n)) amort.* | Yes              | No           | Accounts merge, string/coordinate grouping, sentence similarity |
| `WeightedUnionFind`      | relational       | O(α(n)) amort.  | Yes (weighted)   | No           | Evaluate Division, equation equality, parity / 2-color checks |
| `RollbackUnionFind`      | ints `[0, n)`    | O(log n)        | No (required)    | Yes          | Offline dynamic connectivity, DSU on tree, MST with edge deletions |

**Rules of thumb**

- Default to the **array `UnionFind`** whenever node IDs are small dense
  integers — it is the fastest and simplest.
- Reach for **`UnionFind<T>`** only when relabeling to integers is awkward;
  otherwise map your labels to `0..n-1` once and use the array version.
- Use **`WeightedUnionFind`** when you must answer *relationships*, not just
  membership.
- Use **`RollbackUnionFind`** only when you genuinely need to remove edges /
  undo merges — you pay an extra `log n` factor and lose path compression.

> **Last Updated:** 2026-06-26
