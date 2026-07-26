# Graphs — Grid and Matrix Graph Problems

## 1. 0/1 Matrix — Distance of Nearest 0 for Each Cell (Multi-source BFS)

**Problem Statement:**
Given an `n x m` binary matrix containing only `0`s and `1`s, return a matrix of the same size where each cell contains the distance (number of moves, up/down/left/right only) to the nearest `0` cell in the original matrix.

**Example:**
- Input:
  ```
  0 0 0
  0 1 0
  1 1 1
  ```
- Output:
  ```
  0 0 0
  0 1 0
  1 2 1
  ```
- Explanation: Every `0` cell has distance `0`. The `1` at `(1,1)` is adjacent to `0`s, so distance `1`. The `1` at `(2,1)` is two steps away from the nearest `0` (via `(1,1)` or `(2,0)`/`(2,2)` are also `1`s), so its distance is `2`.

**Brute Force Approach:**
For every cell, run a BFS/DFS to find the nearest `0`. This means starting a fresh BFS from each of the `n*m` cells, each BFS potentially visiting all `n*m` cells, giving `O((n*m)^2)` time.

```csharp
public class Solution 
{
    public int[,] NearestZeroBruteForce(int[,] mat)
    {
        int n = mat.GetLength(0);
        int m = mat.GetLength(1);
        int[,] result = new int[n, m];
        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (mat[i, j] == 0)
                {
                    result[i, j] = 0;
                    continue;
                }

                // BFS from (i, j) to find nearest 0
                bool[,] visited = new bool[n, m];
                Queue<(int r, int c, int dist)> q = new Queue<(int, int, int)>();
                q.Enqueue((i, j, 0));
                visited[i, j] = true;
                int found = -1;

                while (q.Count > 0)
                {
                    var (r, c, dist) = q.Dequeue();
                    if (mat[r, c] == 0)
                    {
                        found = dist;
                        break;
                    }

                    for (int d = 0; d < 4; d++)
                    {
                        int nr = r + dx[d];
                        int nc = c + dy[d];
                        if (nr >= 0 && nr < n && nc >= 0 && nc < m && !visited[nr, nc])
                        {
                            visited[nr, nc] = true;
                            q.Enqueue((nr, nc, dist + 1));
                        }
                    }
                }

                result[i, j] = found;
            }
        }

        return result;
    }
}
```

Time Complexity: `O((n*m)^2)` — a separate BFS over the whole grid for every cell.
Space Complexity: `O(n*m)` for the visited array and queue used per BFS call.

**Optimized Approach:**
Instead of searching outward from every `1`, search outward from every `0` simultaneously — a **multi-source BFS**. Push all `0` cells into the queue first (distance `0`), mark them visited, then do a standard BFS. Because all sources start at the same "time," the first time a cell is reached is guaranteed to be its shortest distance to *any* `0`.

```csharp
public class Solution 
{
    public int[,] NearestZeroMultiSourceBFS(int[,] mat)
    {
        int n = mat.GetLength(0);
        int m = mat.GetLength(1);
        int[,] dist = new int[n, m];
        bool[,] visited = new bool[n, m];
        Queue<(int r, int c)> q = new Queue<(int, int)>();

        // Push all 0-cells as initial sources
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (mat[i, j] == 0)
                {
                    q.Enqueue((i, j));
                    visited[i, j] = true;
                    dist[i, j] = 0;
                }
            }
        }

        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };

        while (q.Count > 0)
        {
            var (r, c) = q.Dequeue();
            for (int d = 0; d < 4; d++)
            {
                int nr = r + dx[d];
                int nc = c + dy[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m && !visited[nr, nc])
                {
                    visited[nr, nc] = true;
                    dist[nr, nc] = dist[r, c] + 1;
                    q.Enqueue((nr, nc));
                }
            }
        }

        return dist;
    }
}
```

Time Complexity: `O(n*m)` — every cell is enqueued and dequeued exactly once, and each dequeue does O(1) work (4 neighbor checks).
Space Complexity: `O(n*m)` for the `visited`, `dist` arrays and the BFS queue.

**Explanation:**
Dry run on:
```
0 1 1
1 1 1
1 1 1
```
Step 0 (initialization): Only `(0,0)` is a `0`. Queue = `[(0,0)]`, `dist[0,0] = 0`, all others unvisited.

Layer 1 (process `(0,0)`): neighbors `(0,1)` and `(1,0)` are unvisited → mark visited, `dist = 1`, enqueue both. Queue = `[(0,1), (1,0)]`.

Layer 2 (process `(0,1)`): neighbors `(0,2)` unvisited → `dist = 2`; `(1,1)` unvisited → `dist = 2`. Enqueue both.
(process `(1,0)`): neighbor `(1,1)` already visited (skip), `(2,0)` unvisited → `dist = 2`. Queue = `[(0,2), (1,1), (2,0)]`.

Layer 3 (process `(0,2)`): neighbor `(1,2)` unvisited → `dist = 3`.
(process `(1,1)`): neighbors `(1,2)` already queued (skip if marked visited when enqueued), `(2,1)` unvisited → `dist = 3`.
(process `(2,0)`): neighbor `(2,1)` already visited (skip).

Layer 4 (process remaining): `(2,2)` gets `dist = 4` once reached from `(1,2)` or `(2,1)`.

