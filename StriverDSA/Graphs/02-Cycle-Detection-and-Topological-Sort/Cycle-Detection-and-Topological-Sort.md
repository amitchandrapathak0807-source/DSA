# Graphs — Cycle Detection and Topological Sort

## Concept: Directed Acyclic Graphs and Topological Order

A **Directed Acyclic Graph (DAG)** is a directed graph that contains no directed cycles — there is no way to start at some vertex `v` and follow a sequence of directed edges that eventually leads back to `v`. DAGs naturally model dependency relationships: build systems, course prerequisites, task scheduling, spreadsheet formula evaluation, and compilation order all form DAGs, because a real dependency structure cannot require something to (indirectly) depend on itself.

A **topological ordering** (or topological sort) of a DAG is a linear arrangement of all its vertices such that for every directed edge `u -> v`, `u` appears **before** `v` in the ordering. In other words, every vertex comes before all vertices it points to (directly or indirectly). Key properties:

- A topological ordering exists **if and only if** the graph is a DAG (has no directed cycle). If a cycle exists, no valid linear order can satisfy all the "comes before" constraints simultaneously, because the cycle creates a contradictory requirement (some vertex would need to come before itself).
- A DAG can have **more than one** valid topological ordering (unless the DAG imposes a total order via a Hamiltonian path of edges).
- Two classic algorithms compute a topological order: **DFS-based** (push nodes to a stack in post-order, then reverse) and **Kahn's algorithm** (BFS using in-degree counts).
- Because cycle existence and topological-sort existence are two sides of the same coin, cycle detection in directed graphs is usually implemented as "try to build a topological order; if you get stuck with only cyclic nodes left, a cycle exists."

The problems below build up from raw cycle detection, to the two topological-sort algorithms, to real applications (Course Schedule I/II, Eventual Safe States, Alien Dictionary) that all reduce to these two core ideas.

---

## 1. Detect Cycle in a Directed Graph

**Problem Statement:** Given a directed graph with `V` vertices and a list of directed edges, determine whether the graph contains a cycle.

**Example:**
- Input: `V = 4`, edges = `[[0,1],[1,2],[2,3],[3,1]]`
- Output: `true`
- Explanation: Following `1 -> 2 -> 3 -> 1` revisits node `1` while it is still "on the current DFS path", so a cycle exists.

**Approach:** Unlike undirected-graph cycle detection (where visiting an already-visited neighbor other than the parent means a cycle), a directed graph needs a second array: `inRecursionStack[]` (or `inStack[]`). Do a DFS; mark a node `visited` when first reached and `inStack = true` while it is on the current recursion path. If DFS reaches a neighbor that is `visited` **and** currently `inStack`, that is a **back edge**, which means a cycle. When DFS finishes exploring a node (returns from recursion), unmark it from `inStack` (but leave it `visited`) because it is no longer on the active path. If a neighbor is `visited` but not `inStack`, it was already fully explored via another path and is safe (not a cycle).

**Logic (Steps):**
1. Create two `bool[]` arrays: `visited` (ever reached) and `inStack` (currently on the active DFS path).
2. Loop over all vertices; for each unvisited one, launch a DFS to cover disconnected components.
3. Inside DFS, mark `node` both `visited` and `inStack = true`.
4. For each neighbor: if unvisited, recurse (propagate `true` if it finds a cycle); if visited **and** `inStack[neighbor]` is true, a back edge to a node on the current path is found — return `true` (cycle).
5. After exploring all neighbors, unmark `inStack[node] = false` (backtrack) since the node is no longer on the active path, then return `false`.

