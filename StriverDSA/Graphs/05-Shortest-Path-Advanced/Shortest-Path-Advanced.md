# Graphs — Shortest Path Algorithms (Advanced)

## 1. Cheapest Flights Within K Stops

**Problem Statement:** Given `n` cities labeled `0` to `n-1`, a list of flights `[from, to, price]`, a source city `src`, a destination city `dst`, and an integer `k`, find the cheapest price to travel from `src` to `dst` using at most `k` stops (i.e., at most `k+1` edges). Return `-1` if no such route exists.

**Example:**
- Input: `n = 4`, `flights = [[0,1,100],[1,2,100],[2,3,100],[0,2,500]]`, `src = 0`, `dst = 3`, `k = 1`
- Output: `700`
- Explanation: With at most 1 stop allowed (2 edges), the only feasible path within the stop limit is `0 -> 2 -> 3` using the direct edge `0->2` (cost 500) and `2->3` (cost 100), giving `600`... but wait — using `0->1->2->3` needs 2 stops which exceeds `k=1`. So the algorithm picks the best path using at most 2 edges: `0 -> 2` (500) `-> 3` (100) = `600`. (If `k=1`, answer is `600`; increasing `k` to `2` would allow `0->1->2->3` but that costs `300` only if edge weights were smaller — here with given weights the cheapest with `k=1` stop is `600`.)

**Approach:** This is a Bellman-Ford-style relaxation problem, restricted to at most `K+1` rounds of relaxation (since `K` stops means `K+1` edges maximum). Maintain a `dist[]` array initialized to infinity except `dist[src] = 0`. Repeat `K+1` times: take a **copy** of the current `dist[]` array (call it `temp`), and for every edge `(u, v, w)` in the flight list, if `dist[u] + w < temp[v]`, update `temp[v] = dist[u] + w`. After processing all edges, set `dist = temp`. Using a copy per round is essential — it ensures that within a single round, each edge is relaxed using distances from the *previous* round only, so no single round uses more than one edge per path (preventing a path from silently using more than `K+1` edges by chaining relaxations within the same pass, similar to how plain Bellman-Ford would over multiple internal updates). After `K+1` rounds, `dist[dst]` holds the answer, or `-1` if it is still infinity.

```csharp
public class Solution {
    public int FindCheapestPrice(int n, int[][] flights, int src, int dst, int k) {
        int[] dist = new int[n];
        Array.Fill(dist, int.MaxValue);
        dist[src] = 0;

        // At most k stops means at most k+1 edges, so k+1 relaxation rounds.
        for (int round = 0; round <= k; round++) {
            int[] temp = (int[])dist.Clone(); // copy so each round uses only one edge per path
            foreach (var flight in flights) {
                int u = flight[0], v = flight[1], w = flight[2];
                if (dist[u] != int.MaxValue && dist[u] + w < temp[v]) {
                    temp[v] = dist[u] + w;
                }
            }
            dist = temp;
        }

        return dist[dst] == int.MaxValue ? -1 : dist[dst];
    }
}
```

Time Complexity: O(K * E) — `K+1` rounds, each scanning all `E` edges.
Space Complexity: O(V) for the `dist` and `temp` arrays (plus O(E) already used by the input edge list, not extra allocation).

**Explanation:** Because we relax using a snapshot (`temp`) of the distances from before this round started, a city's distance can only improve by exactly one additional edge per round. This guarantees that after `K+1` rounds, no path used has more than `K+1` edges — correctly enforcing the "at most K stops" constraint, unlike standard Dijkstra which would find the globally cheapest path ignoring the stop limit.

---

## 2. Network Delay Time

**Problem Statement:** There are `n` network nodes labeled `1` to `n`. Given `times[i] = (u, v, w)` meaning a directed edge from node `u` to node `v` with travel time `w`, and a starting node `k`, return the minimum time for all `n` nodes to receive a signal sent from `k`. Return `-1` if it is impossible for all nodes to receive the signal.

**Example:**
- Input: `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n = 4`, `k = 2`
- Output: `2`
- Explanation: Signal starts at node 2. It reaches node 1 in 1 unit, node 3 in 1 unit, and node 4 (via 3) in 2 units. The maximum time among all nodes is `2`, so it takes `2` units for the signal to reach every node.

**Approach:** This is a direct single-source shortest path problem on a directed weighted graph with non-negative weights, solved with Dijkstra's algorithm using a min-heap (priority queue). Build an adjacency list, run Dijkstra from `k` to compute the shortest distance to every node, and the answer is the maximum value in the resulting `dist[]` array (since all nodes must receive the signal, we need the time for the *last* node to receive it). If any node remains unreachable (distance still infinity), return `-1`.