Final distances:
```
0 1 2
1 2 3
2 3 4
```
Because BFS expands one "ring" of distance at a time from all `0`s simultaneously, the first visit to any cell is its true shortest distance — no revisiting or re-relaxation is ever needed, unlike the brute-force per-cell BFS.

---

## 2. Surrounded Regions (Replace 'O's Surrounded by 'X's)

**Problem Statement:**
Given an `m x n` matrix containing `'X'` and `'O'`, capture (flip to `'X'`) all regions of `'O'` that are completely surrounded by `'X'`. A region of `'O'`s is **not** captured if it is connected (directly or indirectly, via up/down/left/right moves) to an `'O'` on the border of the board.

**Example:**
- Input:
  ```
  X X X X
  X O O X
  X X O X
  X O X X
  ```
- Output:
  ```
  X X X X
  X X X X
  X X X X
  X O X X
  ```
- Explanation: The `O`s at `(1,1)`, `(1,2)`, `(2,2)` form a region not touching the border, so they get flipped to `X`. The `O` at `(3,1)` touches the border (row 3 is the last row), so it survives.

**Brute Force Approach:**
For every internal `'O'` cell, run a BFS/DFS to check if that region can reach the boundary. If it cannot, flip the whole region. This repeats work because cells belonging to the same region get re-explored, and in the worst case each of the `O(n*m)` cells triggers a traversal of size `O(n*m)`, giving `O((n*m)^2)`.

```csharp
public class Solution 
{
    public void SolveBruteForce(char[,] board)
    {
        int n = board.GetLength(0);
        int m = board.GetLength(1);
        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (board[i, j] != 'O') continue;

                // BFS this region, check if it touches the boundary
                List<(int, int)> region = new List<(int, int)>();
                bool[,] visited = new bool[n, m];
                Queue<(int, int)> q = new Queue<(int, int)>();
                q.Enqueue((i, j));
                visited[i, j] = true;
                bool touchesBoundary = false;

                while (q.Count > 0)
                {
                    var (r, c) = q.Dequeue();
                    region.Add((r, c));
                    if (r == 0 || r == n - 1 || c == 0 || c == m - 1) touchesBoundary = true;

                    for (int d = 0; d < 4; d++)
                    {
                        int nr = r + dx[d];
                        int nc = c + dy[d];
                        if (nr >= 0 && nr < n && nc >= 0 && nc < m &&
                            !visited[nr, nc] && board[nr, nc] == 'O')
                        {
                            visited[nr, nc] = true;
                            q.Enqueue((nr, nc));
                        }
                    }
                }

                if (!touchesBoundary)
                {
                    foreach (var (r, c) in region) board[r, c] = 'X';
                }
                else
                {
                    // Mark temporarily so we don't reprocess this region
                    foreach (var (r, c) in region) board[r, c] = 'B'; // 'B' = border-connected, restore later
                }
            }
        }

        // Restore 'B' back to 'O'
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (board[i, j] == 'B') board[i, j] = 'O';
    }
}
```

Time Complexity: `O((n*m)^2)` in the worst case, since every cell can trigger a region traversal proportional to the grid size.
Space Complexity: `O(n*m)` for visited array, region list, and queue.

**Optimized Approach:**
Only the `'O'`s connected to the border can possibly survive. So run BFS/DFS starting from every border `'O'` cell exactly once, marking all reachable `'O'`s as "safe" (e.g., temporarily change them to `'#'`/`'S'`). After that single multi-source traversal, scan the whole grid: any remaining `'O'` (never marked safe) gets flipped to `'X'`, and every safe-marked cell gets restored back to `'O'`.

```csharp
public class Solution 
{
    public void Solve(char[,] board)
    {
        int n = board.GetLength(0);
        int m = board.GetLength(1);
        if (n == 0 || m == 0) return;

        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };
        Queue<(int r, int c)> q = new Queue<(int, int)>();

        // Enqueue all border 'O' cells as BFS sources
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                bool isBorder = (i == 0 || i == n - 1 || j == 0 || j == m - 1);
                if (isBorder && board[i, j] == 'O')
                {
                    q.Enqueue((i, j));
                    board[i, j] = '#'; // mark safe
                }
            }
        }

        // Multi-source BFS marking every 'O' reachable from the border as safe
        while (q.Count > 0)
        {
            var (r, c) = q.Dequeue();
            for (int d = 0; d < 4; d++)
            {
                int nr = r + dx[d];
                int nc = c + dy[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m && board[nr, nc] == 'O')
                {
                    board[nr, nc] = '#';
                    q.Enqueue((nr, nc));
                }
            }
        }

        // Final pass: flip unsafe 'O' -> 'X', restore safe '#' -> 'O'
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (board[i, j] == 'O') board[i, j] = 'X';
                else if (board[i, j] == '#') board[i, j] = 'O';
            }
        }
    }
}
```

Time Complexity: `O(n*m)` — each cell is enqueued at most once during BFS, plus one final `O(n*m)` scan.
Space Complexity: `O(n*m)` for the BFS queue in the worst case (all cells are `'O'`); no separate visited array is needed since the board itself is mutated to mark state.

**Explanation:**
Dry run island-labeling-style reasoning is covered in problem 6; here is a quick dry run of the border-BFS marking for problem 2 on:
```
X O X
O O X
X X X
```
Border cells: `(0,1)`='O', `(1,0)`='O' (border since column 0), all of row 2 is 'X'. Enqueue `(0,1)` and `(1,0)`, mark both as `'#'`.