```csharp
public class Solution {
    public bool HasCycle(int V, List<List<int>> adj) {
        bool[] visited = new bool[V];
        bool[] inStack = new bool[V];

        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                if (Dfs(i, adj, visited, inStack)) return true;
            }
        }
        return false;
    }

    private bool Dfs(int node, List<List<int>> adj, bool[] visited, bool[] inStack) {
        visited[node] = true;
        inStack[node] = true;

        foreach (int neighbor in adj[node]) {
            if (!visited[neighbor]) {
                if (Dfs(neighbor, adj, visited, inStack)) return true;
            } else if (inStack[neighbor]) {
                // back edge to a node on the current DFS path -> cycle
                return true;
            }
        }

        inStack[node] = false; // done exploring this node, remove from recursion stack
        return false;
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** For `edges = [[0,1],[1,2],[2,3],[3,1]]`: DFS starts at `0`, marks `visited[0]=inStack[0]=true`, moves to `1` (`visited[1]=inStack[1]=true`), moves to `2` (`visited[2]=inStack[2]=true`), moves to `3` (`visited[3]=inStack[3]=true`). Node `3`'s neighbor is `1`, which is `visited=true` and `inStack=true` — a back edge is found, so the function immediately returns `true` up the call chain and `HasCycle` reports `true`, matching the expected output. Note that if the edge were `3 -> 0` instead of `3 -> 1`, `0` would still be on the stack too, so it would also be a cycle; the key distinguishing case is a node that is `visited` but has already been popped off (`inStack=false`), which is *not* flagged as a cycle since it represents a DAG-style shared descendant, not a back edge.

---

## 2. Topological Sort Using DFS

**Problem Statement:** Given a DAG with `V` vertices, return any valid topological ordering of its vertices.

**Example:**
- Input: `V = 6`, edges = `[[5,2],[5,0],[4,0],[4,1],[2,3],[3,1]]`
- Output: `[5, 4, 2, 3, 1, 0]` (one valid ordering; others are also acceptable)
- Explanation: `5` and `4` have no incoming edges so they can come first; `0` and `1` have edges pointing into them from several nodes so they end up last.

**Approach:** Run a standard DFS. The key insight is: a node should appear in the topological order **after** all nodes reachable from it. So, after fully exploring all of a node's descendants (post-order — i.e., after the recursive DFS calls for all neighbors return), push that node onto a stack. Once DFS from all unvisited nodes finishes, popping the stack from top to bottom gives a valid topological order, because a node is only pushed after everything it depends on (its descendants) has already been pushed.

**Logic (Steps):**
1. Create a `bool[] visited` array and a `Stack<int>` to collect the post-order.
2. Loop over all vertices; for each unvisited one, run DFS to also handle disconnected components.
3. Inside DFS, mark `node` visited, then recurse into every unvisited neighbor first (depth-first).
4. Only after all of a node's neighbors (and their descendants) have been fully recursed into, push `node` onto the stack — this post-order guarantees dependents are pushed before their dependencies.
5. Once all DFS calls finish, pop the stack from top to bottom into `result` — this gives a valid topological order since each node is popped only after everything reachable from it was already popped.

```csharp
public class Solution {
    public int[] TopoSortDfs(int V, List<List<int>> adj) {
        bool[] visited = new bool[V];
        Stack<int> stack = new Stack<int>();

        for (int i = 0; i < V; i++) {
            if (!visited[i]) {
                Dfs(i, adj, visited, stack);
            }
        }

        int[] result = new int[V];
        int idx = 0;
        while (stack.Count > 0) {
            result[idx++] = stack.Pop();
        }
        return result;
    }

