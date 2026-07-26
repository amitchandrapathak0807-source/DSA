# Graphs — Basics and Traversal

## Concept: Graph Representation

A graph is a collection of nodes (vertices) connected by edges. Before solving any graph problem, you need a way to store the connections between nodes in memory. The two standard representations are the **adjacency list** and the **adjacency matrix**.

### Adjacency List

An adjacency list stores, for every vertex, only the list of vertices it is directly connected to. In C# this is typically modeled as:

```csharp
List<List<int>> adj = new List<List<int>>();       // adj[u] contains all neighbors of u
// or
Dictionary<int, List<int>> adj = new Dictionary<int, List<int>>(); // useful when node ids are sparse/non-contiguous
```

For an undirected edge `(u, v)` you push `v` into `adj[u]` and `u` into `adj[v]`. For a directed edge `u -> v` you only push `v` into `adj[u]`.

- **Space:** O(V + E) — you only store edges that actually exist.
- **Check if edge (u, v) exists:** O(degree(u)) — you must scan u's neighbor list.
- **Iterate all neighbors of u:** O(degree(u)) — very fast, exactly what BFS/DFS need.

### Adjacency Matrix

An adjacency matrix is a `V x V` 2D array (`int[,] adj = new int[V, V]` or `bool[,]`) where `adj[u][v] = 1` (or `true`, or a weight) if an edge exists between `u` and `v`, else `0`.

- **Space:** O(V²) — regardless of how many edges actually exist.
- **Check if edge (u, v) exists:** O(1) — direct array lookup.
- **Iterate all neighbors of u:** O(V) — must scan the entire row even if u has only one neighbor.

### When to Use Each

| Situation | Preferred Representation |
|---|---|
| Sparse graph (E << V²), typical of most interview problems, trees, road networks | Adjacency List |
| Traversal-heavy algorithms (BFS/DFS, shortest path via BFS, cycle detection) | Adjacency List — visiting neighbors is the dominant operation |
| Dense graph (E close to V²) | Adjacency Matrix — space overhead is acceptable and O(1) edge lookup pays off |
| Frequent "does edge (u, v) exist?" queries | Adjacency Matrix |
| Very large V (e.g., V = 10^5 or more) | Adjacency List — an adjacency matrix would need up to 10^10 cells, which is infeasible |
| Node ids are not small contiguous integers (strings, sparse ids) | `Dictionary<int, List<int>>` (or a generic key) instead of array-indexed lists |

**Rule of thumb:** default to the adjacency list. It is the standard choice for almost all competitive programming and interview graph problems because most real-world and test-case graphs are sparse, and BFS/DFS only ever need to enumerate a node's direct neighbors — which the adjacency list does in time proportional to the degree, not to V.

---

## 1. Breadth-First Search (BFS) Traversal of a Graph

**Problem Statement:** Given an undirected (or directed) graph with `V` vertices represented as an adjacency list, and a starting source vertex, print/return the BFS traversal — visiting all vertices level by level, starting from the source, using a queue.

**Example:**
- Input: `V = 5`, edges = `[(0,1), (0,2), (1,3), (1,4)]`, source = `0`
  - Adjacency list: `adj[0] = [1,2]`, `adj[1] = [0,3,4]`, `adj[2] = [0]`, `adj[3] = [1]`, `adj[4] = [1]`
- Output: `[0, 1, 2, 3, 4]`
- Explanation: Start at 0, visit its neighbors 1 and 2, then move to 1's unvisited neighbors 3 and 4. This produces a level-order traversal from the source.

**Approach:** Use a `Queue<int>` and a `bool[] visited` array. Mark the source visited and enqueue it. While the queue is non-empty, dequeue a node, add it to the result, and enqueue all of its unvisited neighbors (marking each visited **at the time it is enqueued**, not when dequeued — this avoids enqueuing the same node multiple times).