Process `(0,1)`: neighbors `(1,1)`='O' → mark `'#'`, enqueue.
Process `(1,0)`: neighbors `(1,1)` already `'#'` (skip).
Process `(1,1)`: neighbors `(0,1)` already `'#'`, `(1,0)` already `'#'`, `(1,2)`='X' (skip), `(2,1)`='X' (skip).

Queue empties. Board now:
```
X # X
# # X
X X X
```
Final pass: no leftover `'O'` cells remain (all were reachable from the border), so nothing gets flipped to `'X'`; every `'#'` reverts to `'O'`, giving back the original board unchanged — correct, since this entire region touches the border.

---

## 3. Number of Enclaves (Count Land Cells That Can't Reach the Boundary)

**Problem Statement:**
Given an `m x n` binary matrix where `1` represents land and `0` represents water, a move consists of walking from one land cell to another adjacent (up/down/left/right) land cell, or walking off the boundary of the grid. Return the number of land cells for which you **cannot** walk off the boundary of the grid in any number of moves (i.e., land cells that never touch or connect to the border).

**Example:**
- Input:
  ```
  0 0 0 0
  1 0 1 0
  0 1 1 0
  0 0 0 0
  ```
- Output: `3`
- Explanation: The land cells at `(1,2)`, `(2,1)`, `(2,2)` form a connected component that never touches the boundary, so all 3 are enclaves. The cell `(1,0)` touches the boundary directly (column 0), so it and anything connected to it are excluded.

**Brute Force Approach:**
For every land cell, run BFS/DFS to see whether its connected component reaches the boundary. As with Surrounded Regions, this re-explores the same components repeatedly, costing `O((n*m)^2)` in the worst case.

```csharp
public class Solution 
{
    public int NumEnclavesBruteForce(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };
        int enclaveCount = 0;

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] != 1) continue;

                List<(int, int)> component = new List<(int, int)>();
                bool[,] visited = new bool[n, m];
                Queue<(int, int)> q = new Queue<(int, int)>();
                q.Enqueue((i, j));
                visited[i, j] = true;
                bool touchesBoundary = false;

                while (q.Count > 0)
                {
                    var (r, c) = q.Dequeue();
                    component.Add((r, c));
                    if (r == 0 || r == n - 1 || c == 0 || c == m - 1) touchesBoundary = true;

                    for (int d = 0; d < 4; d++)
                    {
                        int nr = r + dx[d];
                        int nc = c + dy[d];
                        if (nr >= 0 && nr < n && nc >= 0 && nc < m &&
                            !visited[nr, nc] && grid[nr, nc] == 1)
                        {
                            visited[nr, nc] = true;
                            q.Enqueue((nr, nc));
                        }
                    }
                }

                if (!touchesBoundary) enclaveCount += component.Count;

                // Prevent recounting this component: mark visited cells as water temporarily
                foreach (var (r, c) in component) grid[r, c] = 2; // processed marker
            }
        }

        // Restore grid (2 -> 1)
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (grid[i, j] == 2) grid[i, j] = 1;

        return enclaveCount;
    }
}
```

Time Complexity: `O(n*m)` amortized here because of the "mark processed" trick, but conceptually `O((n*m)^2)` if components were not marked to avoid reprocessing — included as the naive baseline before the multi-source boundary optimization below.
Space Complexity: `O(n*m)` for visited array and queue/component storage.

**Optimized Approach:**
Run a single multi-source BFS/DFS starting from every land cell that lies on the boundary. Mark every land cell reachable from these boundary cells as "escaping" (cannot be an enclave). Then scan the whole grid once: count land cells that were never marked as escaping — those are the enclaves.

```csharp
public class Solution 
{
    public int NumEnclaves(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        bool[,] visited = new bool[n, m];
        Queue<(int r, int c)> q = new Queue<(int, int)>();
        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };

        // Seed BFS with all boundary land cells
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                bool isBorder = (i == 0 || i == n - 1 || j == 0 || j == m - 1);
                if (isBorder && grid[i, j] == 1 && !visited[i, j])
                {
                    visited[i, j] = true;
                    q.Enqueue((i, j));
                }
            }
        }

        // Multi-source BFS marking every land cell reachable from the boundary
        while (q.Count > 0)
        {
            var (r, c) = q.Dequeue();
            for (int d = 0; d < 4; d++)
            {
                int nr = r + dx[d];
                int nc = c + dy[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m &&
                    !visited[nr, nc] && grid[nr, nc] == 1)
                {
                    visited[nr, nc] = true;
                    q.Enqueue((nr, nc));
                }
            }
        }

        // Count land cells never reached from the boundary
        int enclaveCount = 0;
        for (int i = 0; i < n; i++)
            for (int j = 0; j < m; j++)
                if (grid[i, j] == 1 && !visited[i, j]) enclaveCount++;

        return enclaveCount;
    }
}
```

Time Complexity: `O(n*m)` — each land cell is enqueued and processed at most once, plus one final `O(n*m)` scan.
Space Complexity: `O(n*m)` for the `visited` array and BFS queue.

