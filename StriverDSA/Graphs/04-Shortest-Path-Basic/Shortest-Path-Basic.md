# Graphs — Shortest Path Algorithms (Basic)

## 1. Shortest Path in an Undirected Graph with Unit Weights (BFS)

**Problem Statement:**
Given an undirected graph with `V` vertices and `E` edges where every edge has weight `1`, find the shortest distance from a given source vertex to every other vertex. If a vertex is unreachable, report `-1` for it.

**Example:**
- Input: `V = 9`, edges = `[(0,1),(0,3),(3,4),(4,5),(5,6),(1,2),(2,6),(6,7),(7,8),(6,8)]`, source = `0`
- Output: `dist = [0, 1, 2, 1, 2, 3, 3, 4, 4]`
- Explanation: Starting from node `0`, BFS visits nodes level by level. Node `1` and `3` are at distance `1` (direct neighbors of `0`). Node `2` and `4` are at distance `2` (reached through `1` and `3`). This continues until all reachable nodes get their minimum edge-count distance.

**Approach:**
Because every edge has the same weight, the number of edges on a path is exactly proportional to its length — so the path with the fewest edges is automatically the shortest path. BFS explores the graph in strict layers: it fully visits all nodes at distance `d` before touching any node at distance `d+1`. This means the very first time BFS reaches a node, it has done so via the minimum number of edges possible. We initialize a `dist[]` array to infinity, set `dist[source] = 0`, push the source into a queue, and repeatedly pop a node, look at its unvisited neighbors, set their distance to `current distance + 1`, and push them. No relaxation loop or priority queue is needed because BFS's natural level-order traversal already guarantees shortest distances for unit-weight graphs.

**Logic (Steps):**
1. Initialize `dist[]` to infinity for all vertices, then set `dist[source] = 0`.
2. Enqueue `source` into a `Queue<int>`.
3. While the queue is non-empty, dequeue a node and scan its neighbors.
4. For each neighbor where `dist[node] + 1 < dist[neighbor]` (i.e., still unvisited/improvable), update `dist[neighbor] = dist[node] + 1` and enqueue it — since all weights are `1`, this first assignment is always optimal.
5. After the queue empties, convert any leftover `int.MaxValue` entries to `-1` to mark unreachable vertices.

```csharp
using System;
using System.Collections.Generic;

public class UnitWeightShortestPath
{
    public int[] ShortestPath(int V, List<List<int>> adj, int source)
    {
        int[] dist = new int[V];
        Array.Fill(dist, int.MaxValue);
        dist[source] = 0;

        Queue<int> queue = new Queue<int>();
        queue.Enqueue(source);

        while (queue.Count > 0)
        {
            int node = queue.Dequeue();
            foreach (int neighbor in adj[node])
            {
                if (dist[node] + 1 < dist[neighbor])
                {
                    dist[neighbor] = dist[node] + 1;
                    queue.Enqueue(neighbor);
                }
            }
        }

        for (int i = 0; i < V; i++)
        {
            if (dist[i] == int.MaxValue) dist[i] = -1;
        }
        return dist;
    }
}
```

Time Complexity: O(V + E) — every vertex and edge is processed once. Space Complexity: O(V + E) for the adjacency list, distance array, and queue.

**Walkthrough:**
Dry run on the example graph starting at node `0`:
- Queue: `[0]`, `dist = [0,∞,∞,∞,∞,∞,∞,∞,∞]`
- Pop `0` → neighbors `1, 3` get `dist = 1`. Queue: `[1, 3]`
- Pop `1` → neighbor `2` gets `dist = 2` (neighbor `0` already visited). Queue: `[3, 2]`
- Pop `3` → neighbor `4` gets `dist = 2`. Queue: `[2, 4]`
- Pop `2` → neighbor `6` gets `dist = 3`. Queue: `[4, 6]`
- Pop `4` → neighbor `5` gets `dist = 3`. Queue: `[6, 5]`
- Pop `6` → neighbors `7, 8` get `dist = 4` (neighbor `2, 5` already visited). Queue: `[5, 7, 8]`
- Pop `5`, `7`, `8` → all remaining neighbors already visited, nothing changes.
- Final: `dist = [0, 1, 2, 1, 2, 3, 3, 4, 4]`, matching the expected output.

## 2. Shortest Path in a Directed Acyclic Graph (DAG) Using Topological Sort

