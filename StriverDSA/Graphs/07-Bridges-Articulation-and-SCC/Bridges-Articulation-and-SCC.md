# Graphs — Bridges, Articulation Points, and Strongly Connected Components

## Concept: DFS Timers — Discovery Time and Low-Link Value

All four problems in this file are solved with a single DFS pass augmented with two arrays:

- **`disc[u]`** — the "discovery time" of vertex `u`: a timestamp (a simple incrementing counter) recording the order in which `u` was first visited by the DFS. `disc[u]` is set once, when `u` is first pushed into the recursion, and never changes afterward.
- **`low[u]`** — the "low-link value" of vertex `u`: the smallest `disc[]` value reachable from the subtree rooted at `u` in the DFS tree, **including through at most one back-edge** (an edge from `u`'s subtree to one of `u`'s ancestors). Initially `low[u] = disc[u]`, and it only ever gets smaller as the DFS explores children and back-edges.

`low[u]` is updated in two situations while processing neighbor `v` of `u`:

1. If `v` is an unvisited child (tree edge `u -> v`): after the recursive call returns, `low[u] = min(low[u], low[v])` — `u` inherits whatever `v`'s subtree can reach.
2. If `v` is already visited and is **not** the direct parent of `u` (a back-edge to an ancestor, or a cross edge in the directed case): `low[u] = min(low[u], disc[v])` — note we use `disc[v]`, not `low[v]`, because we are only allowed to "borrow" one back-edge; using `low[v]` here would incorrectly chain through the ancestor's own back-edges.

**Why this detects structure in one O(V+E) pass:**