**Explanation:**
Dry run on the example grid:
```
0 0 0 0
1 0 1 0
0 1 1 0
0 0 0 0
```
Boundary land cells: only `(1,0)` is land and on the border (column 0). Enqueue `(1,0)`, `visited[1,0] = true`.

Process `(1,0)`: neighbors `(0,0)`=0 skip, `(2,0)`=0 skip, `(1,1)`=0 skip. No new land reached. Queue empties.

`visited` after BFS: only `(1,0)` is true.

Final scan counts land cells (`grid[i,j]==1`) that are **not** visited: `(1,2)`, `(2,1)`, `(2,2)` — 3 cells. `(1,0)` is land but visited, so excluded. Result = `3`, matching the expected output. Note `(1,2)` and `(2,1)`/`(2,2)` are connected to each other but never to `(1,0)` or the boundary, so the multi-source BFS correctly never reaches them.

---

## 4. Word Ladder I and II (Transform One Word to Another Changing One Letter at a Time via a Word List)

**Problem Statement:**
**Word Ladder I:** Given a `beginWord`, an `endWord`, and a dictionary `wordList`, find the length of the shortest transformation sequence from `beginWord` to `endWord`, such that only one letter can be changed at a time, and each transformed word must exist in `wordList`. Return `0` if no such sequence exists.

**Word Ladder II:** Return **all** the shortest transformation sequences from `beginWord` to `endWord` (each sequence is a list of words), using the same one-letter-change rule and dictionary constraint.

**Example:**
- Input: `beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log","cog"]`
- Output (Word Ladder I): `5`
- Output (Word Ladder II): `[["hit","hot","dot","dog","cog"], ["hit","hot","lot","log","cog"]]`
- Explanation: `hit -> hot -> dot -> dog -> cog` changes one letter each step and every intermediate word is in the list, taking 5 words (4 transformations). `hit -> hot -> lot -> log -> cog` is an equally short alternative path, so Word Ladder II returns both.

**Brute Force Approach:**
For Word Ladder I, do a BFS from `beginWord`, but at each step scan the *entire* word list to find every word that differs by exactly one character from the current word (instead of generating candidates by mutating each character position). This is `O(N^2 * L)` where `N` is the number of words and `L` is word length, since for each of the `N` words in the BFS we compare against all `N` other words, each comparison costing `O(L)`.

```csharp
public class Solution 
{
    public int LadderLengthBruteForce(string beginWord, string endWord, IList<string> wordList)
    {
        HashSet<string> unvisited = new HashSet<string>(wordList);
        if (!unvisited.Contains(endWord)) return 0;

        Queue<(string word, int steps)> q = new Queue<(string, int)>();
        q.Enqueue((beginWord, 1));

        while (q.Count > 0)
        {
            var (word, steps) = q.Dequeue();
            if (word == endWord) return steps;

            // Brute force: scan every remaining word, check one-letter difference
            List<string> toRemove = new List<string>();
            foreach (string candidate in unvisited)
            {
                if (IsOneLetterDiff(word, candidate))
                {
                    q.Enqueue((candidate, steps + 1));
                    toRemove.Add(candidate);
                }
            }
            foreach (string r in toRemove) unvisited.Remove(r);
        }

        return 0;
    }

    private bool IsOneLetterDiff(string a, string b)
    {
        if (a.Length != b.Length) return false;
        int diff = 0;
        for (int i = 0; i < a.Length; i++)
        {
            if (a[i] != b[i])
            {
                diff++;
                if (diff > 1) return false;
            }
        }
        return diff == 1;
    }
}
```

Time Complexity: `O(N^2 * L)` — for every word popped from the queue (up to `N`), we scan the remaining unvisited set (up to `N`) and compare strings of length `L`.
Space Complexity: `O(N * L)` for the word set and queue.

**Optimized Approach:**
Instead of comparing against every other word, generate candidates by mutating each of the `L` positions of the current word through all 26 letters (`O(L * 26)` per word) and checking membership in an `O(1)` `HashSet` lookup. This turns candidate generation from `O(N)` per word into `O(L * 26)` per word.

For **Word Ladder I**, plain BFS layer by layer gives the shortest length directly.

For **Word Ladder II**, BFS level by level while recording, for each word, the set of predecessor words that reach it at the current shortest level (a parent map). Once `endWord` is reached, backtrack from `endWord` through the parent map to reconstruct all shortest paths.