```csharp
public class GraphBFS
{
    public List<int> BFSTraversal(int V, List<List<int>> adj, int source)
    {
        bool[] visited = new bool[V];
        List<int> result = new List<int>();
        Queue<int> queue = new Queue<int>();

        visited[source] = true;
        queue.Enqueue(source);

        while (queue.Count > 0)
        {
            int node = queue.Dequeue();
            result.Add(node);

            foreach (int neighbor in adj[node])
            {
                if (!visited[neighbor])
                {
                    visited[neighbor] = true;
                    queue.Enqueue(neighbor);
                }
            }
        }

        return result;
    }

    // Handles disconnected graphs by starting BFS from every unvisited node
    public List<int> BFSAllComponents(int V, List<List<int>> adj)
    {
        bool[] visited = new bool[V];
        List<int> result = new List<int>();

        for (int i = 0; i < V; i++)
        {
            if (!visited[i])
            {
                Queue<int> queue = new Queue<int>();
                visited[i] = true;
                queue.Enqueue(i);

                while (queue.Count > 0)
                {
                    int node = queue.Dequeue();
                    result.Add(node);

                    foreach (int neighbor in adj[node])
                    {
                        if (!visited[neighbor])
                        {
                            visited[neighbor] = true;
                            queue.Enqueue(neighbor);
                        }
                    }
                }
            }
        }

        return result;
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation:** Starting at `0`, mark it visited and enqueue: `queue = [0]`. Dequeue `0`, add to result (`result = [0]`), look at `adj[0] = [1,2]` — both unvisited, mark and enqueue both: `queue = [1,2]`. Dequeue `1`, `result = [0,1]`, `adj[1] = [0,3,4]` — `0` already visited (skip), `3` and `4` unvisited so mark and enqueue: `queue = [2,3,4]`. Dequeue `2`, `result = [0,1,2]`, `adj[2] = [0]` already visited, nothing added. Dequeue `3`, `result = [0,1,2,3]`, `adj[3]=[1]` visited, nothing added. Dequeue `4`, `result = [0,1,2,3,4]`, done. Final BFS order: `[0,1,2,3,4]`.

---

## 2. Depth-First Search (DFS) Traversal of a Graph

**Problem Statement:** Given an undirected (or directed) graph with `V` vertices as an adjacency list and a starting source vertex, print/return the DFS traversal — going as deep as possible along each branch before backtracking.

**Example:**
- Input: `V = 5`, edges = `[(0,1), (0,2), (1,3), (1,4)]`, source = `0`
  - Adjacency list: `adj[0] = [1,2]`, `adj[1] = [0,3,4]`, `adj[2] = [0]`, `adj[3] = [1]`, `adj[4] = [1]`
- Output: `[0, 1, 3, 4, 2]`
- Explanation: From 0 we go deep into 1 first, then deep into 1's first unvisited neighbor 3, backtrack to 1, go to 4, backtrack all the way to 0, then finally visit 2.

**Approach:** Use recursion (the call stack implicitly acts as the "stack") with a `bool[] visited` array — visit the current node, mark it visited, then recursively visit each unvisited neighbor. Alternatively use an explicit `Stack<int>` for an iterative version.

```csharp
public class GraphDFS
{
    public List<int> DFSTraversal(int V, List<List<int>> adj, int source)
    {
        bool[] visited = new bool[V];
        List<int> result = new List<int>();
        DFSUtil(source, adj, visited, result);
        return result;
    }

    private void DFSUtil(int node, List<List<int>> adj, bool[] visited, List<int> result)
    {
        visited[node] = true;
        result.Add(node);

        foreach (int neighbor in adj[node])
        {
            if (!visited[neighbor])
            {
                DFSUtil(neighbor, adj, visited, result);
            }
        }
    }