- **Bridges**: a tree edge `u -> v` is a bridge exactly when nothing in `v`'s subtree can reach `u` or any ancestor of `u` — i.e. `low[v] > disc[u]`. No back-edge escapes the subtree, so removing the edge disconnects it.
- **Articulation points**: a vertex `u` is a cut vertex if removing it disconnects some subtree from the rest of the graph. For the root, this happens if it has 2+ DFS-tree children (each child's subtree can only reach the rest of the graph through the root). For a non-root `u`, this happens if some child `v` satisfies `low[v] >= disc[u]` — `v`'s subtree cannot reach any ancestor of `u`, so `u` is the sole connector.
- **SCCs**: `low[]` (as reachability information) generalizes to directed graphs. Kosaraju's uses finishing times plus a graph reversal to isolate SCCs; Tarjan's uses `low[]` directly with a stack of "on-stack" vertices — whenever `low[u] == disc[u]`, `u` is the root of an SCC, and everything above it on the stack belongs to that SCC.

Because each vertex and edge is visited a constant number of times (once to discover, once to relax `low[]` per adjacent edge, and O(1) stack push/pop for Tarjan's), all four algorithms run in **O(V + E)** time and **O(V)** extra space for the timer arrays and the recursion/explicit stack.

---

## 1. Bridges in a Graph

**Problem Statement:** Given an undirected graph, find all "bridges" — edges whose removal increases the number of connected components (equivalently, edges not lying on any cycle).

**Example:**
- Input: Vertices `0..4`, Edges: `(0,1), (1,2), (2,0), (1,3), (3,4)`
- Output: Bridges = `[(1,3), (3,4)]`
- Explanation: `0-1-2-0` forms a triangle (a cycle), so none of its edges are bridges — removing any one of them still leaves the triangle's vertices connected via the other two edges. But `1-3` and `3-4` are the only paths connecting `3` and `4` to the rest of the graph; removing either splits the graph into more pieces.

**Approach:** Run a single DFS maintaining `disc[]` and `low[]`. When exploring edge `u -> v`:
- If `v` is unvisited, recurse (tree edge), then update `low[u] = min(low[u], low[v])`. After the recursive call returns, check the bridge condition: if `low[v] > disc[u]`, the edge `(u, v)` is a bridge (no back-edge from `v`'s subtree reaches `u` or higher).
- If `v` is visited and is not the parent we came from, it's a back-edge: `low[u] = min(low[u], disc[v])`.

We must be careful with the "parent" check when the graph has parallel edges — track the specific edge (or its index) used to enter `u`, not just the parent vertex, to avoid treating a duplicate edge as a back-edge.

```csharp
using System;
using System.Collections.Generic;

public class BridgesFinder
{
    private int timer = 0;
    private int[] disc, low;
    private bool[] visited;
    private List<List<int>> adj;
    private List<(int, int)> bridges;

    public List<(int, int)> FindBridges(int n, List<List<int>> adj)
    {
        this.adj = adj;
        disc = new int[n];
        low = new int[n];
        visited = new bool[n];
        bridges = new List<(int, int)>();

        for (int i = 0; i < n; i++)
        {
            if (!visited[i])
                Dfs(i, -1);
        }
        return bridges;
    }

    private void Dfs(int u, int parentEdgeVertex)
    {
        visited[u] = true;
        disc[u] = low[u] = timer++;
        int parentSkipped = 0; // handles at most one parallel edge to parent correctly

        foreach (int v in adj[u])
        {
            if (v == parentEdgeVertex && parentSkipped == 0)
            {
                parentSkipped = 1;
                continue;
            }

            if (!visited[v])
            {
                Dfs(v, u);
                low[u] = Math.Min(low[u], low[v]);

                if (low[v] > disc[u])
                    bridges.Add((u, v));
            }
            else
            {
                low[u] = Math.Min(low[u], disc[v]);
            }
        }
    }
}
```

Time Complexity: O(V+E) — every vertex is visited once and every edge is examined a constant number of times.
Space Complexity: O(V) for `disc[]`, `low[]`, `visited[]`, and the recursion stack (plus O(E) to store adjacency and bridges output).

**Explanation:** Dry run on the example graph `0-1, 1-2, 2-0, 1-3, 3-4` starting DFS at `0`:

1. Visit `0`: `disc[0]=low[0]=0`.
2. Go to neighbor `1` (tree edge): visit `1`: `disc[1]=low[1]=1`.
3. From `1`, go to neighbor `2` (tree edge): visit `2`: `disc[2]=low[2]=2`.
4. From `2`, neighbor `0` is visited and is not `2`'s direct parent (`1` is) — it's a back-edge: `low[2] = min(low[2], disc[0]) = min(2, 0) = 0`.
5. `2` has no more neighbors; return to `1`. Update `low[1] = min(low[1], low[2]) = min(1, 0) = 0`. Check bridge: `low[2]=0 > disc[1]=1`? No — so `(1,2)` is not a bridge.
6. From `1`, neighbor `0` is visited but is `1`'s parent — skip (via `parentSkipped`).
7. From `1`, go to neighbor `3` (tree edge): visit `3`: `disc[3]=low[3]=3`.
8. From `3`, go to neighbor `4` (tree edge): visit `4`: `disc[4]=low[4]=4`. `4` has no unvisited neighbors and no back-edges (its only neighbor `3` is its parent — skipped); return to `3`.
9. Update `low[3] = min(low[3], low[4]) = min(3, 4) = 3`. Check bridge: `low[4]=4 > disc[3]=3`? Yes — **`(3,4)` is a bridge.**
10. `3` has no more neighbors; return to `1`. Update `low[1] = min(low[1], low[3]) = min(0, 3) = 0`. Check bridge: `low[3]=3 > disc[1]=1`? Yes — **`(1,3)` is a bridge.**
11. Return to `0`. Update `low[0] = min(low[0], low[1]) = min(0, 0) = 0`. Check bridge: `low[1]=0 > disc[0]=0`? No — `(0,1)` is not a bridge.

Final bridges found: `(3,4)` and `(1,3)`, matching the expected output.

---

## 2. Articulation Points in a Graph

**Problem Statement:** Given an undirected graph, find all "articulation points" (cut vertices) — vertices whose removal (along with all incident edges) increases the number of connected components.

**Example:**
- Input: Vertices `0..4`, Edges: `(0,1), (1,2), (2,0), (1,3), (3,4)`
- Output: Articulation points = `[1, 3]`
- Explanation: Removing vertex `1` disconnects `{3, 4}` from `{0, 2}`. Removing vertex `3` disconnects `{4}` from the rest. Removing `0`, `2`, or `4` does not disconnect the graph, since the `0-1-2` triangle keeps `0` and `2` connected via `2` even without `1` directly, and `4` is a leaf.

**Approach:** Same DFS with `disc[]`/`low[]`, plus a `childCount` for the DFS root. For non-root vertex `u` with a tree-edge child `v`: after recursing, if `low[v] >= disc[u]`, then `u` is an articulation point (removing `u` cuts `v`'s subtree off from any ancestor of `u`, since the subtree cannot reach past `u`). The DFS root is a special case — it has no ancestors, so it's an articulation point only if it has **2 or more** children in the DFS tree (each child's subtree connects to the others only through the root). Use `Math.Min(low[u], disc[v])` for back-edges as before.

```csharp
using System;
using System.Collections.Generic;

public class ArticulationPointsFinder
{
    private int timer = 0;
    private int[] disc, low;
    private bool[] visited, isArticulation;
    private List<List<int>> adj;

    public List<int> FindArticulationPoints(int n, List<List<int>> adj)
    {
        this.adj = adj;
        disc = new int[n];
        low = new int[n];
        visited = new bool[n];
        isArticulation = new bool[n];

        for (int i = 0; i < n; i++)
        {
            if (!visited[i])
                Dfs(i, -1);
        }

        List<int> result = new List<int>();
        for (int i = 0; i < n; i++)
            if (isArticulation[i]) result.Add(i);
        return result;
    }

    private void Dfs(int u, int parent)
    {
        visited[u] = true;
        disc[u] = low[u] = timer++;
        int children = 0;
        int parentSkipped = 0;

        foreach (int v in adj[u])
        {
            if (v == parent && parentSkipped == 0)
            {
                parentSkipped = 1;
                continue;
            }

            if (!visited[v])
            {
                children++;
                Dfs(v, u);
                low[u] = Math.Min(low[u], low[v]);

                if (parent != -1 && low[v] >= disc[u])
                    isArticulation[u] = true;
            }
            else
            {
                low[u] = Math.Min(low[u], disc[v]);
            }
        }

        if (parent == -1 && children >= 2)
            isArticulation[u] = true;
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V) for the timer/flag arrays and recursion stack.

**Explanation:** Using the same graph as above (root DFS at `0`): the `disc`/`low` values computed in problem 1's dry run apply identically here.

- At vertex `3` (non-root, parent `1`): its child `4` has `low[4]=4 >= disc[3]=3`, so `3` is marked an articulation point.
- At vertex `1` (non-root, parent `0`): its child `3` has `low[3]=3 >= disc[1]=1`, so `1` is marked an articulation point. (Its other "child" `2` was actually reached and its low value updated `1`'s low, but `2` is not a tree child of `1` in this DFS order since `2` gets discovered via `1 -> 2` directly as a tree edge too — check: from `1`, neighbor `2` is unvisited at that point since DFS visits `2` from `1` directly in adjacency order before backtracking... in our specific traversal above `2` was visited as a child of `1` as well, with `low[2]=0 < disc[1]=1`, so that child does NOT trigger the articulation condition for `1`. `1` is still marked as an articulation point because of its other child `3`.)
- At the root `0`: it has only 1 DFS-tree child (`1`; `2` was reached via a back-edge, not as a fresh tree child), so the root condition (`children >= 2`) is not met — `0` is not an articulation point.

Final articulation points: `{1, 3}`, matching the expected output.

---

## 3. Kosaraju's Algorithm for Strongly Connected Components (SCC)

**Problem Statement:** Given a directed graph, find all Strongly Connected Components — maximal sets of vertices where every vertex can reach every other vertex in the same set via directed paths.

**Example:**
- Input: Vertices `0..4`, directed edges: `0->1, 1->2, 2->0, 1->3, 3->4`
- Output: SCCs = `[{0, 1, 2}, {3}, {4}]`
- Explanation: `0, 1, 2` form a cycle (`0->1->2->0`), so each can reach the others — one SCC. `3` can reach `4` but `4` cannot reach back to `3` (no return edge), so they are singleton SCCs each.

**Approach:** Kosaraju's algorithm uses two DFS passes:
1. Run DFS on the original graph, and push each vertex onto a stack when it **finishes** (i.e., in post-order / finishing-time order).
2. Reverse all edges of the graph (transpose graph).
3. Pop vertices off the stack one at a time; for each unvisited vertex, run DFS on the **reversed** graph — every vertex reached in this DFS call belongs to the same SCC as the popped vertex.

Why this works: processing vertices in decreasing finish-time order on the transpose graph ensures that when we start a new DFS, the vertex we start from cannot be reached by any not-yet-visited vertex's SCC in a way that would incorrectly merge components — the finish-time ordering guarantees each DFS call on the transpose exactly carves out one SCC.

```csharp
using System;
using System.Collections.Generic;

public class KosarajuSCC
{
    public List<List<int>> FindSCCs(int n, List<List<int>> adj)
    {
        bool[] visited = new bool[n];
        Stack<int> finishOrder = new Stack<int>();

        // Step 1: order vertices by finish time
        for (int i = 0; i < n; i++)
            if (!visited[i])
                FillOrder(i, adj, visited, finishOrder);

        // Step 2: build transpose graph
        List<List<int>> transpose = new List<List<int>>();
        for (int i = 0; i < n; i++) transpose.Add(new List<int>());
        for (int u = 0; u < n; u++)
            foreach (int v in adj[u])
                transpose[v].Add(u);

        // Step 3: DFS on transpose in finish-time order
        Array.Clear(visited, 0, n);
        List<List<int>> sccs = new List<List<int>>();

        while (finishOrder.Count > 0)
        {
            int u = finishOrder.Pop();
            if (!visited[u])
            {
                List<int> component = new List<int>();
                DfsCollect(u, transpose, visited, component);
                sccs.Add(component);
            }
        }
        return sccs;
    }

    private void FillOrder(int u, List<List<int>> adj, bool[] visited, Stack<int> finishOrder)
    {
        visited[u] = true;
        foreach (int v in adj[u])
            if (!visited[v])
                FillOrder(v, adj, visited, finishOrder);
        finishOrder.Push(u);
    }

    private void DfsCollect(int u, List<List<int>> transpose, bool[] visited, List<int> component)
    {
        visited[u] = true;
        component.Add(u);
        foreach (int v in transpose[u])
            if (!visited[v])
                DfsCollect(v, transpose, visited, component);
    }
}
```

Time Complexity: O(V+E) — two DFS passes plus building the transpose, each O(V+E). Space Complexity: O(V+E) for the transpose graph and O(V) for the stack/visited arrays.

**Explanation:** Dry run on the example graph: edges `0->1, 1->2, 2->0, 1->3, 3->4`.

**First DFS (original graph), starting at `0`, pushing to stack on finish:**
- Visit `0` -> visit `1` -> visit `2` -> `2`'s only edge goes to `0` (already visiting/visited), so `2` finishes first: push `2`.
- Back to `1`: next edge `1->3` -> visit `3` -> `3->4` -> visit `4`: `4` has no outgoing edges, finishes: push `4`.
- Back to `3`: no more edges, finishes: push `3`.
- Back to `1`: no more edges, finishes: push `1`.
- Back to `0`: no more edges, finishes: push `0`.
- Finish-order stack (top to bottom, last pushed on top): `0, 1, 3, 4, 2`.

**Reverse the graph:** original edges `0->1, 1->2, 2->0, 1->3, 3->4` become transpose edges `1->0, 2->1, 0->2, 3->1, 4->3`.

**Second DFS (on transpose), popping in order `0, 1, 3, 4, 2`:**
- Pop `0`: unvisited. DFS from `0` on transpose: `0 -> 2` (visit `2`), `2 -> 1` (visit `1`), `1 -> 0` (already visited). Component collected: `{0, 2, 1}`. This is SCC #1: `{0, 1, 2}`.
- Pop `1`: already visited, skip.
- Pop `3`: unvisited. DFS from `3` on transpose: `3 -> 1` (already visited, skip). Component: `{3}`. This is SCC #2: `{3}`.
- Pop `4`: unvisited. DFS from `4` on transpose: `4 -> 3` (already visited, skip). Component: `{4}`. This is SCC #3: `{4}`.
- Pop `2`: already visited, skip.

Final SCCs: `{0, 1, 2}`, `{3}`, `{4}` — matching the expected output.

---

## 4. Tarjan's Algorithm for Strongly Connected Components (SCC)

**Problem Statement:** Given a directed graph, find all Strongly Connected Components using a single DFS pass (as opposed to Kosaraju's two-pass approach).

**Example:**
- Input: Vertices `0..4`, directed edges: `0->1, 1->2, 2->0, 1->3, 3->4`
- Output: SCCs = `[{0, 1, 2}, {3}, {4}]`
- Explanation: Same graph as problem 3 — `0, 1, 2` form a cycle (one SCC), while `3` and `4` are each their own SCC since neither has a path back to an earlier vertex that would close a cycle.

**Approach:** Tarjan's algorithm runs a single DFS maintaining `disc[]`, `low[]`, an explicit `Stack<int>` of vertices currently "on the stack" (active, meaning they are in the current DFS path or connected to it and not yet assigned to a finalized SCC), and a `bool[] onStack` for O(1) membership checks. When visiting `u`:
- Push `u` onto the stack, set `disc[u] = low[u] = timer++`, mark `onStack[u] = true`.
- For each neighbor `v`: if unvisited, recurse then `low[u] = min(low[u], low[v])`. If `v` is visited **and on the stack**, it's part of the current active component, so `low[u] = min(low[u], disc[v])` (note: only if `onStack[v]` — if `v` was visited but already popped off in a finalized earlier SCC, it must be ignored, since it belongs to a different, already-closed SCC).
- After processing all neighbors of `u`, check if `low[u] == disc[u]`. If so, `u` is the root of an SCC: pop vertices off the stack (marking `onStack = false`) until `u` itself is popped — all popped vertices form one SCC.

```csharp
using System;
using System.Collections.Generic;

public class TarjanSCC
{
    private int timer = 0;
    private int[] disc, low;
    private bool[] onStack, visited;
    private Stack<int> stack;
    private List<List<int>> adj;
    private List<List<int>> sccs;

    public List<List<int>> FindSCCs(int n, List<List<int>> adj)
    {
        this.adj = adj;
        disc = new int[n];
        low = new int[n];
        visited = new bool[n];
        onStack = new bool[n];
        stack = new Stack<int>();
        sccs = new List<List<int>>();

        for (int i = 0; i < n; i++)
            if (!visited[i])
                Dfs(i);

        return sccs;
    }

    private void Dfs(int u)
    {
        visited[u] = true;
        disc[u] = low[u] = timer++;
        stack.Push(u);
        onStack[u] = true;

        foreach (int v in adj[u])
        {
            if (!visited[v])
            {
                Dfs(v);
                low[u] = Math.Min(low[u], low[v]);
            }
            else if (onStack[v])
            {
                low[u] = Math.Min(low[u], disc[v]);
            }
        }

        if (low[u] == disc[u])
        {
            List<int> component = new List<int>();
            int w;
            do
            {
                w = stack.Pop();
                onStack[w] = false;
                component.Add(w);
            } while (w != u);
            sccs.Add(component);
        }
    }
}
```

Time Complexity: O(V+E) — single DFS pass, each vertex pushed/popped from the stack at most once. Space Complexity: O(V) for `disc[]`, `low[]`, `onStack[]`, the explicit stack, and the recursion stack.

**Explanation:** This is naturally paired with Kosaraju's dry run above (problem 3) for comparison — Tarjan's finds the same SCCs in one pass instead of two. A brief trace on the same graph (`0->1, 1->2, 2->0, 1->3, 3->4`), starting DFS at `0`:

1. Visit `0`: `disc[0]=low[0]=0`, push `0` onto stack → `[0]`.
2. Visit `1` (via `0->1`): `disc[1]=low[1]=1`, push `1` → `[0,1]`.
3. Visit `2` (via `1->2`): `disc[2]=low[2]=2`, push `2` → `[0,1,2]`.
4. From `2`, edge `2->0`: `0` is visited and `onStack[0]=true`, so `low[2] = min(2, disc[0]) = min(2,0) = 0`.
5. `2` has no more edges. Check: `low[2]=0 == disc[2]=2`? No — not an SCC root yet, return to `1`.
6. Back in `1`: `low[1] = min(low[1], low[2]) = min(1, 0) = 0`.
7. From `1`, edge `1->3`: unvisited, visit `3`: `disc[3]=low[3]=3`, push `3` → `[0,1,2,3]`.
8. From `3`, edge `3->4`: unvisited, visit `4`: `disc[4]=low[4]=4`, push `4` → `[0,1,2,3,4]`. `4` has no outgoing edges. Check: `low[4]=4 == disc[4]=4`? Yes — SCC root! Pop stack until `4` is popped: pop `4`. SCC found: `{4}`. Stack now `[0,1,2,3]`.
9. Back in `3`: `low[3] = min(low[3], low[4]) = min(3,4) = 3`. No more edges. Check: `low[3]=3 == disc[3]=3`? Yes — SCC root! Pop `3`. SCC found: `{3}`. Stack now `[0,1,2]`.
10. Back in `1`: `low[1] = min(low[1], low[3]) = min(0,3) = 0`. No more edges. Check: `low[1]=0 == disc[1]=1`? No — return to `0`.
11. Back in `0`: `low[0] = min(low[0], low[1]) = min(0,0) = 0`. No more edges. Check: `low[0]=0 == disc[0]=0`? Yes — SCC root! Pop until `0`: pop `2`, pop `1`, pop `0`. SCC found: `{2, 1, 0}`.

Final SCCs: `{4}`, `{3}`, `{0, 1, 2}` — the same three components Kosaraju's found, produced in a single DFS pass.
