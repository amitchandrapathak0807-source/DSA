# Graphs — Minimum Spanning Tree and Disjoint Set Union

## Concept: Minimum Spanning Tree and Disjoint Set Union

### What is a Minimum Spanning Tree (MST)?

Given a connected, undirected, weighted graph with `V` vertices, a **spanning tree** is a subgraph that:
- Includes all `V` vertices.
- Is connected.
- Has exactly `V - 1` edges (no cycles — it is a tree).

A graph can have many different spanning trees. The **Minimum Spanning Tree** is the spanning tree whose sum of edge weights is the smallest among all possible spanning trees. MSTs are used in network design (laying cable with minimum cost while connecting every node), clustering, approximation algorithms for NP-hard problems (like TSP), and more.

Two classic greedy algorithms build an MST:

- **Prim's Algorithm**: Grows a single tree from an arbitrary start node. At every step, it picks the cheapest edge that connects a vertex already in the MST to a vertex not yet in the MST. Implemented efficiently with a min-priority queue (heap) of `(weight, vertex)` pairs.
- **Kruskal's Algorithm**: Sorts *all* edges of the graph by weight ascending, then greedily adds each edge to the MST as long as it does not form a cycle with edges already chosen. Cycle detection is done efficiently with the **Disjoint Set Union (DSU)** data structure.

Both algorithms rely on the **cut property**: for any partition of vertices into two disjoint sets, the minimum weight edge crossing the partition must belong to some MST. This is why greedily picking the cheapest safe edge always works.

### Disjoint Set (Union-Find) Data Structure

The **Disjoint Set Union (DSU)**, also called **Union-Find**, maintains a collection of disjoint (non-overlapping) sets and supports two primary operations extremely fast:

- **`Find(x)`**: Returns a representative ("root" or "parent") of the set that element `x` belongs to. Two elements are in the same set if and only if `Find` returns the same root for both.
- **`Union(x, y)`**: Merges the sets containing `x` and `y` into a single set.

A DSU is represented internally with a `parent[]` array (where `parent[i]` initially equals `i`, meaning every node is its own parent/root) and typically a `rank[]` or `size[]` array to keep the resulting trees shallow.

**Naive version**: `Find` walks up `parent[x] -> parent[parent[x]] -> ...` until it reaches a node that is its own parent (the root). `Union` just attaches one root under another. Without any care, these trees can degenerate into long chains, making `Find` take O(N) in the worst case.

Two optimizations bring this down to near O(1):

1. **Union by Rank / Size**: When merging two sets, always attach the root of the *smaller* (by rank/height, or by size/node-count) tree under the root of the *larger* tree. This keeps the overall tree shallow — bounded by O(log N) height in the worst case, instead of letting arbitrary unions create long chains.

2. **Path Compression**: While performing `Find(x)`, as we walk up to the root, we make every node on that path point *directly* to the root. The next time `Find` is called on any of those nodes (or descendants of them), the walk is O(1). This "flattens" the tree over time.

Combining both optimizations gives an amortized time complexity per operation of **O(α(N))**, where α is the **inverse Ackermann function** — a function that grows so slowly it is less than 5 for any practically conceivable input size (up to numbers vastly larger than the number of atoms in the observable universe). For all practical purposes, DSU operations are considered **near O(1)**.

```csharp
public class DisjointSet
{
    private readonly int[] parent;
    private readonly int[] rank;
    private readonly int[] size;

    public DisjointSet(int n)
    {
        parent = new int[n];
        rank = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++)
        {
            parent[i] = i;
            rank[i] = 0;
            size[i] = 1;
        }
    }

    // Find with path compression
    public int Find(int x)
    {
        if (parent[x] != x)
        {
            parent[x] = Find(parent[x]); // path compression: point directly to root
        }
        return parent[x];
    }

    // Union by rank
    public bool UnionByRank(int x, int y)
    {
        int rootX = Find(x);
        int rootY = Find(y);
        if (rootX == rootY) return false; // already in same set -> would form a cycle

        if (rank[rootX] < rank[rootY])
        {
            parent[rootX] = rootY;
        }
        else if (rank[rootY] < rank[rootX])
        {
            parent[rootY] = rootX;
        }
        else
        {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
        return true;
    }

    // Union by size (alternative to rank)
    public bool UnionBySize(int x, int y)
    {
        int rootX = Find(x);
        int rootY = Find(y);
        if (rootX == rootY) return false;

        if (size[rootX] < size[rootY])
        {
            (rootX, rootY) = (rootY, rootX);
        }
        parent[rootY] = rootX;
        size[rootX] += size[rootY];
        return true;
    }

    public bool Connected(int x, int y) => Find(x) == Find(y);
}
```