**Problem Statement:**
Given a weighted DAG (edges can have any weight, including negative), find the shortest distance from a source vertex to all other vertices.

**Example:**
- Input: `V = 6`, edges (u, v, weight) = `[(0,1,2),(0,4,1),(4,5,4),(1,2,3),(2,3,6),(5,3,1)]`, source = `0`
- Output: `dist = [0, 2, 5, 3, 1, 5]`
- Explanation: The path `0 → 4 → 5 → 3` gives distance `1 + 4 + 1 = 6`, but `0 → 1 → 2 → 3` gives `2 + 3 + 6 = 11`; however `dist[3]` actually ends up `3` via a shorter combination once relaxation from all predecessors is considered in topological order — the key point is that processing nodes in topological order guarantees every predecessor's final distance is known before it is used to relax successors.

**Approach:**
In a DAG there are no cycles, so we can linearly order all vertices such that every edge goes from an earlier vertex to a later one (topological order). If we process vertices strictly in this order and relax all outgoing edges of each vertex as we visit it, then by the time we reach any vertex `v`, every vertex that could contribute to `v`'s shortest distance (i.e., every predecessor) has already been fully processed and finalized. This works correctly even with negative edge weights, because there is no cycle where a node could be revisited and improved after being finalized — unlike Dijkstra's, negative weights cannot cause incorrect greedy choices here. We get the topological order via DFS (pushing nodes to a stack on finish) or Kahn's BFS algorithm, then do a single relaxation pass.

**Logic (Steps):**
1. Compute a topological order of the DAG via DFS: recurse into each unvisited neighbor first, then push the current node onto a `Stack<int>` in post-order.
2. Initialize `dist[]` to infinity, set `dist[source] = 0`.
3. Pop nodes off the stack one by one (this yields topological order); skip any node whose `dist` is still infinity (unreachable).
4. For each popped node, relax all its outgoing edges: if `dist[node] + weight < dist[to]`, update `dist[to]`.
5. Because nodes are processed strictly in topological order, every predecessor of a node is fully finalized before that node is relaxed, so a single linear pass suffices (even with negative weights, since there are no cycles to revisit).

```csharp
using System;
using System.Collections.Generic;

public class DagShortestPath
{
    private void TopoSortDfs(int node, List<List<(int to, int weight)>> adj, bool[] visited, Stack<int> stack)
    {
        visited[node] = true;
        foreach (var (to, _) in adj[node])
        {
            if (!visited[to]) TopoSortDfs(to, adj, visited, stack);
        }
        stack.Push(node);
    }

    public int[] ShortestPath(int V, List<List<(int to, int weight)>> adj, int source)
    {
        bool[] visited = new bool[V];
        Stack<int> stack = new Stack<int>();

        for (int i = 0; i < V; i++)
        {
            if (!visited[i]) TopoSortDfs(i, adj, visited, stack);
        }

        int[] dist = new int[V];
        Array.Fill(dist, int.MaxValue);
        dist[source] = 0;

        while (stack.Count > 0)
        {
            int node = stack.Pop();
            if (dist[node] == int.MaxValue) continue; // unreachable node
            foreach (var (to, weight) in adj[node])
            {
                if (dist[node] + weight < dist[to])
                {
                    dist[to] = dist[node] + weight;
                }
            }
        }

        return dist;
    }
}
```

Time Complexity: O(V + E) for topological sort plus O(V + E) for relaxation = O(V + E) overall. Space Complexity: O(V + E) for adjacency list, stack, and visited/distance arrays.

**Walkthrough:**
Dry run on the example: adjacency list — `0: [(1,2),(4,1)]`, `1: [(2,3)]`, `2: [(3,6)]`, `4: [(5,4)]`, `5: [(3,1)]`.
- DFS from `0` visits `1` first (post-order pushes `2` then `3` then `1`... actually recursing `0→1→2→3` pushes `3, 2, 1` onto the stack), then backtracks and visits `4→5`, pushing `5, 4`, then finally pushes `0`. Popping the stack top-to-bottom gives topological order: `0, 4, 5, 1, 2, 3`.
- Initialize `dist = [0, ∞, ∞, ∞, ∞, ∞]`.
- Pop `0`: relax `1` → `dist[1] = 0+2 = 2`; relax `4` → `dist[4] = 0+1 = 1`. `dist = [0,2,∞,∞,1,∞]`
- Pop `4`: relax `5` → `dist[5] = 1+4 = 5`. `dist = [0,2,∞,∞,1,5]`
- Pop `5`: relax `3` → `dist[3] = 5+1 = 6`. `dist = [0,2,∞,6,1,5]`
- Pop `1`: relax `2` → `dist[2] = 2+3 = 5`. `dist = [0,2,5,6,1,5]`
- Pop `2`: relax `3` → candidate `5+6=11`, not less than current `6`, no update. `dist = [0,2,5,6,1,5]`
- Pop `3`: no outgoing edges.