```csharp
public class Solution 
{
    // Word Ladder I: shortest transformation length
    public int LadderLength(string beginWord, string endWord, IList<string> wordList)
    {
        HashSet<string> dict = new HashSet<string>(wordList);
        if (!dict.Contains(endWord)) return 0;

        Queue<(string word, int steps)> q = new Queue<(string, int)>();
        q.Enqueue((beginWord, 1));
        HashSet<string> visited = new HashSet<string> { beginWord };

        while (q.Count > 0)
        {
            var (word, steps) = q.Dequeue();
            if (word == endWord) return steps;

            char[] chars = word.ToCharArray();
            for (int i = 0; i < chars.Length; i++)
            {
                char original = chars[i];
                for (char c = 'a'; c <= 'z'; c++)
                {
                    if (c == original) continue;
                    chars[i] = c;
                    string candidate = new string(chars);
                    if (dict.Contains(candidate) && !visited.Contains(candidate))
                    {
                        visited.Add(candidate);
                        q.Enqueue((candidate, steps + 1));
                    }
                }
                chars[i] = original;
            }
        }

        return 0;
    }

    // Word Ladder II: all shortest transformation sequences
    public IList<IList<string>> FindLadders(string beginWord, string endWord, IList<string> wordList)
    {
        IList<IList<string>> result = new List<IList<string>>();
        HashSet<string> dict = new HashSet<string>(wordList);
        if (!dict.Contains(endWord)) return result;

        // parents[word] = set of words that lead to 'word' on a shortest path
        Dictionary<string, List<string>> parents = new Dictionary<string, List<string>>();
        HashSet<string> currentLevel = new HashSet<string> { beginWord };
        HashSet<string> visitedOverall = new HashSet<string> { beginWord };
        bool found = false;

        while (currentLevel.Count > 0 && !found)
        {
            // Remove this level's words from dict so they aren't reused as candidates
            foreach (string w in currentLevel) dict.Remove(w);

            HashSet<string> nextLevel = new HashSet<string>();

            foreach (string word in currentLevel)
            {
                char[] chars = word.ToCharArray();
                for (int i = 0; i < chars.Length; i++)
                {
                    char original = chars[i];
                    for (char c = 'a'; c <= 'z'; c++)
                    {
                        if (c == original) continue;
                        chars[i] = c;
                        string candidate = new string(chars);

                        if (dict.Contains(candidate))
                        {
                            if (candidate == endWord) found = true;

                            nextLevel.Add(candidate);
                            if (!parents.ContainsKey(candidate))
                                parents[candidate] = new List<string>();
                            parents[candidate].Add(word);
                        }
                    }
                    chars[i] = original;
                }
            }

            currentLevel = nextLevel;
        }

        if (!found) return result;

        // Backtrack from endWord to beginWord using the parent map
        List<string> path = new List<string> { endWord };
        Backtrack(endWord, beginWord, parents, path, result);

        return result;
    }

    private void Backtrack(string word, string beginWord, Dictionary<string, List<string>> parents,
        List<string> path, IList<IList<string>> result)
    {
        if (word == beginWord)
        {
            List<string> reversed = new List<string>(path);
            reversed.Reverse();
            result.Add(reversed);
            return;
        }

        if (!parents.ContainsKey(word)) return;

        foreach (string parent in parents[word])
        {
            path.Add(parent);
            Backtrack(parent, beginWord, parents, path, result);
            path.RemoveAt(path.Count - 1);
        }
    }
}
```

Time Complexity: Word Ladder I is `O(N * L * 26)` — each of up to `N` words generates `L * 26` candidates, each checked in `O(1)` (amortized `O(L)` for hashing/string build, so more precisely `O(N * L^2 * 26)` including string construction cost). Word Ladder II has the same BFS cost for building `parents`, plus the backtracking reconstruction cost which is proportional to the total number and length of shortest paths found (can be exponential in the worst case, but bounded by the actual output size).
Space Complexity: `O(N * L)` for the dictionary/visited sets, plus `O(N)` for the `parents` map (each word stores its predecessor list), plus `O(P * L)` for storing `P` output paths of length up to `L` words each.

**Explanation:**
Word Ladder I dry run on `beginWord = "hit"`, `endWord = "cog"`, dict = `{hot, dot, dog, lot, log, cog}`:

Level 1: Queue = `[("hit", 1)]`. Process `"hit"`: mutate each position through a-z. Position 0: `"ait"`..`"zit"` — none in dict. Position 1: `"hot"` found in dict, not visited → enqueue `("hot", 2)`, mark visited. Position 2: `"hia"`..`"hiz"` — none match. Queue = `[("hot", 2)]`.

Level 2: Process `"hot"`: generates `"dot"` and `"lot"` (both in dict, unvisited) → enqueue `("dot", 3)`, `("lot", 3)`. Also `"hog"`? Not in dict here. Queue = `[("dot",3), ("lot",3)]`.

Level 3: Process `"dot"` → generates `"dog"` → enqueue `("dog", 4)`. Process `"lot"` → generates `"log"` → enqueue `("log", 4)`. Queue = `[("dog",4), ("log",4)]`.

Level 4: Process `"dog"` → generates `"cog"` → enqueue `("cog", 5)`. Process `"log"` → generates `"cog"` but already visited, skip. Queue = `[("cog", 5)]`.

Level 5: Process `"cog"`: `word == endWord` → return `5`. Matches expected output.

For Word Ladder II on the same input, the level-by-level BFS additionally records `parents["hot"] = ["hit"]`, `parents["dot"] = ["hot"]`, `parents["lot"] = ["hot"]`, `parents["dog"] = ["dot"]`, `parents["log"] = ["lot"]`, `parents["cog"] = ["dog", "log"]` (both `"dog"` and `"log"` reach `"cog"` at the same shortest level, so both are kept as parents — this is why removing a level's words from `dict` only happens *after* fully processing that level, so same-level candidates aren't missed). Backtracking from `"cog"` follows both parent branches: `cog -> dog -> dot -> hot -> hit` and `cog -> log -> lot -> hot -> hit`, reversed to give the two expected shortest paths.

---

## 5. Number of Distinct Islands (Count Islands Considering Shape, Not Just Count)