---

## 1. Prim's Algorithm for Minimum Spanning Tree

**Problem Statement:** Given a connected, undirected, weighted graph with `V` vertices represented as an adjacency list of `(neighbor, weight)` pairs, find the sum of edge weights of a Minimum Spanning Tree that connects all vertices.

**Example:**
- Input: `V = 5`, edges = `[(0,1,2), (0,3,6), (1,2,3), (1,3,8), (1,4,5), (2,4,7), (3,4,9)]`
- Output: `16`
- Explanation: One MST uses edges (0,1,2), (1,2,3), (1,4,5), (0,3,6) — total weight `2+3+5+6=16`, connecting all 5 vertices with 4 edges.

**Approach:** Start from any node (say node `0`), mark it visited, and push all its edges into a min-priority queue keyed by weight. Repeatedly pop the minimum-weight edge; if it leads to an unvisited node, mark that node visited, add the weight to the MST total, and push all edges from the newly visited node into the heap. Continue until all vertices are visited. This greedily grows one connected tree, always extending via the cheapest available edge crossing the visited/unvisited boundary — an application of the cut property.

```csharp
using System;
using System.Collections.Generic;

public class PrimMST
{
    public int SpanningTreeWeight(int v, List<List<(int neighbor, int weight)>> adj)
    {
        var visited = new bool[v];
        // PriorityQueue<TElement, TPriority>: element = node, priority = edge weight
        var pq = new PriorityQueue<int, int>();
        pq.Enqueue(0, 0); // start from node 0 with weight 0

        int mstWeight = 0;

        while (pq.Count > 0)
        {
            var (node, weight) = (pq.Peek(), pq.PeekPriority());
            pq.Dequeue();

            if (visited[node]) continue;
            visited[node] = true;
            mstWeight += weight;

            foreach (var (neighbor, edgeWeight) in adj[node])
            {
                if (!visited[neighbor])
                {
                    pq.Enqueue(neighbor, edgeWeight);
                }
            }
        }

        return mstWeight;
    }
}
```

Note: The standard `System.Collections.Generic.PriorityQueue<TElement,TPriority>` does not expose `PeekPriority()`; the snippet above shows the conceptual API. A drop-in-correct version dequeues via `pq.TryDequeue(out var node, out var weight)`:

```csharp
while (pq.Count > 0)
{
    pq.TryDequeue(out int node, out int weight);
    if (visited[node]) continue;
    visited[node] = true;
    mstWeight += weight;
    foreach (var (neighbor, edgeWeight) in adj[node])
        if (!visited[neighbor]) pq.Enqueue(neighbor, edgeWeight);
}
```

**Time Complexity:** O(E log V) — each edge may be pushed/popped from the heap, heap operations cost O(log E) ~ O(log V).
**Space Complexity:** O(V + E) for the adjacency list, visited array, and heap.

**Explanation:** Dry run on the example graph starting at node `0`:
1. Push `(0, w=0)`. Pop `0` (weight 0), mark visited, `mstWeight = 0`. Push edges from 0: `(1,2)`, `(3,6)`.
2. Pop `(1,2)` — smallest in heap. Node 1 unvisited → visit it, `mstWeight = 2`. Push edges from 1: `(2,3)`, `(3,8)`, `(4,5)`.
3. Heap now has `(3,6)`, `(2,3)`, `(3,8)`, `(4,5)`. Pop `(2,3)` — smallest. Node 2 unvisited → visit, `mstWeight = 5`. Push edges from 2: `(4,7)`.
4. Heap has `(3,6)`, `(3,8)`, `(4,5)`, `(4,7)`. Pop `(3,6)` — smallest. Node 3 unvisited → visit, `mstWeight = 11`. Push edges from 3: `(4,9)`.
5. Heap has `(3,8)` [node 3 already visited, will be skipped], `(4,5)`, `(4,7)`, `(4,9)`. Pop `(4,5)` — smallest among remaining valid entries. Node 4 unvisited → visit, `mstWeight = 16`.
6. All 5 vertices visited. Remaining heap entries for already-visited nodes get popped and skipped. Final MST weight = **16**, matching the example.

---

## 2. Implement Disjoint Set (Union by Rank/Size and Path Compression)

**Problem Statement:** Implement a Disjoint Set Union data structure supporting `Find(x)` (with path compression) and `Union(x, y)` (with union by rank or size), so that after a sequence of unions, queries of the form "are `x` and `y` in the same component?" can be answered efficiently.