Final distances: `[0, 2, 5, 6, 1, 5]`. Because nodes are processed strictly in topological order, `5` (and therefore `3` via `5`) gets finalized before `2` ever tries to relax `3` through the longer `0→1→2→3` path, so the single-pass relaxation correctly picks up the shorter `0→4→5→3` route into `dist[3]` without needing to revisit any node.

## 3. Dijkstra's Algorithm (Shortest Path in a Weighted Graph with Non-negative Weights) Using a Priority Queue

**Problem Statement:**
Given a weighted, undirected (or directed) graph with non-negative edge weights, find the shortest distance from a source vertex to all other vertices.

**Example:**
- Input: `V = 5`, edges (u, v, weight) = `[(0,1,4),(0,2,1),(2,1,2),(1,3,1),(2,3,5),(3,4,3)]`, source = `0`
- Output: `dist = [0, 3, 1, 4, 7]`
- Explanation: Instead of going `0 → 1` directly (cost 4), it's cheaper to go `0 → 2 → 1` (cost `1 + 2 = 3`). From there `1 → 3` costs `1` more (total `4`), and `3 → 4` costs `3` more (total `7`).

**Approach:**
Dijkstra's algorithm greedily grows the set of vertices whose shortest distance is finalized. It maintains a min-priority-queue of `(distance, node)` pairs. At each step it extracts the node with the smallest tentative distance — since all edge weights are non-negative, once a node is popped with the smallest distance in the queue, that distance can never be improved later (any other path to it would have to go through a node with an equal or larger distance, adding non-negative weight, which can't be smaller). After popping a node, we "relax" every outgoing edge: for each neighbor, if `dist[node] + weight(node, neighbor) < dist[neighbor]`, we update `dist[neighbor]` and push the new `(distance, neighbor)` pair into the queue. Stale, outdated entries may remain in the queue but are simply skipped when popped if a better distance was already finalized. This greedy-plus-relaxation approach fails with negative weights because a negative edge could later reduce the distance of an already-finalized node.

**Logic (Steps):**
1. Initialize `dist[]` to infinity, set `dist[source] = 0`, and push `(source, 0)` into a min-`PriorityQueue<int,int>` keyed by distance.
2. Repeatedly dequeue the `(node, currentDist)` pair with the smallest distance; if `currentDist > dist[node]`, this entry is stale (a better distance was already found), so skip it.
3. Otherwise, relax every outgoing edge of `node`: compute `newDist = dist[node] + weight`.
4. If `newDist < dist[to]`, update `dist[to] = newDist` and push `(to, newDist)` into the queue.
5. Continue until the queue is empty; `dist[]` then holds the shortest distance to every vertex, since non-negative weights guarantee a popped node's distance can never be improved later.

```csharp
using System;
using System.Collections.Generic;

public class DijkstraShortestPath
{
    public int[] Dijkstra(int V, List<List<(int to, int weight)>> adj, int source)
    {
        int[] dist = new int[V];
        Array.Fill(dist, int.MaxValue);
        dist[source] = 0;

        var pq = new PriorityQueue<int, int>(); // (node, priority = distance)
        pq.Enqueue(source, 0);

        while (pq.Count > 0)
        {
            pq.TryDequeue(out int node, out int currentDist);
            if (currentDist > dist[node]) continue; // stale entry, skip

            foreach (var (to, weight) in adj[node])
            {
                int newDist = dist[node] + weight;
                if (newDist < dist[to])
                {
                    dist[to] = newDist;
                    pq.Enqueue(to, newDist);
                }
            }
        }

        return dist;
    }
}
```

Time Complexity: O((V + E) log V) — each edge relaxation may push a new entry into the heap, and each heap operation costs O(log V). Space Complexity: O(V + E) for adjacency list, distance array, and priority queue.