**Problem Statement:**
Given an `m x n` binary grid where `1` represents land and `0` represents water, an island is a maximal group of `1`s connected 4-directionally. Return the number of **distinct** islands, where two islands are considered the same if one can be translated (shifted, not rotated or reflected) to exactly match the other's shape.

**Example:**
- Input:
  ```
  1 1 0 0 1 1
  1 0 0 0 0 1
  0 0 0 0 0 0
  1 1 0 0 1 1
  1 0 0 0 0 1
  ```
- Output: `1`
- Explanation: There are 4 separate islands (top-left, top-right, bottom-left, bottom-right), but every one has the exact same "L-shape" (two cells forming an inverted-L). Since shape matters and not position, they all count as a single distinct shape.

**Brute Force Approach:**
Find each island's absolute cell coordinates via BFS/DFS, then normalize the shape by subtracting the coordinates of some anchor cell (e.g., the top-left-most cell of that island) from every cell so that islands can be compared regardless of position. Store each normalized shape (a sorted list/set of relative coordinates) into a collection and compare it against every previously found shape using full set equality — an `O(k^2)` comparison across `k` islands where each comparison is `O(size of island)`.

```csharp
public class Solution 
{
    public int NumDistinctIslandsBruteForce(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        bool[,] visited = new bool[n, m];
        List<List<(int, int)>> islandShapes = new List<List<(int, int)>>();

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] == 1 && !visited[i, j])
                {
                    List<(int, int)> cells = new List<(int, int)>();
                    CollectIsland(grid, visited, i, j, n, m, cells);

                    // Normalize relative to top-left anchor (min row, min col among island cells)
                    int minR = int.MaxValue, minC = int.MaxValue;
                    foreach (var (r, c) in cells)
                    {
                        minR = Math.Min(minR, r);
                        minC = Math.Min(minC, c);
                    }
                    List<(int, int)> normalized = new List<(int, int)>();
                    foreach (var (r, c) in cells) normalized.Add((r - minR, c - minC));
                    normalized.Sort();

                    islandShapes.Add(normalized);
                }
            }
        }

        // Compare every shape against every other shape (O(k^2))
        List<List<(int, int)>> distinctShapes = new List<List<(int, int)>>();
        foreach (var shape in islandShapes)
        {
            bool isDuplicate = false;
            foreach (var existing in distinctShapes)
            {
                if (ShapesEqual(shape, existing)) { isDuplicate = true; break; }
            }
            if (!isDuplicate) distinctShapes.Add(shape);
        }

        return distinctShapes.Count;
    }

    private void CollectIsland(int[,] grid, bool[,] visited, int r, int c, int n, int m, List<(int, int)> cells)
    {
        if (r < 0 || r >= n || c < 0 || c >= m || visited[r, c] || grid[r, c] == 0) return;
        visited[r, c] = true;
        cells.Add((r, c));
        CollectIsland(grid, visited, r - 1, c, n, m, cells);
        CollectIsland(grid, visited, r + 1, c, n, m, cells);
        CollectIsland(grid, visited, r, c - 1, n, m, cells);
        CollectIsland(grid, visited, r, c + 1, n, m, cells);
    }

    private bool ShapesEqual(List<(int, int)> a, List<(int, int)> b)
    {
        if (a.Count != b.Count) return false;
        for (int i = 0; i < a.Count; i++)
            if (a[i] != b[i]) return false;
        return true;
    }
}
```

Time Complexity: `O(n*m)` to find all islands via DFS, plus `O(k^2 * s)` to deduplicate shapes, where `k` is the number of islands and `s` is average island size — worst case `O((n*m)^2)` when there are many islands.
Space Complexity: `O(n*m)` for visited array and storing all island cell lists.

**Optimized Approach:**
Instead of storing raw coordinates and comparing shapes pairwise, encode each island's shape as a canonical **string signature** built from the sequence of DFS moves relative to the starting cell (e.g., appending a direction character like `'U'`, `'D'`, `'L'`, `'R'` when moving into a neighbor, and a distinct backtrack marker like `'B'` when returning from recursion so that shape ambiguity is avoided). Insert each signature into a `HashSet<string>` — duplicates collapse automatically, and the final answer is just the set's size. This avoids pairwise comparison entirely.

```csharp
public class Solution 
{
    public int NumDistinctIslands(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        bool[,] visited = new bool[n, m];
        HashSet<string> shapes = new HashSet<string>();

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] == 1 && !visited[i, j])
                {
                    StringBuilder sb = new StringBuilder();
                    Dfs(grid, visited, i, j, n, m, sb, 'S'); // 'S' = start marker
                    shapes.Add(sb.ToString());
                }
            }
        }

        return shapes.Count;
    }

    private void Dfs(int[,] grid, bool[,] visited, int r, int c, int n, int m, StringBuilder sb, char moveDir)
    {
        if (r < 0 || r >= n || c < 0 || c >= m || visited[r, c] || grid[r, c] == 0) return;

        visited[r, c] = true;
        sb.Append(moveDir); // record the move that got us here

        Dfs(grid, visited, r - 1, c, n, m, sb, 'U');
        Dfs(grid, visited, r + 1, c, n, m, sb, 'D');
        Dfs(grid, visited, r, c - 1, n, m, sb, 'L');
        Dfs(grid, visited, r, c + 1, n, m, sb, 'R');

        sb.Append('B'); // backtrack marker distinguishes shapes that would otherwise collide
    }
}
```