**Example:**
- Input: `n = 7` elements (0..6). Operations: `Union(1,2)`, `Union(2,3)`, `Union(4,5)`, `Union(6,7 -> out of range, ignore)`, `Find(1)`, `Find(3)`, `Union(4,3)`, `Find(1)` vs `Find(5)`
- Output: `Find(1) == Find(3)` is `true` before the last union; after `Union(4,3)`, `Find(1) == Find(5)` becomes `true`.
- Explanation: Union operations merge components incrementally; Find reports the common root of a component.

**Approach:** Maintain `parent[]` initialized so `parent[i] = i`, and `rank[]` initialized to 0 (or `size[]` initialized to 1). `Find(x)` recursively follows `parent` pointers to the root and compresses the path by re-pointing every visited node directly to the root. `Union(x, y)` finds both roots; if they differ, it attaches the smaller-rank (or smaller-size) tree under the larger one, breaking ties by incrementing rank. This keeps trees shallow, and combined with path compression yields near O(1) amortized operations.

```csharp
using System;

public class DisjointSet
{
    private readonly int[] parent;
    private readonly int[] rank;
    private readonly int[] size;

    public DisjointSet(int n)
    {
        parent = new int[n];
        rank = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++)
        {
            parent[i] = i;
            rank[i] = 0;
            size[i] = 1;
        }
    }

    public int Find(int x)
    {
        if (parent[x] != x)
        {
            parent[x] = Find(parent[x]); // path compression
        }
        return parent[x];
    }

    public bool UnionByRank(int x, int y)
    {
        int rootX = Find(x), rootY = Find(y);
        if (rootX == rootY) return false;

        if (rank[rootX] < rank[rootY])
        {
            parent[rootX] = rootY;
        }
        else if (rank[rootY] < rank[rootX])
        {
            parent[rootY] = rootX;
        }
        else
        {
            parent[rootY] = rootX;
            rank[rootX]++;
        }
        return true;
    }

    public bool UnionBySize(int x, int y)
    {
        int rootX = Find(x), rootY = Find(y);
        if (rootX == rootY) return false;

        if (size[rootX] < size[rootY]) (rootX, rootY) = (rootY, rootX);
        parent[rootY] = rootX;
        size[rootX] += size[rootY];
        return true;
    }

    public bool Connected(int x, int y) => Find(x) == Find(y);
}
```

**Time Complexity:** O(α(n)) amortized per `Find`/`Union` call, where α is the inverse Ackermann function (effectively constant).
**Space Complexity:** O(n) for the `parent`, `rank`, and `size` arrays.

**Explanation:** Dry run `Union(1,2)`, `Union(2,3)`, `Union(4,5)`, `Union(4,3)` using union-by-rank on `n=7` elements (indices 0..6, all start as their own root with rank 0):
1. `Union(1,2)`: `Find(1)=1`, `Find(2)=2`. Ranks equal (0,0) → attach `parent[2]=1`, `rank[1]++ = 1`.
2. `Union(2,3)`: `Find(2)` walks `2 -> parent[2]=1`, root is `1`; path compression sets `parent[2]=1` (already direct). `Find(3)=3`. `rank[1]=1 > rank[3]=0` → attach `parent[3]=1`.
3. `Union(4,5)`: `Find(4)=4`, `Find(5)=5`, ranks equal → `parent[5]=4`, `rank[4]++ = 1`.
4. `Union(4,3)`: `Find(4)=4`. `Find(3)` walks `3 -> parent[3]=1`, root is `1` (path compression sets `parent[3]=1`, unchanged). `rank[4]=1 == rank[1]=1` → attach `parent[1]=4`, `rank[4]++ = 2`.

After this, calling `Find(1)`: `1 -> parent[1]=4`, root `4`; path compression sets `parent[1]=4`. Calling `Find(5)`: `5 -> parent[5]=4`, root `4` directly (already flat). So `Find(1) == Find(5) == 4` → all of `{1,2,3,4,5}` are now one component, and the parent array has flattened so most nodes point directly (or in one hop) to root `4`, demonstrating how path compression collapses long chains as `Find` is called.

---

## 3. Kruskal's Algorithm for Minimum Spanning Tree (using DSU)

**Problem Statement:** Given a connected, undirected, weighted graph, find the total weight of its Minimum Spanning Tree using Kruskal's algorithm.

**Example:**
- Input: `V = 4`, edges = `[(0,1,10), (0,2,6), (0,3,5), (1,3,15), (2,3,4)]`
- Output: `19`
- Explanation: Sorted edges by weight: (2,3,4), (0,3,5), (0,2,6), (0,1,10), (1,3,15). Accept (2,3,4) [4+2 unconnected], accept (0,3,5) [0+3 unconnected], skip (0,2,6) [0 and 2 already connected via 3 → cycle], accept (0,1,10) [connects last vertex]. Total = 4+5+10 = 19.