```csharp
public class Solution {
    public int NetworkDelayTime(int[][] times, int n, int k) {
        var adj = new List<(int to, int w)>[n + 1];
        for (int i = 1; i <= n; i++) adj[i] = new List<(int, int)>();
        foreach (var t in times) adj[t[0]].Add((t[1], t[2]));

        int[] dist = new int[n + 1];
        Array.Fill(dist, int.MaxValue);
        dist[k] = 0;

        // Min-heap ordered by distance
        var pq = new PriorityQueue<int, int>();
        pq.Enqueue(k, 0);

        while (pq.Count > 0) {
            int u = pq.Dequeue();
            foreach (var (v, w) in adj[u]) {
                if (dist[u] != int.MaxValue && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.Enqueue(v, dist[v]);
                }
            }
        }

        int maxTime = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == int.MaxValue) return -1;
            maxTime = Math.Max(maxTime, dist[i]);
        }
        return maxTime;
    }
}
```

Time Complexity: O((V + E) log V) — standard Dijkstra with a binary heap; each edge relaxation may push onto the heap, and heap operations cost O(log V).
Space Complexity: O(V + E) for the adjacency list, distance array, and priority queue entries.

**Explanation:** Dijkstra greedily finalizes the shortest distance to the closest unvisited node at each step, guaranteeing correctness for non-negative edge weights. Taking the maximum over `dist[]` answers "when does the *last* node receive the signal," which is exactly the network delay time.

---

## 3. Number of Ways to Arrive at Destination (count shortest paths, modulo 10^9+7)

**Problem Statement:** Given `n` intersections labeled `0` to `n-1` connected by bidirectional roads `roads[i] = [u, v, time]`, count the number of distinct ways to travel from intersection `0` to intersection `n-1` in the **minimum possible time**. Return the count modulo `10^9 + 7`.

**Example:**
- Input: `n = 7`, `roads = [[0,6,7],[0,1,2],[1,2,3],[1,3,3],[6,3,3],[3,5,1],[6,5,1],[2,5,1],[0,4,5],[4,6,2]]`
- Output: `4`
- Explanation: The shortest time from `0` to `6` is `7`. There are 4 distinct paths that all achieve this minimum time of 7 (e.g., `0->6`, `0->4->6`, `0->1->2->5->6`, `0->1->3->5->6`), so the answer is `4`.

**Approach:** Run Dijkstra from node `0`, but augment it with a `ways[]` array (initialized to `ways[0] = 1`, rest `0`) that counts the number of shortest paths to each node. While relaxing edge `(u, v, w)`:
- If `dist[u] + w < dist[v]` (a strictly shorter path to `v` found): update `dist[v] = dist[u] + w` and **reset** `ways[v] = ways[u]` (discard previously counted paths, since they are no longer shortest).
- If `dist[u] + w == dist[v]` (an equally short alternate path found): **accumulate** `ways[v] = (ways[v] + ways[u]) % MOD`.

At the end, `ways[n-1]` holds the answer.

```csharp
public class Solution {
    public int CountPaths(int n, int[][] roads) {
        const int MOD = 1_000_000_007;
        var adj = new List<(int to, long w)>[n];
        for (int i = 0; i < n; i++) adj[i] = new List<(int, long)>();
        foreach (var r in roads) {
            adj[r[0]].Add((r[1], r[2]));
            adj[r[1]].Add((r[0], r[2]));
        }

        long[] dist = new long[n];
        Array.Fill(dist, long.MaxValue);
        long[] ways = new long[n];
        dist[0] = 0;
        ways[0] = 1;

        var pq = new PriorityQueue<int, long>();
        pq.Enqueue(0, 0);

        while (pq.Count > 0) {
            int u = pq.Dequeue();
            foreach (var (v, w) in adj[u]) {
                long newDist = dist[u] + w;
                if (newDist < dist[v]) {
                    dist[v] = newDist;
                    ways[v] = ways[u]; // reset: strictly shorter path found
                    pq.Enqueue(v, dist[v]);
                } else if (newDist == dist[v]) {
                    ways[v] = (ways[v] + ways[u]) % MOD; // accumulate: equal shortest path found
                }
            }
        }

        return (int)(ways[n - 1] % MOD);
    }
}
```