Time Complexity: `O(n*m)` — each cell is visited once by DFS, and each visit does `O(1)` amortized string-append work (StringBuilder append is O(1) amortized).
Space Complexity: `O(n*m)` for the visited array and for storing all shape signatures in the worst case (every cell is its own island, each signature of length O(1)); more generally `O(total land cells)` for signature storage.

**Explanation:**
Dry run multi-source BFS is for problem 1; here instead is the shape-signature DFS dry run for problem 5, then the island-labeling dry run for problem 6 below (as requested for problem 6).

Distinct-islands dry run on:
```
1 1 0
0 1 0
0 0 1
```
Start at `(0,0)`, unvisited land. Call `Dfs(0,0,'S')`: append `'S'` → sig = `"S"`. Try Up `(-1,0)` invalid, skip (no append since it returns immediately without visiting). Try Down `(1,0)`: `grid[1,0]=0`, skip. Try Left `(0,-1)` invalid, skip. Try Right `(0,1)`: land, unvisited → `Dfs(0,1,'R')`: append `'R'` → sig = `"SR"`. From `(0,1)`: Up invalid, Down `(1,1)`: land, unvisited → `Dfs(1,1,'D')`: append `'D'` → sig = `"SRD"`. From `(1,1)`: Up `(0,1)` visited skip, Down `(2,1)`=0 skip, Left `(1,0)`=0 skip, Right `(1,2)`=0 skip. No further recursion → append backtrack `'B'` → sig = `"SRDB"`. Return to `(0,1)` frame: Left `(0,0)` visited skip, Right `(0,2)`=0 skip. Append `'B'` → sig = `"SRDBB"`. Return to `(0,0)` frame: append `'B'` → sig = `"SRDBBB"`.

First island's signature: `"SRDBBB"`. Add to `HashSet`.

Second island starts at `(2,2)` (the isolated `1`): `Dfs(2,2,'S')` → append `'S'`; all 4 neighbors are either out of bounds or water, so immediately append `'B'` → signature `"SB"`. Add to set (distinct from `"SRDBBB"`).

Final `shapes` set = `{"SRDBBB", "SB"}`, size `2` — correctly identifying two shape-distinct islands (an L-tromino shape and a single-cell shape).

---

## 6. Making a Large Island (Flip At Most One 0 to Maximize the Resulting Island Size)

**Problem Statement:**
Given an `n x n` binary grid, you may change **at most one** `0` to a `1`. Return the size of the largest possible island after doing so (an island is a group of `1`s connected 4-directionally). If no `0` exists (or flipping doesn't help), the answer may simply be the size of the whole grid or the largest existing island.

**Example:**
- Input:
  ```
  1 0
  0 1
  ```
- Output: `3`
- Explanation: Flipping the `0` at `(0,1)` connects the two `1`-islands (sizes 1 and 1) into one island of size `1 + 1 + 1(the flipped cell) = 3`. Flipping `(1,0)` gives the same result. Flipping either produces a bigger island than doing nothing (max original island size = 1).

**Brute Force Approach:**
For every `0` cell in the grid, temporarily flip it to `1`, then run a full BFS/DFS over the grid to compute the size of the island that cell now belongs to, then flip it back. Track the maximum size seen. This redoes a full grid traversal for every candidate `0`, giving `O((n*m)^2)`.

```csharp
public class Solution 
{
    public int LargestIslandBruteForce(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        int best = 0;
        bool anyZero = false;

        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] == 0)
                {
                    anyZero = true;
                    grid[i, j] = 1; // temporarily flip

                    bool[,] visited = new bool[n, m];
                    int size = BfsSize(grid, visited, i, j, n, m);
                    best = Math.Max(best, size);

                    grid[i, j] = 0; // revert
                }
            }
        }

        if (!anyZero)
        {
            // Entire grid is land already
            return n * m;
        }

        return best;
    }

    private int BfsSize(int[,] grid, bool[,] visited, int startR, int startC, int n, int m)
    {
        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };
        Queue<(int, int)> q = new Queue<(int, int)>();
        q.Enqueue((startR, startC));
        visited[startR, startC] = true;
        int count = 0;

        while (q.Count > 0)
        {
            var (r, c) = q.Dequeue();
            count++;
            for (int d = 0; d < 4; d++)
            {
                int nr = r + dx[d];
                int nc = c + dy[d];
                if (nr >= 0 && nr < n && nc >= 0 && nc < m && !visited[nr, nc] && grid[nr, nc] == 1)
                {
                    visited[nr, nc] = true;
                    q.Enqueue((nr, nc));
                }
            }
        }

        return count;
    }
}
```

Time Complexity: `O((n*m)^2)` — for each of up to `n*m` zero-cells, a full `O(n*m)` BFS is run.
Space Complexity: `O(n*m)` for the visited array used per BFS call.

**Optimized Approach:**
First pass: DFS/BFS over the whole grid once, assigning each island a unique integer id (starting at 2, since 0 and 1 are already used) and recording each id's size in a dictionary/array. Second pass: for every `0` cell, look at its (up to 4) neighboring island ids, **deduplicate them using a HashSet** (so an island touched from two different directions by the same `0` cell isn't double-counted), sum the sizes of the unique neighboring islands, add 1 for the flipped cell itself, and track the maximum. If the grid has no `0` cells at all, the answer is simply `n*m`.