**Approach:** Sort all `E` edges by weight ascending. Initialize a DSU with `V` singleton sets. Iterate edges in sorted order; for each edge `(u, v, w)`, check `Find(u) != Find(v)` — if they're in different components, accepting this edge cannot create a cycle, so `Union(u, v)` and add `w` to the MST total. If they're already in the same component (`Find(u) == Find(v)`), adding this edge would form a cycle, so skip it. Stop early once `V-1` edges have been accepted.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class KruskalMST
{
    public int MinimumSpanningTree(int v, List<(int u, int w, int weight)> edges)
    {
        var sortedEdges = edges.OrderBy(e => e.weight).ToList();
        var dsu = new DisjointSet(v);

        int mstWeight = 0;
        int edgesUsed = 0;

        foreach (var (u, w, weight) in sortedEdges)
        {
            if (dsu.Find(u) != dsu.Find(w))
            {
                dsu.UnionByRank(u, w);
                mstWeight += weight;
                edgesUsed++;
                if (edgesUsed == v - 1) break;
            }
        }

        return mstWeight;
    }
}
```

**Time Complexity:** O(E log E) for sorting the edges (dominant term) + O(E · α(V)) for the union-find operations, which is effectively O(E log E).
**Space Complexity:** O(V + E) for the DSU arrays and edge list.

**Explanation:** Dry run on the example graph, `V=4`, sorted edges `[(2,3,4), (0,3,5), (0,2,6), (0,1,10), (1,3,15)]`:
1. `(2,3,4)`: `Find(2)=2`, `Find(3)=3` — different → accept, `Union(2,3)`, `mstWeight=4`, `edgesUsed=1`.
2. `(0,3,5)`: `Find(0)=0`, `Find(3)` = root of {2,3} (say `2`) — different → accept, `Union(0,3)`, `mstWeight=9`, `edgesUsed=2`.
3. `(0,2,6)`: `Find(0)` now resolves to root of {0,2,3} (say `2`), `Find(2)=2` — **same root** → cycle detected → **skip**.
4. `(0,1,10)`: `Find(0)=2` (root), `Find(1)=1` — different → accept, `Union(0,1)`, `mstWeight=19`, `edgesUsed=3 = V-1` → stop.
5. Final MST weight = **19**, matching the expected output. Edge `(0,2,6)` was correctly skipped because `Find` showed 0 and 2 already shared a root after edges `(2,3,4)` and `(0,3,5)` were accepted.

---

## 4. Number of Operations to Make Network Connected

**Problem Statement:** There are `n` computers numbered `0` to `n-1` connected by ethernet cables given as `connections[i] = [a, b]`. Any computer can reach any other via cables directly or indirectly. You can remove a cable between two directly connected computers and use it to connect a pair of computers that are not directly connected. Return the minimum number of operations (cable moves) needed to make all computers connected, or `-1` if impossible.

**Example:**
- Input: `n = 6`, `connections = [[0,1],[0,2],[0,3],[1,2],[1,3]]`
- Output: `2`
- Explanation: There are 5 cables but only 6 nodes need 5 to connect all — however cables 0-2 and 1-2 (and 1-3) are redundant relative to what's needed. We have enough cables (5 >= n-1 = 5) but two separate components exist ({0,1,2,3} and {4}, {5}), so we need 2 operations to connect the 2 extra isolated components.

**Approach:** First, if `connections.Length < n - 1`, it's impossible to connect everything (not enough cables) → return `-1`. Otherwise, use DSU: union every pair in `connections`. Count the number of **redundant** edges (edges where `Find(a) == Find(b)` already, i.e., adding them doesn't reduce component count) — these are the "extra" cables available to redeploy. Then count the number of distinct connected **components** at the end. The answer is `components - 1` (each extra cable can join two components together), which is guaranteed to be `<= redundant edges` when `connections.Length >= n-1`.

```csharp
using System.Collections.Generic;