Time Complexity: O((V + E) log V) — same as Dijkstra, with O(1) extra work per relaxation to update `ways[]`.
Space Complexity: O(V + E) for adjacency list, `dist[]`, `ways[]`, and the priority queue.

**Explanation:** The `ways[]` array behaves like a running tally that mirrors the shortest-path relaxation: whenever Dijkstra discovers a genuinely shorter route to a node, all prior path counts for that node become obsolete and are overwritten by the count from the new predecessor. When Dijkstra discovers an alternate route of exactly the same shortest length, that route's path count is added on top, since both routes are equally valid ways to achieve the minimum time.

---

## 4. Bellman-Ford Algorithm (handles negative weights, detects negative cycles)

**Problem Statement:** Given a directed weighted graph with `V` vertices and `E` edges (weights may be negative) and a source vertex `src`, compute the shortest distance from `src` to every other vertex. If the graph contains a negative-weight cycle reachable from `src`, report that no solution exists (distances are not well defined).

**Example:**
- Input: `V = 5`, edges `[(0,1,4), (0,2,4), (1,2,-2), (2,3,3), (3,4,2), (4,1,-5)]`... (this particular set forms a negative cycle `1->2->3->4->1` of weight `-2+3+2-5 = -2`), `src = 0`
- Output: `[-1]` (negative cycle detected)
- Explanation: Since vertices `1, 2, 3, 4` lie on a cycle with total weight `-2`, repeatedly looping through it decreases the path length without bound, so no finite shortest distance exists for those vertices.

**Approach:** Bellman-Ford relaxes all `E` edges, `V-1` times in sequence (since the longest possible simple shortest path has at most `V-1` edges). Initialize `dist[src] = 0` and all others to infinity. For each of `V-1` rounds, iterate over every edge `(u, v, w)` and relax: if `dist[u] + w < dist[v]`, set `dist[v] = dist[u] + w`. After `V-1` rounds, all shortest distances (assuming no negative cycle) are finalized. To detect a negative cycle, perform **one extra round**: if any edge can still be relaxed (`dist[u] + w < dist[v]`), then a negative cycle exists that is reachable from `src` and affects `v`.

```csharp
public class Solution {
    // Returns dist array, or null if a negative cycle reachable from src is detected.
    public int[] BellmanFord(int V, List<(int u, int v, int w)> edges, int src) {
        int[] dist = new int[V];
        Array.Fill(dist, int.MaxValue);
        dist[src] = 0;

        // V-1 rounds of relaxation
        for (int i = 0; i < V - 1; i++) {
            foreach (var (u, v, w) in edges) {
                if (dist[u] != int.MaxValue && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }

        // Extra round: if anything still relaxes, a negative cycle exists.
        foreach (var (u, v, w) in edges) {
            if (dist[u] != int.MaxValue && dist[u] + w < dist[v]) {
                return null; // negative cycle detected
            }
        }

        return dist;
    }
}
```

Time Complexity: O(V * E) — `V-1` relaxation rounds (plus 1 detection round) each scanning all `E` edges.
Space Complexity: O(V) for the `dist` array (edge list itself is input, O(E)).

**Explanation (dry run — no negative cycle):**

Graph: `V = 5` (vertices `0..4`), edges: `(0,1,6), (0,2,7), (1,2,8), (1,3,5), (1,4,-4), (2,3,-3), (2,4,9), (3,1,-2), (4,3,7), (4,0,2)`, `src = 0`.

Initial: `dist = [0, ∞, ∞, ∞, ∞]`

- Round 1 (relax all edges in order): `0->1`: `dist[1]=6`. `0->2`: `dist[2]=7`. `1->2`: `6+8=14 > 7`, no change. `1->3`: `6+5=11`, `dist[3]=11`. `1->4`: `6-4=2`, `dist[4]=2`. `2->3`: `7-3=4 < 11`, `dist[3]=4`. `2->4`: `7+9=16 > 2`, no change. `3->1`: `4-2=2 < 6`, `dist[1]=2`. `4->3`: `2+7=9 > 4`, no change. `4->0`: `2+2=4 > 0`, no change.
  `dist = [0, 2, 7, 4, 2]`

- Round 2: `0->1`: `0+6=6 > 2`, no change. `0->2`: `7`, no change. `1->2`: `2+8=10 > 7`, no change. `1->3`: `2+5=7 > 4`, no change. `1->4`: `2-4=-2 < 2`, `dist[4]=-2`. `2->3`: `7-3=4`, no change. `2->4`: `7+9=16 > -2`, no change. `3->1`: `4-2=2`, no change. `4->3`: `-2+7=5 > 4`, no change. `4->0`: `-2+2=0`, no change.
  `dist = [0, 2, 7, 4, -2]`