```csharp
public class Solution 
{
    public int LargestIsland(int[,] grid)
    {
        int n = grid.GetLength(0);
        int m = grid.GetLength(1);
        int[,] labels = new int[n, m]; // 0 = water, >=2 = island id
        Dictionary<int, int> islandSize = new Dictionary<int, int>();
        int nextId = 2;

        int[] dx = { -1, 1, 0, 0 };
        int[] dy = { 0, 0, -1, 1 };

        // Pass 1: label every island with a unique id and compute its size
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] == 1 && labels[i, j] == 0)
                {
                    int size = LabelIsland(grid, labels, i, j, n, m, nextId);
                    islandSize[nextId] = size;
                    nextId++;
                }
            }
        }

        int best = 0;
        foreach (var kvp in islandSize) best = Math.Max(best, kvp.Value);

        bool hasZero = false;

        // Pass 2: try flipping each 0-cell
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                if (grid[i, j] != 0) continue;
                hasZero = true;

                HashSet<int> neighborIds = new HashSet<int>();
                for (int d = 0; d < 4; d++)
                {
                    int nr = i + dx[d];
                    int nc = j + dy[d];
                    if (nr >= 0 && nr < n && nc >= 0 && nc < m && labels[nr, nc] >= 2)
                    {
                        neighborIds.Add(labels[nr, nc]); // dedupe so same island isn't counted twice
                    }
                }

                int total = 1; // the flipped cell itself
                foreach (int id in neighborIds) total += islandSize[id];

                best = Math.Max(best, total);
            }
        }

        if (!hasZero) return n * m; // no water cells; whole grid is already one island

        return best;
    }

    private int LabelIsland(int[,] grid, int[,] labels, int r, int c, int n, int m, int id)
    {
        if (r < 0 || r >= n || c < 0 || c >= m || grid[r, c] == 0 || labels[r, c] != 0) return 0;

        labels[r, c] = id;
        int size = 1;
        size += LabelIsland(grid, labels, r - 1, c, n, m, id);
        size += LabelIsland(grid, labels, r + 1, c, n, m, id);
        size += LabelIsland(grid, labels, r, c - 1, n, m, id);
        size += LabelIsland(grid, labels, r, c + 1, n, m, id);
        return size;
    }
}
```

Time Complexity: `O(n*m)` — Pass 1 labels every land cell exactly once via DFS; Pass 2 examines every cell once and each `0` cell does O(1) work (checking 4 neighbors, deduped via a small HashSet).
Space Complexity: `O(n*m)` for the `labels` array, plus `O(number of islands)` for the size dictionary, both bounded by `O(n*m)`.

**Explanation:**
Dry run island-labeling on:
```
1 1 0
0 0 0
0 1 1
```
Pass 1: Start at `(0,0)`, unlabeled land → label id `2`. DFS spreads to `(0,1)` (also `1`, unlabeled) → labeled `2`. No other land neighbors reachable (row 1 is all water). Island `2` size = `2`. Continue scanning: `(2,1)` unlabeled land → label id `3`, spreads to `(2,2)` → labeled `3`. Island `3` size = `2`.

`labels` grid:
```
2 2 0
0 0 0
0 3 3
```
`islandSize = {2: 2, 3: 2}`. `best` after pass 1 = `2`.

Pass 2: examine each `0` cell.
- `(0,2)`: neighbors are `(0,1)`=id 2 (Left... actually Up invalid, Down `(1,2)`=0, Left `(0,1)`=2, Right invalid). `neighborIds = {2}`. total = `1 + 2 = 3`. best = `3`.
- `(1,0)`: neighbors Up `(0,0)`=2, Down `(2,0)`=0, Left invalid, Right `(1,1)`=0. `neighborIds = {2}`. total = `1 + 2 = 3`. best stays `3`.
- `(1,1)`: neighbors Up `(0,1)`=2, Down `(2,1)`=3, Left `(1,0)`=0, Right `(1,2)`=0. `neighborIds = {2, 3}` — this cell touches **both** islands from two different directions (Up and Down), and since we use a `HashSet<int>`, each unique id contributes its size only once even if it could be reached from multiple neighbor directions. total = `1 + islandSize[2] + islandSize[3] = 1 + 2 + 2 = 5`. best = `5`.
- `(1,2)`: neighbors Up `(0,2)`=0, Down `(2,2)`=3, Left `(1,1)`=0, Right invalid. `neighborIds = {3}`. total = `1 + 2 = 3`. best stays `5`.
- `(2,0)`: neighbors Up `(1,0)`=0, Down invalid, Left invalid, Right `(2,1)`=3. `neighborIds = {3}`. total = `1 + 2 = 3`. best stays `5`.

Final answer: `5` (flipping `(1,1)` merges both islands plus itself: `2 + 2 + 1 = 5`). The critical correctness point is at `(1,1)`: if the code had summed `islandSize` per neighbor direction without deduplication, an id appearing twice from two directions of the *same* island would double count — but here ids `2` and `3` are genuinely distinct islands, so no double-counting risk existed in this particular cell; the `HashSet` guards against the case where, for example, a `0` cell has two neighbors both belonging to island `2` (e.g., Up and Left both point into the same island), which would otherwise wrongly add `islandSize[2]` twice.