public class MakeConnected
{
    public int MakeConnectedOps(int n, int[][] connections)
    {
        if (connections.Length < n - 1) return -1;

        var dsu = new DisjointSet(n);
        foreach (var c in connections)
        {
            dsu.UnionByRank(c[0], c[1]);
        }

        var roots = new HashSet<int>();
        for (int i = 0; i < n; i++)
        {
            roots.Add(dsu.Find(i));
        }

        return roots.Count - 1;
    }
}
```

**Time Complexity:** O(E · α(N) + N) for processing connections and counting roots.
**Space Complexity:** O(N) for the DSU arrays and root set.

**Explanation:** Dry run with `n=6`, `connections = [[0,1],[0,2],[0,3],[1,2],[1,3]]`: `connections.Length = 5 >= n-1 = 5`, so proceed. Union all pairs: `0-1`, `0-2`, `0-3`, `1-2` (redundant, 1 and 2 already connected via 0), `1-3` (redundant). After processing, components are `{0,1,2,3}`, `{4}`, `{5}` → 3 distinct roots found via `Find(i)` for `i=0..5`. Answer = `3 - 1 = 2` operations, matching the expected output — take 2 of the redundant cables and use them to connect `{4}` and `{5}` to the main component.

---

## 5. Most Stones Removed with Same Row or Column

**Problem Statement:** Given `n` stones on a 2D plane at integer coordinates `stones[i] = [xi, yi]`, a stone can be removed if it shares a row or a column with another stone that has not yet been removed. Return the maximum number of stones that can be removed.

**Example:**
- Input: `stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]`
- Output: `5`
- Explanation: All stones connected (directly or transitively) via shared rows/columns form one group of 6, and every group of size `k` allows removing `k - 1` stones (leaving exactly one). Here there's a single connected group of size 6, so `6 - 1 = 5` stones can be removed.

**Approach:** Model stones sharing a row or column as connected. Union each stone's row-index and column-index together (using a trick: encode column indices as `col + 10001` or similar offset to keep row-ids and column-ids from colliding in the same DSU space, or simply union stone `i` directly with stone `j` if they share a row/column — but the efficient way is to union each stone with its row id and its column id in a combined DSU of size `rows + cols`). The maximum number of removable stones equals `totalStones - numberOfConnectedComponents`, since each component of size `k` can be reduced to 1 remaining stone (remove `k-1`).

```csharp
using System.Collections.Generic;

public class RemoveStones
{
    public int RemoveStonesMax(int[][] stones)
    {
        const int COL_OFFSET = 10001; // separate row-ids from column-ids
        var dsu = new DisjointSet(20002);
        var usedIds = new HashSet<int>();

        foreach (var stone in stones)
        {
            int row = stone[0];
            int col = stone[1] + COL_OFFSET;
            dsu.UnionByRank(row, col);
            usedIds.Add(row);
            usedIds.Add(col);
        }

        var roots = new HashSet<int>();
        foreach (var id in usedIds)
        {
            roots.Add(dsu.Find(id));
        }

        return stones.Length - roots.Count;
    }
}
```

**Time Complexity:** O(N · α(N)) where N is the number of stones (each stone triggers a constant number of Find/Union calls).
**Space Complexity:** O(N) for the DSU structure sized to the range of row/column ids actually used.

**Explanation:** Dry run on `stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]` (using row-id `r` and column-id `c+10001`):
1. `(0,0)`: union row `0` with col `10001`.
2. `(0,1)`: union row `0` with col `10002` — row 0 already connects col 10001, now also col 10002 joins that component.
3. `(1,0)`: union row `1` with col `10001` — col 10001 is already in the row-0 component, so row 1 joins it too.
4. `(1,2)`: union row `1` with col `10003` — joins the same growing component.
5. `(2,1)`: union row `2` with col `10002` — col 10002 already in the component, row 2 joins.
6. `(2,2)`: union row `2` with col `10003` — already same component (redundant union).

All 6 stones end up mapped to ids that resolve to a **single root**. `roots.Count = 1`. Answer = `6 - 1 = 5`, matching the expected output.

---

## 6. Accounts Merge

**Problem Statement:** Given a list of accounts where `accounts[i] = [name, email1, email2, ...]`, merge accounts that belong to the same person. Two accounts belong to the same person if they share at least one common email. After merging, return each person's account with their name followed by all their emails sorted in ascending order (a person's name is the same across all their accounts).

**Example:**
- Input: `accounts = [["John","johnsmith@mail.com","john_newyork@mail.com"], ["John","johnsmith@mail.com","john00@mail.com"], ["Mary","mary@mail.com"], ["John","johnnybravo@mail.com"]]`
- Output: `[["John","john00@mail.com","john_newyork@mail.com","johnsmith@mail.com"], ["Mary","mary@mail.com"], ["John","johnnybravo@mail.com"]]`
- Explanation: The first and second "John" accounts share `johnsmith@mail.com`, so they merge into one account with all three unique emails, sorted. "Mary" and the third "John" (johnnybravo) share no emails with anyone, so they stay separate.

**Approach:** Treat each account index as a DSU node. For each account, union its index with the index of any other account that shares an email — practically done by mapping each email to the first account index that owns it; when a later account also contains that email, union the current account index with the stored one. After processing all accounts, group account indices by their DSU root, collect the union of all emails for each group, attach the (shared) name, sort emails, and output one merged record per group.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class AccountsMerge
{
    public IList<IList<string>> MergeAccounts(List<List<string>> accounts)
    {
        int n = accounts.Count;
        var dsu = new DisjointSet(n);
        var emailToAccountIndex = new Dictionary<string, int>();

        for (int i = 0; i < n; i++)
        {
            for (int j = 1; j < accounts[i].Count; j++)
            {
                string email = accounts[i][j];
                if (emailToAccountIndex.TryGetValue(email, out int existingIndex))
                {
                    dsu.UnionByRank(i, existingIndex);
                }
                else
                {
                    emailToAccountIndex[email] = i;
                }
            }
        }

        var rootToEmails = new Dictionary<int, SortedSet<string>>();
        for (int i = 0; i < n; i++)
        {
            int root = dsu.Find(i);
            if (!rootToEmails.ContainsKey(root))
            {
                rootToEmails[root] = new SortedSet<string>(StringComparer.Ordinal);
            }
            for (int j = 1; j < accounts[i].Count; j++)
            {
                rootToEmails[root].Add(accounts[i][j]);
            }
        }

        var result = new List<IList<string>>();
        foreach (var kvp in rootToEmails)
        {
            string name = accounts[kvp.Key][0];
            var merged = new List<string> { name };
            merged.AddRange(kvp.Value);
            result.Add(merged);
        }

        return result;
    }
}
```