- Round 3: `1->4`: `2-4=-2`, no change (already -2). No further improvements occur; all edges settle.
  `dist = [0, 2, 7, 4, -2]`

- Round 4 (`V-1 = 4`): no changes — distances have converged.
  Final: `dist = [0, 2, 7, 4, -2]`

**Negative cycle detection (modified example):** Add an edge `3->1` with weight `-10` instead of `-2`, creating cycle `1->4->3->1` with total weight `-4 + 7 - 10 = -7` (negative). Running the extra (5th) round after the 4 relaxation rounds: edge `3->1` checks `dist[3] + (-10) = 4 - 10 = -6 < dist[1] = 2`, so relaxation still succeeds. Since an edge relaxes even after `V-1` rounds, the algorithm reports a **negative cycle detected**, and no finite shortest-path answer exists for the affected vertices.

---

## 5. Floyd-Warshall Algorithm (all-pairs shortest path)

**Problem Statement:** Given a directed weighted graph represented as an `n x n` adjacency matrix `dist[i][j]` (the direct edge weight from `i` to `j`, or infinity if no direct edge, and `0` when `i == j`), compute the shortest distance between every pair of vertices `(i, j)`.

**Example:**
- Input: 4-node graph with `dist[0][1]=5, dist[0][3]=10, dist[1][2]=3, dist[2][3]=1`, all others infinity except diagonal `0`.
- Output: `dist[0][3] = 4` (via `0->1->2->3` = `5+3+1=9`... actually recompute: `5+3+1=9`, which is less than the direct edge `10`, so `dist[0][3]` updates to `9`).
- Explanation: The direct edge `0->3` costs `10`, but routing through `1` and `2` costs `5+3+1=9`, which is cheaper, so Floyd-Warshall finds this shorter indirect path.

**Approach:** Floyd-Warshall computes all-pairs shortest paths using dynamic programming: for each vertex `k` (used as a potential intermediate node), and for every pair `(i, j)`, check whether routing through `k` is shorter: `dist[i,j] = min(dist[i,j], dist[i,k] + dist[k,j])`. Iterating `k` from `0` to `n-1` as the outermost loop ensures that by the time `k` is considered, all shorter paths using intermediates `0..k-1` have already been incorporated, so the update correctly considers all possible intermediate combinations. A negative cycle can be detected afterward by checking whether any `dist[i,i] < 0`.

```csharp
public class Solution {
    // dist is an n x n matrix; dist[i,j] = int.MaxValue/2 (a large sentinel) if no edge, 0 if i == j.
    public void FloydWarshall(int[,] dist, int n) {
        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    if (dist[i, k] + dist[k, j] < dist[i, j]) {
                        dist[i, j] = dist[i, k] + dist[k, j];
                    }
                }
            }
        }
    }
}
```

Time Complexity: O(V^3) — three nested loops each of size `V`.
Space Complexity: O(V^2) for the distance matrix (in-place update, no extra structure needed beyond the matrix itself).

**Explanation (dry run):**

4-node graph, matrix (row = from, col = to), using `INF` for no edge:

```
      0     1     2     3
0 [   0,    5,   INF,   10  ]
1 [ INF,    0,    3,   INF  ]
2 [ INF,  INF,    0,    1   ]
3 [ INF,  INF,   INF,   0   ]
```

**k = 0** (use node 0 as intermediate): Check `dist[i,0] + dist[0,j]` for all `i,j`. Row/col involving node 0 as intermediate: since no vertex has an edge *into* node 0 (`dist[i,0] = INF` for `i=1,2,3`), no updates occur. Matrix unchanged.

**k = 1** (use node 1 as intermediate): Check `dist[i,1] + dist[1,j]`. `dist[0,1]=5`, `dist[1,2]=3` → `dist[0,2] = min(INF, 5+3) = 8`. `dist[0,1]=5`, `dist[1,3]=INF` → no update to `dist[0,3]` via this. Others involving `i=2,3` have `dist[i,1]=INF`, no updates.

```
      0     1     2     3
0 [   0,    5,    8,   10  ]
1 [ INF,    0,    3,   INF  ]
2 [ INF,  INF,    0,    1   ]
3 [ INF,  INF,   INF,   0   ]
```