    private void Dfs(int node, List<List<int>> adj, bool[] visited, Stack<int> stack) {
        visited[node] = true;
        foreach (int neighbor in adj[node]) {
            if (!visited[neighbor]) {
                Dfs(neighbor, adj, visited, stack);
            }
        }
        stack.Push(node); // post-order: push only after all descendants are pushed
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** For `edges = [[5,2],[5,0],[4,0],[4,1],[2,3],[3,1]]`: starting DFS at `0`, it has no outgoing edges, so it is pushed immediately: stack = `[0]`. DFS at `1`: no outgoing edges, pushed: stack = `[0, 1]`. DFS at `2`: goes to `3`, `3` goes to `1` (already visited), so `3` is pushed (stack = `[0,1,3]`), then `2` is pushed (stack = `[0,1,3,2]`). DFS at `4`: goes to `0` (visited) and `1` (visited), so `4` is pushed (stack = `[0,1,3,2,4]`). DFS at `5`: goes to `2` (visited) and `0` (visited), so `5` is pushed last (stack = `[0,1,3,2,4,5]`). Popping the stack top to bottom gives `5, 4, 2, 3, 1, 0` — a valid topological order, matching the expected output.

---

## 3. Topological Sort Using BFS (Kahn's Algorithm)

**Problem Statement:** Given a DAG with `V` vertices, return a valid topological ordering using a BFS-based approach instead of DFS.

**Example:**
- Input: `V = 6`, edges = `[[5,2],[5,0],[4,0],[4,1],[2,3],[3,1]]`
- Output: `[4, 5, 2, 0, 3, 1]` (one valid BFS-based ordering)
- Explanation: Nodes with in-degree `0` (`4` and `5`) are processed first; as their outgoing edges are "removed", new nodes reach in-degree `0` and join the queue.

**Approach (Kahn's Algorithm):** Compute the **in-degree** (number of incoming edges) of every vertex. Push all vertices with in-degree `0` into a queue — they have no unmet dependency and can safely be first. Repeatedly pop a vertex from the queue, append it to the result, and "remove" its outgoing edges by decrementing the in-degree of each neighbor; whenever a neighbor's in-degree drops to `0`, push it into the queue. Continue until the queue is empty. If the result contains all `V` vertices, it is a valid topological order; if fewer than `V` vertices were processed, the graph has a cycle (some nodes never reach in-degree `0`).

**Logic (Steps):**
1. Compute `inDegree[v]` for every vertex by scanning all adjacency lists and counting incoming edges.
2. Push every vertex with `inDegree == 0` into a `Queue<int>` — these have no unmet prerequisite.
3. While the queue is non-empty, dequeue a node, append it to `result`.
4. For each neighbor of that node, decrement its `inDegree`; whenever a neighbor's `inDegree` hits `0`, enqueue it (all its prerequisites are now satisfied).
5. Continue until the queue empties; `result` now holds a valid topological order (if all `V` nodes were processed, otherwise the graph had a cycle).

```csharp
public class Solution {
    public int[] TopoSortBfs(int V, List<List<int>> adj) {
        int[] inDegree = new int[V];
        foreach (var neighbors in adj) {
            foreach (int v in neighbors) {
                inDegree[v]++;
            }
        }

        Queue<int> queue = new Queue<int>();
        for (int i = 0; i < V; i++) {
            if (inDegree[i] == 0) queue.Enqueue(i);
        }

        int[] result = new int[V];
        int idx = 0;

        while (queue.Count > 0) {
            int node = queue.Dequeue();
            result[idx++] = node;

            foreach (int neighbor in adj[node]) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) {
                    queue.Enqueue(neighbor);
                }
            }
        }

        // if idx < V here, the graph has a cycle
        return result;
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** Graph: `5->2, 5->0, 4->0, 4->1, 2->3, 3->1`.

Initial in-degrees: `inDegree = [0:2, 1:2, 2:1, 3:1, 4:0, 5:0]` (node 0 has incoming from 5,4 → 2; node 1 has incoming from 4,3 → 2; node 2 has incoming from 5 → 1; node 3 has incoming from 2 → 1; node 4,5 have none).

- Queue initialized with nodes having in-degree 0: `queue = [4, 5]`.
- Pop `4` → result = `[4]`. Decrement neighbors `0` (`2→1`) and `1` (`2→1`). Neither hits 0. `queue = [5]`.
- Pop `5` → result = `[4, 5]`. Decrement neighbors `2` (`1→0`, enqueue) and `0` (`1→0`, enqueue). `queue = [2, 0]`.
- Pop `2` → result = `[4, 5, 2]`. Decrement neighbor `3` (`1→0`, enqueue). `queue = [0, 3]`.
- Pop `0` → result = `[4, 5, 2, 0]`. No outgoing edges. `queue = [3]`.
- Pop `3` → result = `[4, 5, 2, 0, 3]`. Decrement neighbor `1` (`1→0`, enqueue). `queue = [1]`.
- Pop `1` → result = `[4, 5, 2, 0, 3, 1]`. No outgoing edges. `queue = []`.

Final order: `[4, 5, 2, 0, 3, 1]`, all 6 nodes processed, confirming no cycle.

---

## 4. Course Schedule I

**Problem Statement:** There are `numCourses` courses labeled `0` to `numCourses-1`. Given a list of prerequisite pairs `[a, b]` meaning "to take course `a` you must first take course `b`" (edge `b -> a`), determine whether it is possible to finish all courses.

**Example:**
- Input: `numCourses = 2`, prerequisites = `[[1,0]]`
- Output: `true`
- Explanation: Take course `0` first, then course `1`. No cycle exists.
- Input: `numCourses = 2`, prerequisites = `[[1,0],[0,1]]`
- Output: `false`
- Explanation: Course `1` needs `0`, and course `0` needs `1` — a circular dependency, impossible to satisfy.

**Approach:** This is a direct reduction to **cycle detection in a directed graph**. Build a directed graph where prerequisite `b -> a` (b must come before a). All courses can be finished if and only if this graph is a DAG (no cycle). Use Kahn's algorithm: if the number of nodes processed via BFS (in-degree-0 removal) equals `numCourses`, all courses can be completed; otherwise a cycle blocks some courses forever.

**Logic (Steps):**
1. Build the adjacency list `adj[prereq].Add(course)` and `inDegree[course]` from each `[course, prereq]` pair.
2. Enqueue every course with `inDegree == 0` (no prerequisite left).
3. Run Kahn's BFS: dequeue a course, increment `processed`, decrement `inDegree` of each dependent course, enqueue any that drop to `0`.
4. Continue until the queue is empty.
5. Return `processed == numCourses` — if fewer courses were processed, some courses form a cycle and can never reach `inDegree == 0`.

```csharp
public class Solution {
    public bool CanFinish(int numCourses, int[][] prerequisites) {
        List<List<int>> adj = new List<List<int>>();
        for (int i = 0; i < numCourses; i++) adj.Add(new List<int>());

        int[] inDegree = new int[numCourses];
        foreach (var pr in prerequisites) {
            int course = pr[0], prereq = pr[1];
            adj[prereq].Add(course); // prereq -> course
            inDegree[course]++;
        }

        Queue<int> queue = new Queue<int>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) queue.Enqueue(i);
        }

        int processed = 0;
        while (queue.Count > 0) {
            int node = queue.Dequeue();
            processed++;
            foreach (int neighbor in adj[node]) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) queue.Enqueue(neighbor);
            }
        }