**Time Complexity:** O(N · K · α(N) + N · K log K) where N is the number of accounts and K is the average number of emails per account (the log K factor from sorting each group's emails via `SortedSet`).
**Space Complexity:** O(N · K) for the email map and grouped email sets.

**Explanation:** Dry run on the example: `emailToAccountIndex` fills as we scan. Account 0 (John): `johnsmith@mail.com -> 0`, `john_newyork@mail.com -> 0`. Account 1 (John): `johnsmith@mail.com` already maps to `0` → `Union(1, 0)`; then `john00@mail.com -> 1`. Account 2 (Mary): `mary@mail.com -> 2`, no unions. Account 3 (John): `johnnybravo@mail.com -> 3`, no unions. Final DSU: accounts `{0,1}` share a root, `{2}` and `{3}` are alone. Grouping emails by root: root(0/1) → `{johnsmith@mail.com, john_newyork@mail.com, john00@mail.com}` sorted → `[john00@mail.com, john_newyork@mail.com, johnsmith@mail.com]` with name "John"; root(2) → `[mary@mail.com]` with name "Mary"; root(3) → `[johnnybravo@mail.com]` with name "John". Matches the expected output.

---

## 7. Number of Islands II (Online Queries)

**Problem Statement:** You are given an `m x n` grid initially all water (`0`). You are given a list of `positions` where `positions[i] = [ri, ci]` represents an operation that turns the cell at `(ri, ci)` into land (`1`). After each operation, return the number of islands (a maximal group of connected `1`s, connected horizontally or vertically) in the grid at that point. Return an array of island counts, one after each query.

**Example:**
- Input: `m=3, n=3, positions = [[0,0],[0,1],[1,2],[2,1]]`
- Output: `[1,1,2,3]`
- Explanation: After `(0,0)` → 1 island. After `(0,1)` → still 1 island (merges with (0,0)). After `(1,2)` → 2 islands (not adjacent to existing land). After `(2,1)` → 3 islands (not adjacent to any existing land at that point).

**Approach:** Use DSU over grid cells (flatten `(r,c) -> r*n+c`). Maintain a `count` of distinct islands and a `land[,]` boolean grid marking which cells are land so far. For each new position: if it's already land, just record the current `count` (LeetCode's version guarantees duplicates report the current count unchanged). Otherwise, mark it as land, increment `count` by 1 (a brand new island), then check its up to 4 neighbors; for each neighbor that is land, if `Find(newCell) != Find(neighbor)`, `Union` them and decrement `count` by 1 (two islands merged into one). After processing all neighbors, append the current `count` to the result list.

```csharp
using System.Collections.Generic;

public class NumberOfIslandsII
{
    public IList<int> NumIslands2(int m, int n, int[][] positions)
    {
        var dsu = new DisjointSet(m * n);
        var isLand = new bool[m, n];
        int count = 0;
        var result = new List<int>();

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        foreach (var pos in positions)
        {
            int r = pos[0], c = pos[1];

            if (isLand[r, c])
            {
                result.Add(count);
                continue;
            }

            isLand[r, c] = true;
            count++;
            int id = r * n + c;

            for (int d = 0; d < 4; d++)
            {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < m && nc >= 0 && nc < n && isLand[nr, nc])
                {
                    int neighborId = nr * n + nc;
                    if (dsu.Find(id) != dsu.Find(neighborId))
                    {
                        dsu.UnionByRank(id, neighborId);
                        count--;
                    }
                }
            }

            result.Add(count);
        }

        return result;
    }
}
```