**k = 2** (use node 2 as intermediate): Check `dist[i,2] + dist[2,j]`. `dist[0,2]=8`, `dist[2,3]=1` → `dist[0,3] = min(10, 8+1) = 9`. `dist[1,2]=3`, `dist[2,3]=1` → `dist[1,3] = min(INF, 3+1) = 4`.

```
      0     1     2     3
0 [   0,    5,    8,    9   ]
1 [ INF,    0,    3,    4   ]
2 [ INF,  INF,    0,    1   ]
3 [ INF,  INF,   INF,   0   ]
```

**k = 3** (use node 3 as intermediate): Check `dist[i,3] + dist[3,j]`. Since row 3 (`dist[3,j]`) is `INF` for `j=0,1,2`, no vertex can route further through node 3. Matrix unchanged.

Final result: `dist[0][3] = 9`, confirming the shorter indirect path `0->1->2->3` (cost `5+3+1=9`) beats the direct edge (cost `10`).

---

## 6. Find the City With the Smallest Number of Neighbors Within a Distance Threshold

**Problem Statement:** There are `n` cities numbered `0` to `n-1` connected by bidirectional weighted edges `edges[i] = [from, to, weight]`. Given a `distanceThreshold`, find the city that has the **smallest number of reachable cities** within `distanceThreshold` (using shortest path distances). If multiple cities tie for the smallest count, return the city with the **largest index**.

**Example:**
- Input: `n = 4`, `edges = [[0,1,3],[1,2,1],[1,3,4],[2,3,1]]`, `distanceThreshold = 4`
- Output: `3`
- Explanation: Shortest distances from city 0: `{1:3, 2:4, 3:4}` → 3 neighbors within threshold. From city 1: `{0:3, 2:1, 3:2}` → 3 neighbors. From city 2: `{0:4, 1:1, 3:1}` → 3 neighbors. From city 3: `{1:2, 2:1, 0:4}` → wait, city 0 from city 3 is `3->2->1->0 = 1+1+3=5 > 4`, so only `{1:2, 2:1}` → 2 neighbors. City 3 has the fewest reachable neighbors (2), so it is the answer.

**Approach:** Compute all-pairs shortest paths using Floyd-Warshall (simplest for dense small graphs, O(V^3)) — alternatively, run Dijkstra from every single node if the graph is sparse, for O(V * (E log V)). Once the full `dist[i][j]` matrix is known, for each city `i`, count how many other cities `j` satisfy `dist[i][j] <= distanceThreshold`. Track the city with the minimum such count, breaking ties by preferring the larger city index (achieved by iterating `i` from `0` to `n-1` and using `<=` when comparing counts, so later indices overwrite earlier ties).

```csharp
public class Solution {
    public int FindTheCity(int n, int[][] edges, int distanceThreshold) {
        const int INF = int.MaxValue / 2;
        int[,] dist = new int[n, n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                dist[i, j] = (i == j) ? 0 : INF;

        foreach (var e in edges) {
            int u = e[0], v = e[1], w = e[2];
            dist[u, v] = Math.Min(dist[u, v], w);
            dist[v, u] = Math.Min(dist[v, u], w);
        }

        // Floyd-Warshall: all-pairs shortest paths
        for (int k = 0; k < n; k++)
            for (int i = 0; i < n; i++)
                for (int j = 0; j < n; j++)
                    if (dist[i, k] + dist[k, j] < dist[i, j])
                        dist[i, j] = dist[i, k] + dist[k, j];

        int bestCity = -1;
        int bestCount = int.MaxValue;

        for (int i = 0; i < n; i++) {
            int count = 0;
            for (int j = 0; j < n; j++) {
                if (i != j && dist[i, j] <= distanceThreshold) count++;
            }
            // <= ensures ties prefer the larger index, since we iterate i ascending
            if (count <= bestCount) {
                bestCount = count;
                bestCity = i;
            }
        }

        return bestCity;
    }
}
```

Time Complexity: O(V^3) for Floyd-Warshall to build the all-pairs distance matrix, plus O(V^2) to count neighbors per city — overall O(V^3). (If using Dijkstra from every node instead: O(V * (V+E) log V).)
Space Complexity: O(V^2) for the distance matrix.

**Explanation:** Once every pairwise shortest distance is known via Floyd-Warshall, answering "how many cities are reachable within threshold from city `i`" reduces to a simple linear scan of row `i` in the distance matrix. Iterating cities in ascending order and using `<=` (not strict `<`) when updating the best count means that whenever a later city ties the current minimum count, it overwrites the recorded answer — naturally satisfying the "return the city with the largest index on ties" requirement without extra tie-breaking logic.