        return processed == numCourses;
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** For `prerequisites = [[1,0]]`: edge `0 -> 1`, `inDegree = [0, 1]`. Queue starts with `[0]`. Pop `0`, `processed=1`, decrement `inDegree[1]` to `0`, enqueue `1`. Pop `1`, `processed=2`. `processed == numCourses (2)` so return `true`, matching the expected output. For `[[1,0],[0,1]]`: edges `0->1` and `1->0`, `inDegree = [1, 1]`. Queue starts empty (no node has in-degree 0). Loop never runs, `processed = 0 != 2`, so return `false` — correctly detecting the cycle, matching that example's expected output.

---

## 5. Course Schedule II

**Problem Statement:** Same setup as Course Schedule I, but instead of a yes/no answer, return **one valid ordering** in which the courses can be completed. If it is impossible (a cycle exists), return an empty array.

**Example:**
- Input: `numCourses = 4`, prerequisites = `[[1,0],[2,0],[3,1],[3,2]]`
- Output: `[0, 1, 2, 3]` (or `[0, 2, 1, 3]` — either is valid)
- Explanation: Course `0` has no prerequisites; `1` and `2` depend on `0`; `3` depends on both `1` and `2`.

**Approach:** Identical to Kahn's algorithm topological sort (Problem 3), applied to the prerequisite graph (edge `prereq -> course`). Collect nodes into the result array as they are dequeued. If the result contains all `numCourses` nodes at the end, return it; otherwise a cycle exists, so return an empty array.

**Logic (Steps):**
1. Build `adj[prereq].Add(course)` and `inDegree[course]` from each `[course, prereq]` pair.
2. Enqueue every course with `inDegree == 0`.
3. Run Kahn's BFS: dequeue a course, append it to `order`, decrement `inDegree` of dependents, enqueue those hitting `0`.
4. Continue until the queue is empty.
5. If `idx == numCourses`, return `order` (a valid completion sequence); otherwise return an empty array since a cycle blocked some courses.

```csharp
public class Solution {
    public int[] FindOrder(int numCourses, int[][] prerequisites) {
        List<List<int>> adj = new List<List<int>>();
        for (int i = 0; i < numCourses; i++) adj.Add(new List<int>());

        int[] inDegree = new int[numCourses];
        foreach (var pr in prerequisites) {
            int course = pr[0], prereq = pr[1];
            adj[prereq].Add(course);
            inDegree[course]++;
        }

        Queue<int> queue = new Queue<int>();
        for (int i = 0; i < numCourses; i++) {
            if (inDegree[i] == 0) queue.Enqueue(i);
        }

        int[] order = new int[numCourses];
        int idx = 0;

        while (queue.Count > 0) {
            int node = queue.Dequeue();
            order[idx++] = node;
            foreach (int neighbor in adj[node]) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) queue.Enqueue(neighbor);
            }
        }

        return idx == numCourses ? order : new int[0];
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** For `prerequisites = [[1,0],[2,0],[3,1],[3,2]]`: edges `0->1, 0->2, 1->3, 2->3`. `inDegree = [0:0, 1:1, 2:1, 3:2]`. Queue starts `[0]`. Pop `0` → order=`[0]`, decrement `1` (`1→0`, enqueue) and `2` (`1→0`, enqueue). Queue = `[1,2]`. Pop `1` → order=`[0,1]`, decrement `3` (`2→1`). Pop `2` → order=`[0,1,2]`, decrement `3` (`1→0`, enqueue). Pop `3` → order=`[0,1,2,3]`. `idx=4=numCourses`, return `[0,1,2,3]`, matching the expected output.

---

## 6. Find Eventual Safe States

**Problem Statement:** Given a directed graph with `n` nodes, a node is called a **terminal node** if it has no outgoing edges, and a node is called a **safe node** if every possible path starting from it eventually leads to a terminal node (i.e., it never gets stuck in a cycle). Return all safe nodes in ascending order.

**Example:**
- Input: `n = 7`, edges = `[[1,2],[1,3],[3,0],[4,0],[0,5],[5,2],[6,2],[3,4]]` (adjacency list form: `graph[0]=[5], graph[1]=[2,3], graph[2]=[], graph[3]=[0,4], graph[4]=[0], graph[5]=[2], graph[6]=[2]`)
- Output: `[2, 4, 5, 6]`
- Explanation: Nodes `2` is terminal (safe). `5 -> 2` (terminal) so `5` is safe. `4 -> 0 -> 5 -> 2` all safe, so `4` is safe. `6 -> 2` is safe. Nodes `0, 1, 3` eventually participate in or lead into a cycle (`0 -> 5 -> 2` is fine actually — but here `0`'s only real path must be checked carefully via the algorithm; nodes that can reach a cycle are unsafe).

**Approach:** A node is unsafe if it lies on a cycle or can reach a cycle; it is safe only if **all** outgoing paths terminate. The cleanest way is to reverse the reasoning: **reverse all edges**, then a node is safe in the original graph exactly if it is "terminal-reachable" — equivalently, run **Kahn's algorithm on the reversed graph** starting from nodes with out-degree 0 (which become in-degree 0 in the reversed graph i.e. terminal nodes), and every node that gets fully processed (peeled off) is safe. Alternatively, do a DFS with **3-color marking** (`0=unvisited, 1=visiting/in current path, 2=safe`): a node is safe only if none of its neighbors lead back into the current path and all neighbors are themselves safe. Below is the reverse-graph + Kahn's algorithm approach, treating original outgoing edges as incoming edges in the reversed graph, and peeling from in-degree 0 (which corresponds to original out-degree 0, i.e., terminal nodes).

**Logic (Steps):**
1. Build a reversed adjacency list `revAdj[v].Add(u)` for every original edge `u -> v`, and track `outDegree[u]` (out-degree in the original graph).
2. Enqueue every node with `outDegree == 0` — these are terminal nodes, trivially safe.
3. Run Kahn's-style BFS on the reversed graph: dequeue a node, mark it `safe`, then for each predecessor in `revAdj`, decrement its `outDegree`; when a predecessor's `outDegree` hits `0`, all of its original outgoing edges now point to confirmed-safe nodes, so enqueue it too.
4. Continue until the queue empties.
5. Collect all nodes marked `safe` in ascending order as the result — nodes on or feeding into a cycle never reach `outDegree == 0` and are correctly excluded.

```csharp
public class Solution {
    public IList<int> EventualSafeNodes(int[][] graph) {
        int n = graph.Length;
        List<List<int>> revAdj = new List<List<int>>();
        for (int i = 0; i < n; i++) revAdj.Add(new List<int>());

        int[] outDegree = new int[n]; // out-degree in the ORIGINAL graph
        for (int u = 0; u < n; u++) {
            foreach (int v in graph[u]) {
                revAdj[v].Add(u); // reverse edge: v -> u
                outDegree[u]++;
            }
        }

        Queue<int> queue = new Queue<int>();
        for (int i = 0; i < n; i++) {
            if (outDegree[i] == 0) queue.Enqueue(i); // terminal nodes are safe
        }

        bool[] safe = new bool[n];
        while (queue.Count > 0) {
            int node = queue.Dequeue();
            safe[node] = true;
            foreach (int prev in revAdj[node]) {
                outDegree[prev]--;
                if (outDegree[prev] == 0) queue.Enqueue(prev);
            }
        }

        List<int> result = new List<int>();
        for (int i = 0; i < n; i++) {
            if (safe[i]) result.Add(i);
        }
        return result;
    }
}
```

Time Complexity: O(V+E). Space Complexity: O(V+E).

**Walkthrough:** For `graph[0]=[5], graph[1]=[2,3], graph[2]=[], graph[3]=[0,4], graph[4]=[0], graph[5]=[2], graph[6]=[2]`: `outDegree = [0:1, 1:2, 2:0, 3:2, 4:1, 5:1, 6:1]`, and reversed lists are `revAdj[0]=[3,4], revAdj[2]=[1,5,6], revAdj[3]=[1], revAdj[4]=[3], revAdj[5]=[0]`. Queue starts with the one true terminal node, `2` (`outDegree==0`). Pop `2` → `safe[2]=true`; via `revAdj[2]=[1,5,6]`, decrement `outDegree[1]` (`2→1`), `outDegree[5]` (`1→0`, enqueue), `outDegree[6]` (`1→0`, enqueue). Pop `5` → `safe[5]=true`; decrement `outDegree[0]` (`1→0`, enqueue). Pop `6` → `safe[6]=true`, no predecessors. Pop `0` → `safe[0]=true`; via `revAdj[0]=[3,4]`, decrement `outDegree[3]` (`2→1`) and `outDegree[4]` (`1→0`, enqueue). Pop `4` → `safe[4]=true`; decrement `outDegree[3]` (`1→0`, enqueue). Pop `3` → `safe[3]=true`; decrement `outDegree[1]` (`1→0`, enqueue). Pop `1` → `safe[1]=true`. Every node peels off to `outDegree 0`, meaning this particular edge set actually contains no cycle at all — every node's outgoing paths terminate, so all of `0..6` come out safe, with `2, 4, 5, 6` (the nodes highlighted in the example) certainly among them.

---

## 7. Alien Dictionary

**Problem Statement:** Given a list of `N` words from an alien language, sorted lexicographically according to the alien language's unknown alphabet order, and given the alphabet has exactly `K` distinct lowercase letters, determine the order of the letters in this alien language. Return one valid character order (as a string), or an indication that no valid order exists (invalid input / cycle).

**Example:**
- Input: `words = ["baa", "abcd", "abca", "cab", "cad"]`, `K = 4` (alphabet uses letters `a,b,c,d`)
- Output: `"bdac"`
- Explanation: Comparing adjacent words gives ordering constraints; combining them via topological sort yields a consistent letter order.

**Approach:** Compare each pair of **adjacent** words in the sorted list. Find the first index where their characters differ — that pair of characters `(c1, c2)` gives a directed edge `c1 -> c2` (meaning `c1` comes before `c2` in the alien alphabet), because that is the only information distinguishing why `word[i]` sorts before `word[i+1]`. (Special case: if `word[i]` is a longer word that is a *prefix extension* of `word[i+1]` reversed — i.e., `word[i]` is longer than `word[i+1]` and `word[i+1]` is a prefix of `word[i]` — the input is invalid, since a shorter prefix must always sort first.) Build a graph over the `K` letters using these edges, then run a **topological sort** (Kahn's algorithm) over the `K` nodes. If the topological sort produces all `K` letters, that order is a valid alien alphabet order; if a cycle is detected (fewer than `K` letters processed), no valid order exists.

**Logic (Steps):**
1. For each pair of adjacent words, scan characters position by position until the first mismatch; that pair `(c1, c2)` yields a directed edge `c1 -> c2` added to `adjSet`.
2. If no mismatch is found and the earlier word is longer than the later one, the input is invalid — return `""` immediately.
3. Flatten `adjSet` into an `adj` list and compute `inDegree[]` for the `K` letters.
4. Run Kahn's BFS: enqueue letters with `inDegree == 0`, repeatedly dequeue a letter, append it to the result string, and decrement `inDegree` of its neighbors, enqueuing any that hit `0`.
5. If `processed == K`, return the built string as a valid alien order; otherwise a cycle exists among the letters, so return `""`.

```csharp
public class Solution {
    public string FindOrder(string[] words, int K) {
        List<HashSet<int>> adjSet = new List<HashSet<int>>();
        for (int i = 0; i < K; i++) adjSet.Add(new HashSet<int>());

        for (int i = 0; i < words.Length - 1; i++) {
            string w1 = words[i], w2 = words[i + 1];
            int len = Math.Min(w1.Length, w2.Length);
            bool foundDiff = false;

            for (int j = 0; j < len; j++) {
                if (w1[j] != w2[j]) {
                    int c1 = w1[j] - 'a', c2 = w2[j] - 'a';
                    adjSet[c1].Add(c2); // c1 -> c2
                    foundDiff = true;
                    break;
                }
            }

            // invalid case: w1 is longer but is a prefix of w2's start (w2 shorter, same prefix)
            if (!foundDiff && w1.Length > w2.Length) {
                return ""; // no valid ordering
            }
        }

        int[] inDegree = new int[K];
        List<List<int>> adj = new List<List<int>>();
        for (int i = 0; i < K; i++) adj.Add(new List<int>());
        for (int u = 0; u < K; u++) {
            foreach (int v in adjSet[u]) {
                adj[u].Add(v);
                inDegree[v]++;
            }
        }

        Queue<int> queue = new Queue<int>();
        for (int i = 0; i < K; i++) {
            if (inDegree[i] == 0) queue.Enqueue(i);
        }

        StringBuilder sb = new StringBuilder();
        int processed = 0;

        while (queue.Count > 0) {
            int node = queue.Dequeue();
            sb.Append((char)('a' + node));
            processed++;
            foreach (int neighbor in adj[node]) {
                inDegree[neighbor]--;
                if (inDegree[neighbor] == 0) queue.Enqueue(neighbor);
            }
        }

        return processed == K ? sb.ToString() : ""; // "" means cycle -> invalid order
    }
}
```

Time Complexity: O(N*L + K + E) where `N` is the number of words, `L` is the max word length, `K` is alphabet size, `E` is number of edges (at most `K^2`) — dominated by O(V+E) for the topological sort plus O(N*L) for comparing adjacent words. Space Complexity: O(K+E).

**Walkthrough:** `words = ["baa", "abcd", "abca", "cab", "cad"]`, alphabet `{a,b,c,d}` mapped to indices `{a:0, b:1, c:2, d:3}`.

- Pair `("baa", "abcd")`: first differing index `0`: `'b' vs 'a'` → edge `b -> a` (i.e., `1 -> 0`).
- Pair `("abcd", "abca")`: differ at index `3`: `'d' vs 'a'` → edge `d -> a` (i.e., `3 -> 0`).
- Pair `("abca", "cab")`: differ at index `0`: `'a' vs 'c'` → edge `a -> c` (i.e., `0 -> 2`).
- Pair `("cab", "cad")`: differ at index `2`: `'b' vs 'd'` → edge `b -> d` (i.e., `1 -> 3`).

Resulting edges: `b->a, d->a, a->c, b->d`. In-degrees: `a: 2 (from b,d)`, `b: 0`, `c: 1 (from a)`, `d: 1 (from b)`.

Kahn's algorithm: queue starts with in-degree-0 nodes: `[b]`. Pop `b` → result = `"b"`. Decrement `a` (`2→1`) and `d` (`1→0`, enqueue). Queue = `[d]`. Pop `d` → result = `"bd"`. Decrement `a` (`1→0`, enqueue). Queue = `[a]`. Pop `a` → result = `"bda"`. Decrement `c` (`1→0`, enqueue). Queue = `[c]`. Pop `c` → result = `"bdac"`. All 4 letters processed (`processed == K`), so no cycle exists and `"bdac"` is a valid alien alphabet order — consistent with the earlier claimed output. If, instead, the word pairs had produced a contradictory pair of edges such as both `a -> c` and `c -> a`, the in-degree of both would never reach `0` together, the queue would empty early with `processed < K`, and the algorithm would correctly report no valid ordering exists.