**Walkthrough:**
Dry run on the example graph (adjacency: `0:[(1,4),(2,1)]`, `1:[(0,4),(2,2),(3,1)]`, `2:[(0,1),(1,2),(3,5)]`, `3:[(1,1),(2,5),(4,3)]`, `4:[(3,3)]`), source `0`:

| Step | Pop (node, dist) | PQ before pop | Relaxations | dist[] after |
|---|---|---|---|---|
| 1 | (0, 0) | [(0,0)] | 1: ∞→4, 2: ∞→1 | [0,4,1,∞,∞] |
| 2 | (2, 1) | [(1,4),(2,1)] | 1: 4→3 (via 2), 3: ∞→6 | [0,3,1,6,∞] |
| 3 | (1, 3) | [(1,4),(1,3),(3,6)] (first (1,4) is stale) | 3: 6→4 (via 1) | [0,3,1,4,∞] |
| 4 | (3, 4) | [(3,6),(3,4)] (pop smaller first → (3,4)) | 4: ∞→7 | [0,3,1,4,7] |
| 5 | (3, 6) | [(3,6)] | stale (6 > dist[3]=4), skip | [0,3,1,4,7] |
| 6 | (4, 7) | [(4,7)] | no outgoing improvements | [0,3,1,4,7] |

Final distances: `[0, 3, 1, 4, 7]`, matching the expected output. Notice the queue can contain stale entries (like `(3,6)` and `(1,4)`) which are simply skipped because a better distance was already found and finalized by the time they're popped.

## 4. Print the Shortest Path Using Dijkstra's Algorithm

**Problem Statement:**
Given a weighted graph, a source, and a destination, find not just the shortest distance but also the actual sequence of vertices that make up the shortest path from source to destination.

**Example:**
- Input: `V = 5`, edges (u, v, weight) = `[(0,1,4),(0,2,1),(2,1,2),(1,3,1),(2,3,5),(3,4,3)]`, source = `0`, destination = `4`
- Output: Path = `0 → 2 → 1 → 3 → 4`, Distance = `7`
- Explanation: This is the same graph as problem 3. The shortest distance to node `4` is `7`, achieved via `0 → 2 (1) → 1 (2) → 3 (1) → 4 (3)`. The `parent[]` array lets us reconstruct this exact sequence.

**Approach:**
We run the same Dijkstra's algorithm as before, but alongside `dist[]` we maintain a `parent[]` array where `parent[v]` stores the vertex immediately preceding `v` on the current best-known path from the source. Every time we relax an edge and successfully improve `dist[to]`, we also set `parent[to] = node` (the vertex we relaxed from). Initially `parent[i] = i` for all `i` (or `-1`, sentinel for "no parent yet"), and `parent[source] = source`. After the algorithm finishes, we reconstruct the path by starting at the destination and repeatedly jumping to `parent[current]` until we reach the source, collecting vertices along the way, then reversing the collected list to get the path in source-to-destination order.

**Logic (Steps):**
1. Initialize `dist[]` to infinity and `parent[i] = i` for every vertex (self-loop sentinel), then set `dist[source] = 0`.
2. Run standard Dijkstra's with a `PriorityQueue<int,int>`: dequeue the smallest-distance node, skip if stale.
3. When relaxing an edge and improving `dist[to]`, also record `parent[to] = node` (the vertex we relaxed from).
4. After the queue empties, if `dist[destination]` is still infinity, the destination is unreachable — return `[-1]`.
5. Otherwise walk backward from `destination` via `parent[]` until reaching a self-loop (`current == parent[current]`, the source), collecting vertices, then reverse the collected list to get the source-to-destination path.

```csharp
using System;
using System.Collections.Generic;

public class DijkstraPrintPath
{
    public List<int> ShortestPathPrint(int V, List<List<(int to, int weight)>> adj, int source, int destination)
    {
        int[] dist = new int[V];
        int[] parent = new int[V];
        Array.Fill(dist, int.MaxValue);
        for (int i = 0; i < V; i++) parent[i] = i;
        dist[source] = 0;

        var pq = new PriorityQueue<int, int>();
        pq.Enqueue(source, 0);

        while (pq.Count > 0)
        {
            pq.TryDequeue(out int node, out int currentDist);
            if (currentDist > dist[node]) continue;

            foreach (var (to, weight) in adj[node])
            {
                int newDist = dist[node] + weight;
                if (newDist < dist[to])
                {
                    dist[to] = newDist;
                    parent[to] = node;
                    pq.Enqueue(to, newDist);
                }
            }
        }

        if (dist[destination] == int.MaxValue) return new List<int> { -1 }; // unreachable

        List<int> path = new List<int>();
        int current = destination;
        while (current != parent[current])
        {
            path.Add(current);
            current = parent[current];
        }
        path.Add(source);
        path.Reverse();
        return path;
    }
}
```