    // Iterative DFS using an explicit stack
    public List<int> DFSIterative(int V, List<List<int>> adj, int source)
    {
        bool[] visited = new bool[V];
        List<int> result = new List<int>();
        Stack<int> stack = new Stack<int>();

        stack.Push(source);

        while (stack.Count > 0)
        {
            int node = stack.Pop();

            if (visited[node]) continue;

            visited[node] = true;
            result.Add(node);

            // push in reverse so the first neighbor is processed first
            for (int i = adj[node].Count - 1; i >= 0; i--)
            {
                int neighbor = adj[node][i];
                if (!visited[neighbor])
                {
                    stack.Push(neighbor);
                }
            }
        }

        return result;
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation:** Start at `0`, mark visited, `result = [0]`. Its first neighbor is `1` (unvisited) → recurse into `1`, mark visited, `result = [0,1]`. `1`'s first neighbor is `0` (visited, skip), next is `3` (unvisited) → recurse into `3`, mark visited, `result = [0,1,3]`. `3`'s only neighbor is `1` (visited) → backtrack to `1`. `1`'s next neighbor is `4` (unvisited) → recurse into `4`, mark visited, `result = [0,1,3,4]`. `4`'s only neighbor is `1` (visited) → backtrack to `1`, no more neighbors → backtrack to `0`. `0`'s next neighbor is `2` (unvisited) → recurse into `2`, mark visited, `result = [0,1,3,4,2]`. `2`'s only neighbor is `0` (visited) → backtrack, done. Final DFS order: `[0,1,3,4,2]`.

---

## 3. Number of Provinces (Count Connected Components in an Undirected Graph)

**Problem Statement:** There are `n` cities. Some are directly connected by roads, given as an `n x n` adjacency matrix `isConnected` where `isConnected[i][j] = 1` means city `i` and city `j` are directly connected. A **province** is a group of directly or indirectly connected cities with no other cities outside the group. Return the total number of provinces.

**Example:**
- Input: `isConnected = [[1,1,0],[1,1,0],[0,0,1]]`
- Output: `2`
- Explanation: City 0 and City 1 are connected (province 1). City 2 is isolated (province 2). Total = 2 provinces.

**Approach:** This is equivalent to counting connected components. Iterate over every city; if it hasn't been visited yet, it belongs to a new province — increment the count and run a BFS (or DFS) from it to mark every city reachable from it (directly or transitively) as visited, using the matrix row as the adjacency information.

```csharp
public class NumberOfProvinces
{
    public int FindCircleNum(int[][] isConnected)
    {
        int n = isConnected.Length;
        bool[] visited = new bool[n];
        int provinces = 0;

        for (int i = 0; i < n; i++)
        {
            if (!visited[i])
            {
                provinces++;
                BFS(i, isConnected, visited, n);
            }
        }

        return provinces;
    }

    private void BFS(int start, int[][] isConnected, bool[] visited, int n)
    {
        Queue<int> queue = new Queue<int>();
        visited[start] = true;
        queue.Enqueue(start);

        while (queue.Count > 0)
        {
            int node = queue.Dequeue();

            for (int neighbor = 0; neighbor < n; neighbor++)
            {
                if (isConnected[node][neighbor] == 1 && !visited[neighbor])
                {
                    visited[neighbor] = true;
                    queue.Enqueue(neighbor);
                }
            }
        }
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation:** `n = 3`. `i=0` is unvisited → `provinces = 1`, BFS from 0: enqueue 0, mark visited. Dequeue 0, row `[1,1,0]` → neighbor 1 is connected and unvisited, mark and enqueue: `queue=[1]`. Dequeue 1, row `[1,1,0]` → neighbor 0 already visited, nothing new. Queue empty, BFS ends — cities `{0,1}` visited. `i=1` already visited, skip. `i=2` unvisited → `provinces = 2`, BFS from 2 only marks `{2}`. Final answer: `2` provinces.

---

## 4. Number of Islands (in a 2D grid)

**Problem Statement:** Given an `m x n` binary grid where `1` represents land and `0` represents water, count the number of islands. An island is formed by connecting adjacent lands horizontally, vertically, **or diagonally** (8-directional, as commonly asked in this variant), surrounded by water.

**Example:**
- Input:
```
grid = [
  [1,1,0,0],
  [1,0,0,0],
  [0,0,1,1],
  [0,0,0,0]
]
```
- Output: `2`
- Explanation: The top-left `1`s at (0,0),(0,1),(1,0) form one island (connected). The `1`s at (2,2),(2,3) form a second island. Total = 2 islands.

**Approach:** Scan every cell; whenever an unvisited land cell (`1`) is found, it marks the start of a new island — increment the count and run BFS/DFS from it to visit and mark all connected land cells (checking all 8 neighboring directions) as visited so they aren't recounted.

```csharp
public class NumberOfIslands
{
    private static readonly int[] dRow = { -1, -1, -1, 0, 0, 1, 1, 1 };
    private static readonly int[] dCol = { -1, 0, 1, -1, 1, -1, 0, 1 };

    public int NumIslands(int[][] grid)
    {
        int m = grid.Length;
        int n = grid[0].Length;
        bool[,] visited = new bool[m, n];
        int islands = 0;

        for (int r = 0; r < m; r++)
        {
            for (int c = 0; c < n; c++)
            {
                if (grid[r][c] == 1 && !visited[r, c])
                {
                    islands++;
                    BFS(r, c, grid, visited, m, n);
                }
            }
        }

        return islands;
    }

    private void BFS(int startRow, int startCol, int[][] grid, bool[,] visited, int m, int n)
    {
        Queue<(int row, int col)> queue = new Queue<(int, int)>();
        visited[startRow, startCol] = true;
        queue.Enqueue((startRow, startCol));

        while (queue.Count > 0)
        {
            var (row, col) = queue.Dequeue();

            for (int dir = 0; dir < 8; dir++)
            {
                int newRow = row + dRow[dir];
                int newCol = col + dCol[dir];

                if (newRow >= 0 && newRow < m && newCol >= 0 && newCol < n &&
                    grid[newRow][newCol] == 1 && !visited[newRow, newCol])
                {
                    visited[newRow, newCol] = true;
                    queue.Enqueue((newRow, newCol));
                }
            }
        }
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation:** Scanning row by row: `(0,0)=1`, unvisited → `islands=1`, BFS marks `(0,0),(0,1),(1,0)` visited (all connected via horizontal/vertical/diagonal adjacency). Continue scan: `(0,2)=0`, `(0,3)=0`, `(1,1)=0`, `(1,2)=0`, `(1,3)=0` skip. `(2,0)=0`, `(2,1)=0` skip. `(2,2)=1`, unvisited → `islands=2`, BFS marks `(2,2),(2,3)` visited. Rest of grid is `0`s or already visited. Final count: `2` islands.

---

## 5. Flood Fill Algorithm

**Problem Statement:** Given an `m x n` image (2D grid of integers), a starting pixel `(sr, sc)`, and a `newColor`, perform a flood fill: replace the color of the starting pixel and all pixels connected to it (4-directionally: up/down/left/right) that share the **starting pixel's original color** with `newColor`.

**Example:**
- Input:
```
image = [
  [1,1,1],
  [1,1,0],
  [1,0,1]
]
sr = 1, sc = 1, newColor = 2
```
- Output:
```
[
  [2,2,2],
  [2,2,0],
  [2,0,1]
]
```
- Explanation: Starting color at (1,1) is `1`. All 4-directionally connected pixels with color `1` reachable from (1,1) get changed to `2`. The `0` pixels and the isolated `1` at (2,2) are untouched.

**Approach:** Record the starting pixel's original color. Run BFS (or DFS) from `(sr, sc)`; for every visited pixel whose color equals the original color, change it to `newColor` and enqueue its unvisited same-colored 4-directional neighbors. Guard against the edge case where `newColor == originalColor` (would cause infinite requeueing / no-op needed) by checking before processing, or simply relying on the visited check.

```csharp
public class FloodFill
{
    private static readonly int[] dRow = { -1, 1, 0, 0 };
    private static readonly int[] dCol = { 0, 0, -1, 1 };

    public int[][] FloodFillGrid(int[][] image, int sr, int sc, int newColor)
    {
        int originalColor = image[sr][sc];
        if (originalColor == newColor) return image; // nothing to do, avoids re-processing

        int m = image.Length;
        int n = image[0].Length;
        bool[,] visited = new bool[m, n];

        Queue<(int row, int col)> queue = new Queue<(int, int)>();
        visited[sr, sc] = true;
        queue.Enqueue((sr, sc));
        image[sr][sc] = newColor;

        while (queue.Count > 0)
        {
            var (row, col) = queue.Dequeue();

            for (int dir = 0; dir < 4; dir++)
            {
                int newRow = row + dRow[dir];
                int newCol = col + dCol[dir];

                if (newRow >= 0 && newRow < m && newCol >= 0 && newCol < n &&
                    !visited[newRow, newCol] && image[newRow][newCol] == originalColor)
                {
                    visited[newRow, newCol] = true;
                    image[newRow][newCol] = newColor;
                    queue.Enqueue((newRow, newCol));
                }
            }
        }

        return image;
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation:** `originalColor = image[1][1] = 1`, `newColor = 2`, they differ so proceed. Mark `(1,1)` visited, color it `2`, enqueue: `queue=[(1,1)]`. Dequeue `(1,1)`. Check neighbors: up `(0,1)=1` matches original, color it `2`, mark visited, enqueue. down `(2,1)=0` doesn't match, skip. left `(1,0)=1` matches, color `2`, mark visited, enqueue. right `(1,2)=0` doesn't match, skip. `queue=[(0,1),(1,0)]`. Dequeue `(0,1)`: neighbors `(−1,1)` out of bounds, `(1,1)` visited, `(0,0)=1` matches → color `2`, enqueue, `(0,2)=1` matches → color `2`, enqueue. `queue=[(1,0),(0,0),(0,2)]`. Continue similarly — `(1,0)` neighbor `(2,0)=1` matches → color `2`, enqueue. `(0,0)` and `(0,2)` have no new unvisited matching neighbors. `(2,0)` neighbor `(2,1)=0` skip. Queue empties. Final image has all originally-connected `1`s turned to `2`, while `(2,2)=1` (not reachable, disconnected from the region) stays `1`.

---

## 6. Rotten Oranges (Minimum Time for All Oranges to Rot — Multi-source BFS)

**Problem Statement:** Given an `m x n` grid where each cell is `0` (empty), `1` (fresh orange), or `2` (rotten orange), every minute any fresh orange 4-directionally adjacent to a rotten orange becomes rotten. Return the minimum number of minutes until no cell has a fresh orange, or `-1` if it is impossible for all oranges to rot.

**Example:**
- Input:
```
grid = [
  [2,1,1],
  [1,1,0],
  [0,1,1]
]
```
- Output: `4`
- Explanation: All rotten oranges start rotting their neighbors simultaneously. It takes 4 minutes for the rot to spread to every fresh orange in the grid.

**Approach (Multi-source BFS):** Instead of starting BFS from a single node, push **all initially rotten oranges into the queue at once** (multiple sources) along with a time/level marker, and also count total fresh oranges. Process level by level: for each orange popped, rot its fresh 4-directional neighbors, decrement the fresh count, and enqueue them for the next level. Track elapsed minutes by processing the queue in levels (snapshot the queue size at the start of each level, or store `(row, col, time)` tuples). If fresh count reaches 0, return the elapsed time; if the queue empties while fresh oranges remain, return `-1`.

```csharp
public class RottenOranges
{
    private static readonly int[] dRow = { -1, 1, 0, 0 };
    private static readonly int[] dCol = { 0, 0, -1, 1 };

    public int OrangesRotting(int[][] grid)
    {
        int m = grid.Length;
        int n = grid[0].Length;
        Queue<(int row, int col, int time)> queue = new Queue<(int, int, int)>();
        int freshCount = 0;

        // multi-source seed: enqueue every initially rotten orange
        for (int r = 0; r < m; r++)
        {
            for (int c = 0; c < n; c++)
            {
                if (grid[r][c] == 2)
                {
                    queue.Enqueue((r, c, 0));
                }
                else if (grid[r][c] == 1)
                {
                    freshCount++;
                }
            }
        }

        int minutesElapsed = 0;

        while (queue.Count > 0)
        {
            var (row, col, time) = queue.Dequeue();
            minutesElapsed = Math.Max(minutesElapsed, time);

            for (int dir = 0; dir < 4; dir++)
            {
                int newRow = row + dRow[dir];
                int newCol = col + dCol[dir];

                if (newRow >= 0 && newRow < m && newCol >= 0 && newCol < n &&
                    grid[newRow][newCol] == 1)
                {
                    grid[newRow][newCol] = 2;
                    freshCount--;
                    queue.Enqueue((newRow, newCol, time + 1));
                }
            }
        }

        return freshCount == 0 ? minutesElapsed : -1;
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation (multi-source BFS dry run):**

Grid:
```
2 1 1
1 1 0
0 1 1
```
Initial seeding: the only rotten orange is `(0,0)`. `freshCount = 6` (all the `1`s). Queue after seeding: `[(0,0,0)]`.

- **Minute 0:** Dequeue `(0,0,0)`. Neighbors: right `(0,1)=1` → rot it, `freshCount=5`, enqueue `(0,1,1)`. down `(1,0)=1` → rot it, `freshCount=4`, enqueue `(1,0,1)`. Queue: `[(0,1,1),(1,0,1)]`.
- **Minute 1:** Dequeue `(0,1,1)`. Neighbors: right `(0,2)=1` → rot, `freshCount=3`, enqueue `(0,2,2)`. down `(1,1)=1` → rot, `freshCount=2`, enqueue `(1,1,2)`. left `(0,0)=2` already rotten, skip. Dequeue `(1,0,1)`. Neighbors: right `(1,1)` just became `2`, skip. down `(2,0)=0` empty, skip. up `(0,0)=2` skip. Queue: `[(0,2,2),(1,1,2)]`.
- **Minute 2:** Dequeue `(0,2,2)`. Neighbors: down `(1,2)=0` empty, skip. No fresh neighbors. Dequeue `(1,1,2)`. Neighbors: right `(1,2)=0` skip. down `(2,1)=1` → rot, `freshCount=1`, enqueue `(2,1,3)`. left `(1,0)=2` skip. up `(0,1)=2` skip. Queue: `[(2,1,3)]`.
- **Minute 3:** Dequeue `(2,1,3)`. Neighbors: right `(2,2)=1` → rot, `freshCount=0`, enqueue `(2,2,4)`. left `(2,0)=0` skip. up `(1,1)=2` skip. Queue: `[(2,2,4)]`.
- **Minute 4:** Dequeue `(2,2,4)`. No fresh neighbors left. `minutesElapsed = max(...,4) = 4`. Queue empty.

`freshCount == 0`, so return `minutesElapsed = 4`, matching the expected output. If any fresh orange had remained unreachable, `freshCount` would stay `> 0` after the queue empties and the function would return `-1`.

---

## 7. Detect Cycle in an Undirected Graph (using both BFS and DFS)

**Problem Statement:** Given an undirected graph with `V` vertices represented as an adjacency list, determine whether the graph contains a cycle (a path that starts and ends at the same vertex without reusing an edge, involving at least 3 distinct vertices, or more precisely: any back edge to an already-visited vertex that is not the immediate parent).

**Example:**
- Input: `V = 4`, edges = `[(0,1), (1,2), (2,3), (3,0)]`
  - Adjacency list: `adj[0]=[1,3]`, `adj[1]=[0,2]`, `adj[2]=[1,3]`, `adj[3]=[2,0]`
- Output: `true`
- Explanation: `0 -> 1 -> 2 -> 3 -> 0` forms a cycle back to the starting vertex.

**Approach:** The key idea for undirected cycle detection is **parent-tracking**: since every undirected edge `(u,v)` is stored twice (once as `v` in `adj[u]` and once as `u` in `adj[v]`), when we traverse from `u` to `v`, `v`'s neighbor list will always contain `u` back — that is expected and is NOT a cycle, because it's just the edge we came from. A genuine cycle exists only when we reach a vertex that is already visited **through a different path**, i.e., a visited neighbor that is not the immediate parent we just came from.

- **BFS version:** enqueue `(node, parent)` pairs. For each neighbor of the dequeued node: if unvisited, mark visited and enqueue `(neighbor, node)`. If visited AND neighbor != parent, a cycle is found.
- **DFS version:** recurse with `(node, parent)`. For each neighbor: if unvisited, recurse with `(neighbor, node)` — if that recursive call finds a cycle, propagate `true`. If neighbor is visited AND neighbor != parent, a cycle is found.

```csharp
public class DetectCycleUndirected
{
    // ---------- BFS-based cycle detection ----------
    public bool IsCyclicBFS(int V, List<List<int>> adj)
    {
        bool[] visited = new bool[V];

        for (int i = 0; i < V; i++)
        {
            if (!visited[i])
            {
                if (BFSCheckCycle(i, adj, visited))
                    return true;
            }
        }
        return false;
    }

    private bool BFSCheckCycle(int start, List<List<int>> adj, bool[] visited)
    {
        Queue<(int node, int parent)> queue = new Queue<(int, int)>();
        visited[start] = true;
        queue.Enqueue((start, -1));

        while (queue.Count > 0)
        {
            var (node, parent) = queue.Dequeue();

            foreach (int neighbor in adj[node])
            {
                if (!visited[neighbor])
                {
                    visited[neighbor] = true;
                    queue.Enqueue((neighbor, node));
                }
                else if (neighbor != parent)
                {
                    // visited neighbor that is NOT where we came from => cycle
                    return true;
                }
            }
        }
        return false;
    }

    // ---------- DFS-based cycle detection ----------
    public bool IsCyclicDFS(int V, List<List<int>> adj)
    {
        bool[] visited = new bool[V];

        for (int i = 0; i < V; i++)
        {
            if (!visited[i])
            {
                if (DFSCheckCycle(i, -1, adj, visited))
                    return true;
            }
        }
        return false;
    }

    private bool DFSCheckCycle(int node, int parent, List<List<int>> adj, bool[] visited)
    {
        visited[node] = true;

        foreach (int neighbor in adj[node])
        {
            if (!visited[neighbor])
            {
                if (DFSCheckCycle(neighbor, node, adj, visited))
                    return true;
            }
            else if (neighbor != parent)
            {
                // back-edge to a non-parent visited vertex => cycle
                return true;
            }
        }
        return false;
    }
}
```

Time Complexity: O(V+E) for graph traversal, O(n*m) for grid problems. Space Complexity: O(V) or O(n*m) for visited tracking + recursion/queue.

**Explanation (BFS parent-tracking dry run):**

Graph: `adj[0]=[1,3]`, `adj[1]=[0,2]`, `adj[2]=[1,3]`, `adj[3]=[2,0]` (cycle `0-1-2-3-0`).

Start BFS from `0`: mark visited, enqueue `(0, -1)`. Queue: `[(0,-1)]`.

- Dequeue `(0, -1)`. Neighbors of `0`: `1` — unvisited, mark visited, enqueue `(1, 0)`. `3` — unvisited, mark visited, enqueue `(3, 0)`. Queue: `[(1,0),(3,0)]`.
- Dequeue `(1, 0)`. Neighbors of `1`: `0` — visited, and `0 == parent (0)` → this is just the edge back to where we came from, NOT a cycle, skip. `2` — unvisited, mark visited, enqueue `(2, 1)`. Queue: `[(3,0),(2,1)]`.
- Dequeue `(3, 0)`. Neighbors of `3`: `2` — unvisited, mark visited, enqueue `(2... )` — wait, `2` was already marked visited in the previous step, so actually `2` is now visited → check `2 != parent(0)` → **true, cycle detected!** Return `true` immediately.

So the moment vertex `3` looks at neighbor `2` and finds it already visited (visited earlier via the `1 -> 2` branch) while `3`'s parent is `0` (not `2`), the algorithm correctly identifies that `2` was reached through a different path than the immediate parent edge — confirming the cycle `0 -> 1 -> 2 -> 3 -> 0`. Contrast this with vertex `1` seeing neighbor `0`: `0` is visited, but `0` IS `1`'s parent, so that visited-check is correctly ignored as merely the reverse direction of the edge just traversed, not a cycle.