**Time Complexity:** O(Q · α(M·N)) where Q is the number of queries — each query does O(1) neighbor checks, each involving near-O(1) `Find`/`Union`.
**Space Complexity:** O(M · N) for the DSU arrays and the `isLand` grid.

**Explanation:** Dry run on `m=3,n=3, positions=[[0,0],[0,1],[1,2],[2,1]]`:
1. `(0,0)`: new land, `count=1`. No land neighbors yet. Result so far: `[1]`.
2. `(0,1)`: new land, `count=2`. Check neighbor `(0,0)` — it's land, `Find(0,1)!=Find(0,0)` → union, `count=1`. Result: `[1,1]`.
3. `(1,2)`: new land, `count=2`. Neighbors `(0,2)` water, `(2,2)` water, `(1,1)` water — no unions. Result: `[1,1,2]`.
4. `(2,1)`: new land, `count=3`. Neighbors `(1,1)` water, `(2,0)` water, `(2,2)` water — no unions (note `(1,2)` is diagonal, not adjacent to `(2,1)`). Result: `[1,1,2,3]`, matching expected output.

---

## 8. Making a Large Island (DSU-based approach)

**Problem Statement:** Given an `n x n` binary grid, you may change at most one `0` to `1`. Return the size of the largest island possible after doing so (an island is a group of `1`s connected 4-directionally). This is the same underlying problem as "Max Area of Island" from the Grid topic (problem 8 there), but here it is solved using **DSU** instead of a DFS flood-fill labeling approach.