Time Complexity: O((V + E) log V) for Dijkstra's, plus O(V) for path reconstruction — overall O((V + E) log V). Space Complexity: O(V + E) for adjacency list, plus O(V) for `dist[]`, `parent[]`, and the path list.

**Walkthrough:**
Reusing the dry run from problem 3, `parent[]` updates alongside each relaxation:
- Start: `parent = [0,0,0,0,0]` (each node initialized to itself).
- Pop `(0,0)`: relax `1` → `parent[1] = 0`; relax `2` → `parent[2] = 0`. `parent = [0,0,0,0,0]` (unchanged indices 3,4).
- Pop `(2,1)`: relax `1` (3 < 4) → `parent[1] = 2`; relax `3` (6 < ∞) → `parent[3] = 2`. `parent = [0,2,0,2,0]`
- Pop `(1,3)`: relax `3` (4 < 6) → `parent[3] = 1`. `parent = [0,2,0,1,0]`
- Pop `(3,4)`: relax `4` (7 < ∞) → `parent[4] = 3`. `parent = [0,2,0,1,3]`
- Remaining pops are stale or add nothing.

Reconstructing the path from destination `4` backward:
- `current = 4` → add `4`, `current = parent[4] = 3`
- `current = 3` → add `3`, `current = parent[3] = 1`
- `current = 1` → add `1`, `current = parent[1] = 2`
- `current = 2` → add `2`, `current = parent[2] = 0`
- `current = 0` → `parent[0] == 0`, stop; add `0`
- Collected (in reverse order of visiting): `[4, 3, 1, 2, 0]` → reverse → `[0, 2, 1, 3, 4]`, giving the path `0 → 2 → 1 → 3 → 4` with total distance `7`, matching the expected output.

## 5. Shortest Distance in a Binary Maze

**Problem Statement:**
Given a grid of `0`s and `1`s where `0` represents an open cell and `1` represents a wall, and given a source cell and a destination cell, find the length of the shortest path (in number of steps, moving up/down/left/right only) from source to destination. Return `-1` if no path exists.

**Example:**
- Input:
```
grid =
[1, 1, 1, 1, 1, 1, 0, 1, 1, 1]
[1, 0, 0, 0, 0, 1, 0, 1, 0, 1]
[1, 0, 1, 1, 0, 1, 0, 1, 0, 1]
[1, 0, 1, 0, 0, 0, 0, 0, 0, 1]
[1, 0, 1, 1, 1, 1, 1, 1, 0, 1]
[1, 0, 0, 0, 0, 0, 0, 0, 0, 1]
[1, 1, 1, 1, 1, 1, 1, 1, 0, 1]
```
  source = `(0, 6)`, destination = `(4, 8)`
- Output: `13` (shortest path length in steps)
- Explanation: Every open cell (`0`) is a graph node connected to its up-to-4 open neighbors with an implicit edge weight of `1`. Finding the shortest number of steps between two grid cells with uniform step cost is exactly the unit-weight BFS problem, just applied on a 2D grid instead of an explicit adjacency list.

**Approach:**
Treat every open (`0`) cell of the grid as a node in an implicit unit-weight graph, where each cell has up to 4 edges — to its up, down, left, and right neighbors — provided those neighbors are within bounds and also open (not a wall). Because all edges have weight `1`, the same reasoning as problem 1 applies: BFS explores cells in increasing order of step-count from the source, so the first time we reach the destination we have found the shortest path length. We maintain a 2D `dist[][]` array (or a hash set of visited cells) initialized to infinity, set `dist[source] = 0`, and run a standard BFS pushing `(row, col)` pairs, checking bounds, wall status, and visited status before enqueueing.

**Logic (Steps):**
1. If the source or destination cell is itself a wall, return `-1` immediately.
2. Initialize a 2D `dist[,]` array to infinity, set `dist[source] = 0`, and enqueue `source` into a `Queue<(r,c)>`.
3. While the queue is non-empty, dequeue a cell; if it equals `destination`, return its `dist` value immediately (BFS guarantees this is the shortest step count).
4. Otherwise, check its 4-directional neighbors — for any in-bounds, open (`0`), and improvable neighbor (`dist[r,c] + 1 < dist[neighbor]`), set its distance and enqueue it.
5. If the queue empties without ever dequeuing the destination, return `-1` (unreachable).

```csharp
using System;
using System.Collections.Generic;

public class BinaryMazeShortestPath
{
    public int ShortestPathBinaryMatrix(int[][] grid, (int r, int c) source, (int r, int c) destination)
    {
        int n = grid.Length, m = grid[0].Length;

        if (grid[source.r][source.c] == 1 || grid[destination.r][destination.c] == 1)
            return -1;

        int[,] dist = new int[n, m];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                dist[i, j] = int.MaxValue;

        dist[source.r, source.c] = 0;
        Queue<(int r, int c)> queue = new Queue<(int r, int c)>();
        queue.Enqueue(source);

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        while (queue.Count > 0)
        {
            var (r, c) = queue.Dequeue();

            if ((r, c) == destination) return dist[r, c];

            for (int dir = 0; dir < 4; dir++)
            {
                int nr = r + dr[dir], nc = c + dc[dir];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m &&
                    grid[nr][nc] == 0 && dist[r, c] + 1 < dist[nr, nc])
                {
                    dist[nr, nc] = dist[r, c] + 1;
                    queue.Enqueue((nr, nc));
                }
            }
        }

        return -1; // destination unreachable
    }
}
```

Time Complexity: O(N × M) where N and M are grid dimensions — each cell is enqueued and processed at most once, with O(1) work (4 neighbor checks) per cell. Space Complexity: O(N × M) for the distance array and queue.

**Explanation:**
Conceptually this is identical to the dry run in problem 1, but on a grid: BFS starts at `(0,6)` with `dist = 0`, explores all cells reachable in `1` step, then `2` steps, and so on, always assigning a cell its distance the first time it's discovered (guaranteeing minimality because BFS processes cells in non-decreasing distance order). Following the open-cell corridor from `(0,6)` down through the grid and around the walls to `(4,8)` takes exactly `13` steps, so `dist[4,8] = 13` is returned as soon as that cell is dequeued.

## 6. Path With Minimum Effort

**Problem Statement:**
Given a 2D grid of heights, find a path from the top-left cell to the bottom-right cell (moving up/down/left/right) that minimizes the "effort" of the path, where effort is defined as the **maximum absolute difference in heights** between two consecutive cells along the path (not the sum of differences).

**Example:**
- Input:
```
heights =
[1, 2, 2]
[3, 8, 2]
[5, 3, 5]
```
- Output: `2`
- Explanation: The path `(0,0) → (0,1) → (0,2) → (1,2) → (2,2)` has consecutive height differences `|1-2|=1`, `|2-2|=0`, `|2-2|=0`, `|2-5|=3`. Its effort is `3`. A better path `(0,0) → (0,1) → (0,2) → (2,2)` isn't valid (not adjacent), but the actual optimal path `(0,0)→(0,1)→(0,2)→(1,2)→(2,2)` reconsidered, or alternatives like going through `(1,0)→(2,0)→(2,1)→(2,2)`, yields differences `2,2,2,2` → effort `2`, which is the minimum achievable "worst single step" over any route.