**Example:**
- Input: `grid = [[1,0],[0,1]]`
- Output: `3`
- Explanation: Flipping either `0` connects to one adjacent `1`, forming an island of size 3 (there is no single flip that touches both existing 1-cells here since they're diagonal, so the best achievable is 3).

**Approach:** First, union every pair of adjacent land (`1`) cells using DSU, so each existing island is one DSU component; track each component's `size[]`. Then, for every water (`0`) cell, look at its up to 4 neighbors, collect the **distinct** roots among land neighbors (using a `HashSet` to avoid double-counting when two neighbors belong to the same island), sum their sizes plus 1 (for the flipped cell itself), and track the maximum such sum across all water cells. If the grid is all land, the answer is `n*n`. If there are no water cells to flip advantageously, also compare against the largest existing island size.

```csharp
using System.Collections.Generic;
using System.Linq;

public class LargestIsland
{
    public int MaxAreaOfIsland(int[][] grid)
    {
        int n = grid.Length;
        var dsu = new DisjointSet(n * n);

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        for (int r = 0; r < n; r++)
        {
            for (int c = 0; c < n; c++)
            {
                if (grid[r][c] == 1)
                {
                    for (int d = 0; d < 4; d++)
                    {
                        int nr = r + dr[d], nc = c + dc[d];
                        if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 1)
                        {
                            dsu.UnionBySize(r * n + c, nr * n + nc);
                        }
                    }
                }
            }
        }

        int best = 0;
        bool hasWater = false;

        for (int r = 0; r < n; r++)
        {
            for (int c = 0; c < n; c++)
            {
                if (grid[r][c] == 0)
                {
                    hasWater = true;
                    var seenRoots = new HashSet<int>();
                    int total = 1; // the flipped cell itself

                    for (int d = 0; d < 4; d++)
                    {
                        int nr = r + dr[d], nc = c + dc[d];
                        if (nr >= 0 && nr < n && nc >= 0 && nc < n && grid[nr][nc] == 1)
                        {
                            int root = dsu.Find(nr * n + nc);
                            if (seenRoots.Add(root))
                            {
                                total += dsu.ComponentSize(root);
                            }
                        }
                    }
                    best = System.Math.Max(best, total);
                }
            }
        }

        if (!hasWater) return n * n;
        return best;
    }
}
```

This requires `DisjointSet` to expose component sizes; add this method to the class shown in the Concept section (and in problem 2):

```csharp
public int ComponentSize(int root) => size[root];
```

**Time Complexity:** O(N² · α(N²)) for scanning the grid and performing near-constant DSU operations per cell.
**Space Complexity:** O(N²) for the DSU arrays.

**Explanation:** Dry run on `grid = [[1,0],[0,1]]` (n=2): First pass unions adjacent land cells — cell `(0,0)` and `(1,1)` are both land but not adjacent (diagonal), so no unions happen; each remains its own component of size 1. Second pass examines water cells: `(0,1)`: neighbors are `(0,0)` [land, root has size 1] and `(1,1)` [land, root has size 1] — two distinct roots → `total = 1 + 1 + 1 = 3`. `(1,0)`: neighbors are `(0,0)` [land, size 1] and `(1,1)` [land, size 1] — distinct roots → `total = 1 + 1 + 1 = 3`. Best = **3**, matching the expected output.

---

## 9. Swim in Rising Water

**Problem Statement:** You are given an `n x n` grid where `grid[r][c]` is the elevation at that cell. Rain starts at time `0`; at time `t`, water level is `t`, and you may swim from a cell to an adjacent (4-directional) cell only if both cells' elevations are `<= t`. Starting at `(0,0)`, find the minimum time `t` such that you can reach `(n-1, n-1)`.

**Example:**
- Input: `grid = [[0,2],[1,3]]`
- Output: `3`
- Explanation: At time `3`, all cells have elevation `<= 3`, so you can traverse `(0,0) -> (1,0) [elev 1] -> (1,1) [elev 3]`. At time `2`, you cannot reach `(1,1)` (elevation 3 > 2), so the answer is 3.

**Approach:** This is conceptually identical to Kruskal's algorithm. Treat every cell as a DSU node, and every adjacency between two cells `(a, b)` as an "edge" whose weight is `max(elevation[a], elevation[b])` — the minimum time at which that edge becomes traversable. Collect all grid-adjacency edges, sort them by this weight ascending, and process them with Union-Find exactly like Kruskal's, unioning cells as edges become available. After each union, check whether `(0,0)` and `(n-1,n-1)` are already connected (`Find` equal); the first edge weight at which they become connected is the answer — because by then, every edge processed so far had weight `<= answer`, meaning a path exists using only cells reachable at that time.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class SwimInRisingWater
{
    public int SwimInWater(int[][] grid)
    {
        int n = grid.Length;
        var dsu = new DisjointSet(n * n);

        var edges = new List<(int a, int b, int weight)>();
        int[] dr = { 0, 1 };
        int[] dc = { 1, 0 };

        for (int r = 0; r < n; r++)
        {
            for (int c = 0; c < n; c++)
            {
                for (int d = 0; d < 2; d++) // only need right + down to cover all adjacencies once
                {
                    int nr = r + dr[d], nc = c + dc[d];
                    if (nr < n && nc < n)
                    {
                        int weight = Math.Max(grid[r][c], grid[nr][nc]);
                        edges.Add((r * n + c, nr * n + nc, weight));
                    }
                }
            }
        }

        edges = edges.OrderBy(e => e.weight).ToList();

        int start = 0;
        int end = n * n - 1;

        if (start == end) return grid[0][0];

        foreach (var (a, b, weight) in edges)
        {
            dsu.UnionByRank(a, b);
            if (dsu.Find(start) == dsu.Find(end))
            {
                return weight;
            }
        }

        return -1; // unreachable (shouldn't happen for a fully filled grid)
    }
}
```

**Time Complexity:** O(N² log N²) dominated by sorting the ~2N² edges, plus O(N² · α(N²)) for the union-find processing.
**Space Complexity:** O(N²) for the DSU arrays and edge list.

**Explanation:** Dry run on `grid = [[0,2],[1,3]]` (n=2), cells indexed `(0,0)=0, (0,1)=1, (1,0)=2, (1,1)=3`. Edges (right/down only): `(0,0)-(0,1)` weight `max(0,2)=2`; `(0,0)-(1,0)` weight `max(0,1)=1`; `(0,1)-(1,1)` weight `max(2,3)=3`; `(1,0)-(1,1)` weight `max(1,3)=3`. Sorted by weight: `[(0,2,w=1), (0,1,w=2), (1,3,w=3), (2,3,w=3)]` (using flattened ids: 0=(0,0), 1=(0,1), 2=(1,0), 3=(1,1)).
1. Process `(0,2,w=1)`: union cells 0 and 2. `Find(0)` vs `Find(3)` — not equal yet.
2. Process `(0,1,w=2)`: union cells 0 and 1. `Find(0)` vs `Find(3)` — still not equal (3 not yet joined).
3. Process `(1,3,w=3)`: union cells 1 and 3 — now 0,1,2,3 all share a root. `Find(0) == Find(3)` → **true** → return weight `3`.

Matches the expected output of **3**, and mirrors Kruskal's cycle-free greedy union process, just re-purposed to find the minimum threshold at which two specific nodes become connected rather than to build a full spanning tree.