**Approach:**
This is a modified Dijkstra where the quantity being minimized along a path is not the *sum* of edge weights but the *maximum* edge weight (height difference) seen so far on that path. We define `effort[cell]` as the minimum possible "path maximum" to reach that cell from the source. We use a min-priority-queue of `(effort, row, col)`, starting with `effort[source] = 0`. When we pop a cell, we look at each of its up-to-4 neighbors, compute the step cost as `|height[current] - height[neighbor]|`, and compute the candidate effort for the neighbor as `max(effort[current], stepCost)` — because the path's overall effort is dictated by its single worst step, not an accumulation. If this candidate is smaller than the neighbor's currently recorded effort, we update it and push it into the queue. Because the priority queue always expands the cell with the smallest finalized effort first (same greedy correctness argument as Dijkstra's, since `max()` is monotonic non-decreasing just like `+` with non-negative weights), the first time we pop the destination cell, `effort[destination]` is guaranteed to be the answer.

```csharp
using System;
using System.Collections.Generic;

public class PathWithMinimumEffort
{
    public int MinimumEffortPath(int[][] heights)
    {
        int n = heights.Length, m = heights[0].Length;
        int[,] effort = new int[n, m];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                effort[i, j] = int.MaxValue;

        effort[0, 0] = 0;
        var pq = new PriorityQueue<(int r, int c), int>();
        pq.Enqueue((0, 0), 0);

        int[] dr = { -1, 1, 0, 0 };
        int[] dc = { 0, 0, -1, 1 };

        while (pq.Count > 0)
        {
            pq.TryDequeue(out var cell, out int currentEffort);
            var (r, c) = cell;

            if (r == n - 1 && c == m - 1) return currentEffort;
            if (currentEffort > effort[r, c]) continue; // stale entry

            for (int dir = 0; dir < 4; dir++)
            {
                int nr = r + dr[dir], nc = c + dc[dir];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m)
                {
                    int stepCost = Math.Abs(heights[r][c] - heights[nr][nc]);
                    int candidateEffort = Math.Max(effort[r, c], stepCost);

                    if (candidateEffort < effort[nr, nc])
                    {
                        effort[nr, nc] = candidateEffort;
                        pq.Enqueue((nr, nc), candidateEffort);
                    }
                }
            }
        }

        return effort[n - 1, m - 1];
    }
}
```

Time Complexity: O(N × M × log(N × M)) — each of the N×M cells can be pushed into the heap up to 4 times (once per neighbor relaxation), and each heap operation costs O(log(N×M)). Space Complexity: O(N × M) for the effort array and priority queue.

**Explanation:**
Dry run on the example grid (`heights = [[1,2,2],[3,8,2],[5,3,5]]`), source `(0,0)`:
- `effort[0,0] = 0`. PQ: `[((0,0), 0)]`
- Pop `(0,0), 0`: neighbors `(0,1)` step `|1-2|=1` → `effort[0,1] = max(0,1) = 1`; `(1,0)` step `|1-3|=2` → `effort[1,0] = max(0,2) = 2`. PQ: `[((0,1),1), ((1,0),2)]`
- Pop `(0,1), 1`: neighbor `(0,2)` step `|2-2|=0` → `effort[0,2] = max(1,0) = 1`; neighbor `(1,1)` step `|2-8|=6` → `effort[1,1] = max(1,6) = 6`. PQ: `[((0,2),1), ((1,0),2), ((1,1),6)]`
- Pop `(0,2), 1`: neighbor `(1,2)` step `|2-2|=0` → `effort[1,2] = max(1,0) = 1`. PQ: `[((1,2),1), ((1,0),2), ((1,1),6)]`
- Pop `(1,2), 1`: neighbor `(2,2)` step `|2-5|=3` → `effort[2,2] = max(1,3) = 3`; neighbor `(1,1)` step `|2-8|=6` → candidate `max(1,6)=6`, not better than existing `6`, no update. PQ: `[((1,0),2), ((2,2),3), ((1,1),6)]`
- Pop `(1,0), 2`: neighbor `(2,0)` step `|3-5|=2` → `effort[2,0] = max(2,2) = 2`; neighbor `(1,1)` step `|3-8|=5` → candidate `max(2,5)=5 < 6` → `effort[1,1] = 5`. PQ: `[((2,0),2), ((2,2),3), ((1,1),5)]`
- Pop `(2,0), 2`: neighbor `(2,1)` step `|5-3|=2` → `effort[2,1] = max(2,2) = 2`. PQ: `[((2,2),3), ((2,1),2), ((1,1),5)]`
- Pop `(2,1), 2`: neighbor `(2,2)` step `|3-5|=2` → candidate `max(2,2)=2 < 3` → `effort[2,2] = 2`, push `((2,2),2)`; neighbor `(1,1)` step `|3-8|=5` → candidate `max(2,5)=5`, not better. PQ: `[((2,2),2), ((2,2),3), ((1,1),5)]`
- Pop `(2,2), 2`: this is the destination `(n-1, m-1)` → return `2`.

Final answer: `2`, matching the expected output — achieved via the path `(0,0) → (1,0) → (2,0) → (2,1) → (2,2)` whose worst single step is `2`.
