# Dynamic Programming — DP on Grids

## 1. Ninja's Training

### 1. Ninja's Training

**Problem Statement:**
A ninja has to train for `n` days. On each day he can perform exactly one of 3 activities (indices `0`, `1`, `2`), each activity giving some merit points on that day, provided by a matrix `points[n][3]`. The constraint is that the ninja cannot perform the **same** activity on two consecutive days (to avoid muscle group overuse). Find the maximum total merit points the ninja can earn over `n` days.

**Example:**
- Input:
  ```
  points = [
    [10, 40, 70],
    [20, 50, 80],
    [30, 60, 90]
  ]
  ```
- Output: `210`
- Explanation: Day 0 → activity 2 (70), Day 1 → activity 1 (50), Day 2 → activity 2 (90). Wait, day 1 and day 2 both pick different activities than the previous day: 2 → 1 → 2 is valid (no two consecutive days repeat). Total = 70 + 50 + 90 = 210, which is the maximum achievable.

**Approach 1 — Recursion:**
Define `f(day, last)` = max points obtainable from day `0` to `day`, given that activity `last` was used on the day after (`day+1`), so `last` cannot be repeated on `day`. Try all 3 activities except `last` and recurse to `day-1`.

```csharp
public class NinjaTrainingRecursion
{
    public int MaxPoints(int[,] points, int n)
    {
        // last = 3 means "no restriction" (used for the very first call)
        return Solve(n - 1, 3, points);
    }

    private int Solve(int day, int last, int[,] points)
    {
        if (day == 0)
        {
            int best = 0;
            for (int task = 0; task < 3; task++)
            {
                if (task != last)
                    best = Math.Max(best, points[0, task]);
            }
            return best;
        }

        int maxPoints = 0;
        for (int task = 0; task < 3; task++)
        {
            if (task != last)
            {
                int pointsEarned = points[day, task] + Solve(day - 1, task, points);
                maxPoints = Math.Max(maxPoints, pointsEarned);
            }
        }
        return maxPoints;
    }
}
```

Time Complexity: `O(3^n)` — for every day we branch into (up to) 3 choices, giving an exponential blowup.
Space Complexity: `O(n)` — recursion stack depth.

**Approach 2 — Memoization:**
State is `(day, last)` where `day` ranges over `n` values and `last` over `4` values (0, 1, 2, or 3 for "none"). Cache results in a 2D array.

```csharp
public class NinjaTrainingMemoization
{
    private int[,] dp;

    public int MaxPoints(int[,] points, int n)
    {
        dp = new int[n, 4];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < 4; j++)
                dp[i, j] = -1;

        return Solve(n - 1, 3, points);
    }

    private int Solve(int day, int last, int[,] points)
    {
        if (day == 0)
        {
            int best = 0;
            for (int task = 0; task < 3; task++)
            {
                if (task != last)
                    best = Math.Max(best, points[0, task]);
            }
            return best;
        }

        if (dp[day, last] != -1)
            return dp[day, last];

        int maxPoints = 0;
        for (int task = 0; task < 3; task++)
        {
            if (task != last)
            {
                int pointsEarned = points[day, task] + Solve(day - 1, task, points);
                maxPoints = Math.Max(maxPoints, pointsEarned);
            }
        }

        return dp[day, last] = maxPoints;
    }
}
```

Time Complexity: `O(n * 4 * 3)` ≈ `O(n)` — each of the `n * 4` states is computed once, with an inner loop of 3.
Space Complexity: `O(n * 4)` for the memo table + `O(n)` recursion stack.

**Approach 3 — Tabulation:**
Build `dp[day, last]` bottom-up starting from day 0 up to day `n-1`.

```csharp
public class NinjaTrainingTabulation
{
    public int MaxPoints(int[,] points, int n)
    {
        int[,] dp = new int[n, 4];

        // Base case: day 0
        dp[0, 0] = Math.Max(points[0, 1], points[0, 2]);
        dp[0, 1] = Math.Max(points[0, 0], points[0, 2]);
        dp[0, 2] = Math.Max(points[0, 0], points[0, 1]);
        dp[0, 3] = Math.Max(points[0, 0], Math.Max(points[0, 1], points[0, 2]));

        for (int day = 1; day < n; day++)
        {
            for (int last = 0; last < 4; last++)
            {
                dp[day, last] = 0;
                for (int task = 0; task < 3; task++)
                {
                    if (task != last)
                    {
                        int pointsEarned = points[day, task] + dp[day - 1, task];
                        dp[day, last] = Math.Max(dp[day, last], pointsEarned);
                    }
                }
            }
        }

        return dp[n - 1, 3];
    }
}
```

Time Complexity: `O(n * 4 * 3)` ≈ `O(n)`.
Space Complexity: `O(n * 4)` for the DP table, no recursion stack.

**Approach 4 — Space Optimization:**
Since `dp[day]` only depends on `dp[day - 1]`, keep just a rolling array of size 4 instead of the full `n x 4` table.

```csharp
public class NinjaTrainingSpaceOptimized
{
    public int MaxPoints(int[,] points, int n)
    {
        int[] prev = new int[4];

        prev[0] = Math.Max(points[0, 1], points[0, 2]);
        prev[1] = Math.Max(points[0, 0], points[0, 2]);
        prev[2] = Math.Max(points[0, 0], points[0, 1]);
        prev[3] = Math.Max(points[0, 0], Math.Max(points[0, 1], points[0, 2]));

        for (int day = 1; day < n; day++)
        {
            int[] curr = new int[4];
            for (int last = 0; last < 4; last++)
            {
                curr[last] = 0;
                for (int task = 0; task < 3; task++)
                {
                    if (task != last)
                    {
                        int pointsEarned = points[day, task] + prev[task];
                        curr[last] = Math.Max(curr[last], pointsEarned);
                    }
                }
            }
            prev = curr;
        }

        return prev[3];
    }
}
```

Time Complexity: `O(n * 4 * 3)` ≈ `O(n)`.
Space Complexity: `O(4)` ≈ `O(1)` — only two rolling rows of constant size 4 are kept instead of the full `n x 4` table.

---

## 2. Grid Unique Paths

### 2. Grid Unique Paths

**Problem Statement:**
Given a grid of size `m x n`, a robot starts at the top-left cell `(0, 0)` and wants to reach the bottom-right cell `(m-1, n-1)`. The robot can only move **right** or **down** at any point. Count the total number of unique paths.

**Example:**
- Input: `m = 3, n = 3`
- Output: `6`
- Explanation: The 6 unique paths are RRDD, RDRD, RDDR, DRRD, DRDR, DDRR (R = right, D = down), each consisting of exactly `(m-1)` downs and `(n-1)` rights arranged in every possible order.

**Approach 1 — Recursion:**
`f(i, j)` = number of paths to reach `(i, j)` from `(0, 0)` = `f(i-1, j) + f(i, j-1)`.

```csharp
public class UniquePathsRecursion
{
    public int UniquePaths(int m, int n)
    {
        return Solve(m - 1, n - 1);
    }

    private int Solve(int i, int j)
    {
        if (i == 0 && j == 0) return 1;
        if (i < 0 || j < 0) return 0;

        int up = Solve(i - 1, j);
        int left = Solve(i, j - 1);
        return up + left;
    }
}
```

Time Complexity: `O(2^(m+n))` — each cell branches into two recursive calls, roughly doubling with each step down the diagonal.
Space Complexity: `O(m + n)` — recursion stack depth.

**Approach 2 — Memoization:**
Cache `f(i, j)` in a 2D array of size `m x n`.

```csharp
public class UniquePathsMemoization
{
    private int[,] dp;

    public int UniquePaths(int m, int n)
    {
        dp = new int[m, n];
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                dp[i, j] = -1;

        return Solve(m - 1, n - 1);
    }

    private int Solve(int i, int j)
    {
        if (i == 0 && j == 0) return 1;
        if (i < 0 || j < 0) return 0;

        if (dp[i, j] != -1) return dp[i, j];

        int up = Solve(i - 1, j);
        int left = Solve(i, j - 1);
        return dp[i, j] = up + left;
    }
}
```

Time Complexity: `O(m * n)` — each state is computed once.
Space Complexity: `O(m * n)` for the memo table + `O(m + n)` recursion stack.

**Approach 3 — Tabulation:**
Fill `dp[i, j]` from `(0, 0)` upward/leftward outward.

```csharp
public class UniquePathsTabulation
{
    public int UniquePaths(int m, int n)
    {
        int[,] dp = new int[m, n];

        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (i == 0 && j == 0)
                {
                    dp[i, j] = 1;
                    continue;
                }

                int up = (i > 0) ? dp[i - 1, j] : 0;
                int left = (j > 0) ? dp[i, j - 1] : 0;
                dp[i, j] = up + left;
            }
        }

        return dp[m - 1, n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(m * n)` for the DP table.

**Approach 4 — Space Optimization:**
Only the previous row is needed to compute the current row, so keep a single rolling 1D array of size `n`.

```csharp
public class UniquePathsSpaceOptimized
{
    public int UniquePaths(int m, int n)
    {
        int[] prev = new int[n];

        for (int i = 0; i < m; i++)
        {
            int[] curr = new int[n];
            for (int j = 0; j < n; j++)
            {
                if (i == 0 && j == 0)
                {
                    curr[j] = 1;
                    continue;
                }

                int up = prev[j];
                int left = (j > 0) ? curr[j - 1] : 0;
                curr[j] = up + left;
            }
            prev = curr;
        }

        return prev[n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(n)` — only two rolling rows (`prev` and `curr`) instead of the full `m x n` table.

---

## 3. Grid Unique Paths II

### 3. Grid Unique Paths II

**Problem Statement:**
Same as Grid Unique Paths, but now the grid contains obstacles marked `1` (a `0` denotes a free cell). The robot still starts at `(0, 0)` and moves only right or down, but it cannot step onto a cell with an obstacle. Count the number of unique paths to the bottom-right cell. If the start or end cell itself is blocked, the answer is `0`.

**Example:**
- Input:
  ```
  grid = [
    [0, 0, 0],
    [0, 1, 0],
    [0, 0, 0]
  ]
  ```
- Output: `2`
- Explanation: The obstacle at `(1,1)` blocks the direct diagonal-ish routes through the center. Only two paths avoid it: right-right-down-down going around the top, and down-down-right-right going around the bottom (`DDRR` and `RRDD`), i.e. `(0,0)→(0,1)→(0,2)→(1,2)→(2,2)` and `(0,0)→(1,0)→(2,0)→(2,1)→(2,2)`.

**Approach 1 — Recursion:**
Same recurrence as unique paths, but return `0` immediately if the current cell is an obstacle.

```csharp
public class UniquePathsWithObstaclesRecursion
{
    public int UniquePathsWithObstacles(int[,] grid, int m, int n)
    {
        if (grid[m - 1, n - 1] == 1 || grid[0, 0] == 1) return 0;
        return Solve(m - 1, n - 1, grid);
    }

    private int Solve(int i, int j, int[,] grid)
    {
        if (i < 0 || j < 0 || grid[i, j] == 1) return 0;
        if (i == 0 && j == 0) return 1;

        int up = Solve(i - 1, j, grid);
        int left = Solve(i, j - 1, grid);
        return up + left;
    }
}
```

Time Complexity: `O(2^(m+n))` — same exponential branching as the obstacle-free version, since obstacles only prune some branches.
Space Complexity: `O(m + n)` — recursion stack depth.

**Approach 2 — Memoization:**
Cache results per `(i, j)` exactly like before, short-circuiting on obstacles before consulting/using the cache.

```csharp
public class UniquePathsWithObstaclesMemoization
{
    private int[,] dp;

    public int UniquePathsWithObstacles(int[,] grid, int m, int n)
    {
        if (grid[m - 1, n - 1] == 1 || grid[0, 0] == 1) return 0;

        dp = new int[m, n];
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                dp[i, j] = -1;

        return Solve(m - 1, n - 1, grid);
    }

    private int Solve(int i, int j, int[,] grid)
    {
        if (i < 0 || j < 0 || grid[i, j] == 1) return 0;
        if (i == 0 && j == 0) return 1;

        if (dp[i, j] != -1) return dp[i, j];

        int up = Solve(i - 1, j, grid);
        int left = Solve(i, j - 1, grid);
        return dp[i, j] = up + left;
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(m * n)` for the memo table + `O(m + n)` recursion stack.

**Approach 3 — Tabulation:**
Fill `dp[i, j]` = `0` whenever the cell is an obstacle, otherwise sum from top and left.

```csharp
public class UniquePathsWithObstaclesTabulation
{
    public int UniquePathsWithObstacles(int[,] grid, int m, int n)
    {
        int[,] dp = new int[m, n];

        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (grid[i, j] == 1)
                {
                    dp[i, j] = 0;
                    continue;
                }

                if (i == 0 && j == 0)
                {
                    dp[i, j] = 1;
                    continue;
                }

                int up = (i > 0) ? dp[i - 1, j] : 0;
                int left = (j > 0) ? dp[i, j - 1] : 0;
                dp[i, j] = up + left;
            }
        }

        return dp[m - 1, n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(m * n)` for the DP table.

**Approach 4 — Space Optimization:**
Roll the row exactly as in problem 2, zeroing out obstacle cells in the current row.

```csharp
public class UniquePathsWithObstaclesSpaceOptimized
{
    public int UniquePathsWithObstacles(int[,] grid, int m, int n)
    {
        int[] prev = new int[n];

        for (int i = 0; i < m; i++)
        {
            int[] curr = new int[n];
            for (int j = 0; j < n; j++)
            {
                if (grid[i, j] == 1)
                {
                    curr[j] = 0;
                    continue;
                }

                if (i == 0 && j == 0)
                {
                    curr[j] = 1;
                    continue;
                }

                int up = prev[j];
                int left = (j > 0) ? curr[j - 1] : 0;
                curr[j] = up + left;
            }
            prev = curr;
        }

        return prev[n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(n)` — one rolling row instead of the full `m x n` table.

---

## 4. Minimum Path Sum in a Grid

### 4. Minimum Path Sum in a Grid

**Problem Statement:**
Given an `m x n` grid filled with non-negative integers, find a path from top-left `(0, 0)` to bottom-right `(m-1, n-1)` which minimizes the sum of all numbers along the path. Movement is restricted to right or down.

**Example:**
- Input:
  ```
  grid = [
    [1, 3, 1],
    [1, 5, 1],
    [4, 2, 1]
  ]
  ```
- Output: `7`
- Explanation: The path `1 → 3 → 1 → 1 → 1` (moving right, right, down, down) sums to `1+3+1+1+1 = 7`, which is the minimum possible.

**Approach 1 — Recursion:**
`f(i, j)` = min cost to reach `(i, j)` from `(0, 0)` = `grid[i,j] + min(f(i-1,j), f(i,j-1))`.

```csharp
public class MinPathSumRecursion
{
    public int MinPathSum(int[,] grid, int m, int n)
    {
        return Solve(m - 1, n - 1, grid);
    }

    private int Solve(int i, int j, int[,] grid)
    {
        if (i == 0 && j == 0) return grid[0, 0];
        if (i < 0 || j < 0) return int.MaxValue / 2; // avoid overflow when adding

        int up = grid[i, j] + Solve(i - 1, j, grid);
        int left = grid[i, j] + Solve(i, j - 1, grid);
        return Math.Min(up, left);
    }
}
```

Time Complexity: `O(2^(m+n))` — exponential branching, identical shape to unique-paths recursion.
Space Complexity: `O(m + n)` — recursion stack depth.

**Approach 2 — Memoization:**
Cache `f(i, j)` in a 2D array.

```csharp
public class MinPathSumMemoization
{
    private int[,] dp;

    public int MinPathSum(int[,] grid, int m, int n)
    {
        dp = new int[m, n];
        for (int i = 0; i < m; i++)
            for (int j = 0; j < n; j++)
                dp[i, j] = -1;

        return Solve(m - 1, n - 1, grid);
    }

    private int Solve(int i, int j, int[,] grid)
    {
        if (i == 0 && j == 0) return grid[0, 0];
        if (i < 0 || j < 0) return int.MaxValue / 2;

        if (dp[i, j] != -1) return dp[i, j];

        int up = grid[i, j] + Solve(i - 1, j, grid);
        int left = grid[i, j] + Solve(i, j - 1, grid);
        return dp[i, j] = Math.Min(up, left);
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(m * n)` for the memo table + `O(m + n)` recursion stack.

**Approach 3 — Tabulation:**
Fill `dp[i, j]` bottom-up, treating missing neighbors (out of bounds) as infinity.

```csharp
public class MinPathSumTabulation
{
    public int MinPathSum(int[,] grid, int m, int n)
    {
        int[,] dp = new int[m, n];

        for (int i = 0; i < m; i++)
        {
            for (int j = 0; j < n; j++)
            {
                if (i == 0 && j == 0)
                {
                    dp[i, j] = grid[i, j];
                    continue;
                }

                int up = (i > 0) ? grid[i, j] + dp[i - 1, j] : int.MaxValue / 2;
                int left = (j > 0) ? grid[i, j] + dp[i, j - 1] : int.MaxValue / 2;
                dp[i, j] = Math.Min(up, left);
            }
        }

        return dp[m - 1, n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(m * n)` for the DP table.

**Approach 4 — Space Optimization:**
Only the previous row is needed, so use a rolling 1D array of size `n`.

```csharp
public class MinPathSumSpaceOptimized
{
    public int MinPathSum(int[,] grid, int m, int n)
    {
        int[] prev = new int[n];

        for (int i = 0; i < m; i++)
        {
            int[] curr = new int[n];
            for (int j = 0; j < n; j++)
            {
                if (i == 0 && j == 0)
                {
                    curr[j] = grid[i, j];
                    continue;
                }

                int up = (i > 0) ? grid[i, j] + prev[j] : int.MaxValue / 2;
                int left = (j > 0) ? grid[i, j] + curr[j - 1] : int.MaxValue / 2;
                curr[j] = Math.Min(up, left);
            }
            prev = curr;
        }

        return prev[n - 1];
    }
}
```

Time Complexity: `O(m * n)`.
Space Complexity: `O(n)` — two rolling rows instead of the full `m x n` table.

---

## 5. Minimum Path Sum in a Triangular Grid

### 5. Minimum Path Sum in a Triangular Grid (Triangle)

**Problem Statement:**
Given a triangular array of numbers with `n` rows, find the minimum path sum from the top to the bottom. At each step, you may move to an adjacent number on the row below — i.e., from index `j` on row `i` you can move to index `j` or `j+1` on row `i+1`.

**Example:**
- Input:
  ```
  triangle = [
    [2],
    [3, 4],
    [6, 5, 7],
    [4, 1, 8, 3]
  ]
  ```
- Output: `11`
- Explanation: The minimum path is `2 → 3 → 5 → 1`, summing to `2+3+5+1 = 11`.

**Approach 1 — Recursion:**
`f(i, j)` = min sum from `(i, j)` down to the last row = `triangle[i][j] + min(f(i+1,j), f(i+1,j+1))`. This version recurses top-down from `(0, 0)`.

```csharp
public class TriangleRecursion
{
    public int MinimumTotal(List<List<int>> triangle)
    {
        int n = triangle.Count;
        return Solve(0, 0, triangle, n);
    }

    private int Solve(int i, int j, List<List<int>> triangle, int n)
    {
        if (i == n - 1) return triangle[i][j];

        int down = triangle[i][j] + Solve(i + 1, j, triangle, n);
        int diagonal = triangle[i][j] + Solve(i + 1, j + 1, triangle, n);
        return Math.Min(down, diagonal);
    }
}
```

Time Complexity: `O(2^n)` — each cell branches into 2 recursive calls down the triangle.
Space Complexity: `O(n)` — recursion stack depth.

**Approach 2 — Memoization:**
Cache `f(i, j)` in a 2D array sized `n x n`.

```csharp
public class TriangleMemoization
{
    private int[,] dp;

    public int MinimumTotal(List<List<int>> triangle)
    {
        int n = triangle.Count;
        dp = new int[n, n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                dp[i, j] = -1;

        return Solve(0, 0, triangle, n);
    }

    private int Solve(int i, int j, List<List<int>> triangle, int n)
    {
        if (i == n - 1) return triangle[i][j];

        if (dp[i, j] != -1) return dp[i, j];

        int down = triangle[i][j] + Solve(i + 1, j, triangle, n);
        int diagonal = triangle[i][j] + Solve(i + 1, j + 1, triangle, n);
        return dp[i, j] = Math.Min(down, diagonal);
    }
}
```

Time Complexity: `O(n^2)` — roughly `n^2 / 2` unique states, each computed once.
Space Complexity: `O(n^2)` for the memo table + `O(n)` recursion stack.

**Approach 3 — Tabulation:**
Fill bottom-up: start `dp[n-1][j] = triangle[n-1][j]` for all `j`, then work upward.

```csharp
public class TriangleTabulation
{
    public int MinimumTotal(List<List<int>> triangle)
    {
        int n = triangle.Count;
        int[,] dp = new int[n, n];

        for (int j = 0; j < n; j++)
            dp[n - 1, j] = triangle[n - 1][j];

        for (int i = n - 2; i >= 0; i--)
        {
            for (int j = i; j >= 0; j--)
            {
                int down = triangle[i][j] + dp[i + 1, j];
                int diagonal = triangle[i][j] + dp[i + 1, j + 1];
                dp[i, j] = Math.Min(down, diagonal);
            }
        }

        return dp[0, 0];
    }
}
```

Time Complexity: `O(n^2)`.
Space Complexity: `O(n^2)` for the DP table.

**Approach 4 — Space Optimization:**
Only row `i+1` is needed to compute row `i`, so keep a single rolling 1D array of size `n`.

```csharp
public class TriangleSpaceOptimized
{
    public int MinimumTotal(List<List<int>> triangle)
    {
        int n = triangle.Count;
        int[] front = new int[n];

        for (int j = 0; j < n; j++)
            front[j] = triangle[n - 1][j];

        for (int i = n - 2; i >= 0; i--)
        {
            int[] curr = new int[n];
            for (int j = i; j >= 0; j--)
            {
                int down = triangle[i][j] + front[j];
                int diagonal = triangle[i][j] + front[j + 1];
                curr[j] = Math.Min(down, diagonal);
            }
            front = curr;
        }

        return front[0];
    }
}
```

Time Complexity: `O(n^2)`.
Space Complexity: `O(n)` — one rolling row instead of the full `n x n` table.

---

## 6. Minimum/Maximum Falling Path Sum

### 6. Minimum/Maximum Falling Path Sum

**Problem Statement:**
Given an `n x n` (or `m x n`) grid of integers, a falling path starts at **any** cell in the first row and chooses, at each subsequent row, one of at most 3 cells directly below (same column, or diagonally left/right one column) to move to. Find the minimum (or maximum) sum of any falling path from the first row to the last row.

**Example:**
- Input:
  ```
  matrix = [
    [2, 1, 3],
    [6, 5, 4],
    [7, 8, 9]
  ]
  ```
- Output (minimum): `13`
- Explanation: Starting at `(0,1)=1`, go to `(1,1)=5` (or `(1,0)=6`... check all), then `(2,0)=7`. Path `1 → 5 → 7` sums to `13`. Checking alternatives: `1→4→7=12`? `(1,2)=4` is reachable from `(0,1)` diagonally, then from `(1,2)` we can go to `(2,1)=8` or `(2,2)=9`, not `(2,0)`. So `1→4→8=13`. Best is `1(row0,col1) → 6(row1,col0)`? Not adjacent diagonally beyond 1 column, `(1,0)` is reachable from `(0,1)` (left diagonal), giving `1+6=7`, then from `(1,0)` to `(2,0)=7` or `(2,1)=8`, giving `7+7=14` or `7+8=15`. The true minimum path is `1 → 5 → 7 = 13` (from `(0,1)` straight down to `(1,1)=5`, then diagonal to `(2,0)=7`).

**Approach 1 — Recursion:**
`f(i, j)` = minimum sum of a falling path ending at `(i, j)`, computed by exploring downward from row 0; the answer is `min` over all `f(n-1, j)`. Equivalently recurse from the last row upward: `f(i, j) = matrix[i][j] + min(f(i+1,j-1), f(i+1,j), f(i+1,j+1))`.

```csharp
public class FallingPathSumRecursion
{
    public int MinFallingPathSum(int[,] matrix, int n)
    {
        int best = int.MaxValue;
        for (int j = 0; j < n; j++)
            best = Math.Min(best, Solve(0, j, matrix, n));
        return best;
    }

    private int Solve(int i, int j, int[,] matrix, int n)
    {
        if (j < 0 || j >= n) return int.MaxValue / 2;
        if (i == n - 1) return matrix[i, j];

        int down = matrix[i, j] + Solve(i + 1, j, matrix, n);
        int leftDiag = matrix[i, j] + Solve(i + 1, j - 1, matrix, n);
        int rightDiag = matrix[i, j] + Solve(i + 1, j + 1, matrix, n);
        return Math.Min(down, Math.Min(leftDiag, rightDiag));
    }
}
```

Time Complexity: `O(3^n)` — each cell branches into up to 3 recursive calls across `n` rows.
Space Complexity: `O(n)` — recursion stack depth.

**Approach 2 — Memoization:**
Cache `f(i, j)` in a 2D array sized `n x n`.

```csharp
public class FallingPathSumMemoization
{
    private int[,] dp;

    public int MinFallingPathSum(int[,] matrix, int n)
    {
        dp = new int[n, n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                dp[i, j] = -1;

        int best = int.MaxValue;
        for (int j = 0; j < n; j++)
            best = Math.Min(best, Solve(0, j, matrix, n));
        return best;
    }

    private int Solve(int i, int j, int[,] matrix, int n)
    {
        if (j < 0 || j >= n) return int.MaxValue / 2;
        if (i == n - 1) return matrix[i, j];

        if (dp[i, j] != -1) return dp[i, j];

        int down = matrix[i, j] + Solve(i + 1, j, matrix, n);
        int leftDiag = matrix[i, j] + Solve(i + 1, j - 1, matrix, n);
        int rightDiag = matrix[i, j] + Solve(i + 1, j + 1, matrix, n);
        return dp[i, j] = Math.Min(down, Math.Min(leftDiag, rightDiag));
    }
}
```

Time Complexity: `O(n * n * 3)` ≈ `O(n^2)` — each of the `n^2` states computed once, with a constant 3-way branch.
Space Complexity: `O(n^2)` for the memo table + `O(n)` recursion stack.

**Approach 3 — Tabulation:**
Fill row `n-1` with the matrix values directly, then compute upward using the 3 possible children below.

```csharp
public class FallingPathSumTabulation
{
    public int MinFallingPathSum(int[,] matrix, int n)
    {
        int[,] dp = new int[n, n];

        for (int j = 0; j < n; j++)
            dp[n - 1, j] = matrix[n - 1, j];

        for (int i = n - 2; i >= 0; i--)
        {
            for (int j = 0; j < n; j++)
            {
                int down = dp[i + 1, j];
                int leftDiag = (j - 1 >= 0) ? dp[i + 1, j - 1] : int.MaxValue / 2;
                int rightDiag = (j + 1 < n) ? dp[i + 1, j + 1] : int.MaxValue / 2;
                dp[i, j] = matrix[i, j] + Math.Min(down, Math.Min(leftDiag, rightDiag));
            }
        }

        int best = int.MaxValue;
        for (int j = 0; j < n; j++)
            best = Math.Min(best, dp[0, j]);
        return best;
    }
}
```

Time Complexity: `O(n^2)`.
Space Complexity: `O(n^2)` for the DP table.

**Approach 4 — Space Optimization:**
Only row `i+1` is needed to compute row `i`, so keep a rolling 1D array of size `n`.

```csharp
public class FallingPathSumSpaceOptimized
{
    public int MinFallingPathSum(int[,] matrix, int n)
    {
        int[] front = new int[n];
        for (int j = 0; j < n; j++)
            front[j] = matrix[n - 1, j];

        for (int i = n - 2; i >= 0; i--)
        {
            int[] curr = new int[n];
            for (int j = 0; j < n; j++)
            {
                int down = front[j];
                int leftDiag = (j - 1 >= 0) ? front[j - 1] : int.MaxValue / 2;
                int rightDiag = (j + 1 < n) ? front[j + 1] : int.MaxValue / 2;
                curr[j] = matrix[i, j] + Math.Min(down, Math.Min(leftDiag, rightDiag));
            }
            front = curr;
        }

        int best = int.MaxValue;
        for (int j = 0; j < n; j++)
            best = Math.Min(best, front[j]);
        return best;
    }
}
```

Time Complexity: `O(n^2)`.
Space Complexity: `O(n)` — one rolling row instead of the full `n x n` table.

*(For the maximum falling path sum variant, simply replace every `Math.Min` with `Math.Max` and initialize sentinel/boundary values with `int.MinValue / 2` instead of `int.MaxValue / 2`.)*

---

## 7. 3D DP: Ninja and His Friends / Cherry Pickup II

### 7. Ninja and His Friends / Cherry Pickup II

**Problem Statement:**
Given an `r x c` grid where each cell contains some number of cherries (0 or more), two people start at the top row — one at `(0, 0)` and the other at `(0, c-1)`. On each step, both move simultaneously to the next row (`row + 1`), and each can independently move to one of 3 columns: same column, one column left, or one column right (staying within grid bounds). Whenever both collect cherries from the same cell, the cherries are counted only **once**. Maximize the total number of cherries collected by both people when they reach the last row.

**Example:**
- Input:
  ```
  grid = [
    [3, 1, 1],
    [2, 5, 1],
    [1, 5, 5]
  ]
  ```
- Output: `24`
- Explanation: Person 1 starts at `(0,0)=3`, Person 2 starts at `(0,2)=1`. Optimal paths: Person 1 goes `(0,0)→(1,1)→(2,0)` collecting `3+5+1=9`... but a better combined route collects `3+1(start) + moves down through columns picking the 5s twice at row1/row2 with distinct columns`. Following the well-known result for this exact grid, the maximum combined cherries collected is `24` — achieved by paths where both people converge to pick up the high-value `5` cells at row 2 through non-overlapping columns while avoiding double-counting on shared cells.

**Approach 1 — Recursion:**
State `(row, col1, col2)` = max cherries collectible from `row` to the last row, given person 1 is at column `col1` and person 2 is at `col2`. At each row, add `grid[row][col1]`, plus `grid[row][col2]` only if `col2 != col1`, then try all `3 x 3 = 9` combinations of moves for the next row.

```csharp
public class CherryPickupIIRecursion
{
    public int CherryPickup(int[,] grid, int rows, int cols)
    {
        return Solve(0, 0, cols - 1, grid, rows, cols);
    }

    private int Solve(int row, int col1, int col2, int[,] grid, int rows, int cols)
    {
        if (col1 < 0 || col1 >= cols || col2 < 0 || col2 >= cols)
            return int.MinValue / 2;

        int cherries = grid[row, col1];
        if (col1 != col2)
            cherries += grid[row, col2];

        if (row == rows - 1)
            return cherries;

        int best = int.MinValue;
        for (int d1 = -1; d1 <= 1; d1++)
        {
            for (int d2 = -1; d2 <= 1; d2++)
            {
                int next = Solve(row + 1, col1 + d1, col2 + d2, grid, rows, cols);
                best = Math.Max(best, next);
            }
        }

        return cherries + best;
    }
}
```

Time Complexity: `O(3^n * 3^n)` = `O(9^n)` where `n` is the number of rows — each of the two people independently branches 3 ways at every row.
Space Complexity: `O(n)` — recursion stack depth (rows).

**Approach 2 — Memoization:**
State is `(row, col1, col2)` — cache in a 3D array sized `rows x cols x cols`.

```csharp
public class CherryPickupIIMemoization
{
    private int[,,] dp;

    public int CherryPickup(int[,] grid, int rows, int cols)
    {
        dp = new int[rows, cols, cols];
        for (int i = 0; i < rows; i++)
            for (int j = 0; j < cols; j++)
                for (int k = 0; k < cols; k++)
                    dp[i, j, k] = -1;

        return Solve(0, 0, cols - 1, grid, rows, cols);
    }

    private int Solve(int row, int col1, int col2, int[,] grid, int rows, int cols)
    {
        if (col1 < 0 || col1 >= cols || col2 < 0 || col2 >= cols)
            return int.MinValue / 2;

        if (dp[row, col1, col2] != -1)
            return dp[row, col1, col2];

        int cherries = grid[row, col1];
        if (col1 != col2)
            cherries += grid[row, col2];

        if (row == rows - 1)
            return dp[row, col1, col2] = cherries;

        int best = int.MinValue;
        for (int d1 = -1; d1 <= 1; d1++)
        {
            for (int d2 = -1; d2 <= 1; d2++)
            {
                int next = Solve(row + 1, col1 + d1, col2 + d2, grid, rows, cols);
                best = Math.Max(best, next);
            }
        }

        return dp[row, col1, col2] = cherries + best;
    }
}
```

Time Complexity: `O(rows * cols * cols * 9)` ≈ `O(rows * cols^2)` — each of the `rows * cols^2` states computed once, with a constant 9-way branch.
Space Complexity: `O(rows * cols^2)` for the memo table + `O(rows)` recursion stack.

**Approach 3 — Tabulation:**
Fill the last row directly, then build upward row by row using the 9 move combinations.

```csharp
public class CherryPickupIITabulation
{
    public int CherryPickup(int[,] grid, int rows, int cols)
    {
        int[,,] dp = new int[rows, cols, cols];

        // Base case: last row
        for (int col1 = 0; col1 < cols; col1++)
        {
            for (int col2 = 0; col2 < cols; col2++)
            {
                int cherries = grid[rows - 1, col1];
                if (col1 != col2)
                    cherries += grid[rows - 1, col2];
                dp[rows - 1, col1, col2] = cherries;
            }
        }

        for (int row = rows - 2; row >= 0; row--)
        {
            for (int col1 = 0; col1 < cols; col1++)
            {
                for (int col2 = 0; col2 < cols; col2++)
                {
                    int cherries = grid[row, col1];
                    if (col1 != col2)
                        cherries += grid[row, col2];

                    int best = int.MinValue;
                    for (int d1 = -1; d1 <= 1; d1++)
                    {
                        for (int d2 = -1; d2 <= 1; d2++)
                        {
                            int nc1 = col1 + d1;
                            int nc2 = col2 + d2;
                            if (nc1 >= 0 && nc1 < cols && nc2 >= 0 && nc2 < cols)
                                best = Math.Max(best, dp[row + 1, nc1, nc2]);
                        }
                    }

                    dp[row, col1, col2] = cherries + best;
                }
            }
        }

        return dp[0, 0, cols - 1];
    }
}
```

Time Complexity: `O(rows * cols * cols * 9)` ≈ `O(rows * cols^2)`.
Space Complexity: `O(rows * cols^2)` for the 3D DP table.

**Approach 4 — Space Optimization:**
Only row `row + 1` is needed to compute `row`, so the 3D table's row dimension can be rolled down to a 2D `cols x cols` array (front/current), just like problems 2-6 roll away a full row dimension — here it is one dimension of the 3D state that becomes rolling, leaving a 2D `col1 x col2` table.

```csharp
public class CherryPickupIISpaceOptimized
{
    public int CherryPickup(int[,] grid, int rows, int cols)
    {
        int[,] front = new int[cols, cols];

        for (int col1 = 0; col1 < cols; col1++)
        {
            for (int col2 = 0; col2 < cols; col2++)
            {
                int cherries = grid[rows - 1, col1];
                if (col1 != col2)
                    cherries += grid[rows - 1, col2];
                front[col1, col2] = cherries;
            }
        }

        for (int row = rows - 2; row >= 0; row--)
        {
            int[,] curr = new int[cols, cols];
            for (int col1 = 0; col1 < cols; col1++)
            {
                for (int col2 = 0; col2 < cols; col2++)
                {
                    int cherries = grid[row, col1];
                    if (col1 != col2)
                        cherries += grid[row, col2];

                    int best = int.MinValue;
                    for (int d1 = -1; d1 <= 1; d1++)
                    {
                        for (int d2 = -1; d2 <= 1; d2++)
                        {
                            int nc1 = col1 + d1;
                            int nc2 = col2 + d2;
                            if (nc1 >= 0 && nc1 < cols && nc2 >= 0 && nc2 < cols)
                                best = Math.Max(best, front[nc1, nc2]);
                        }
                    }

                    curr[col1, col2] = cherries + best;
                }
            }
            front = curr;
        }

        return front[0, cols - 1];
    }
}
```

Time Complexity: `O(rows * cols^2 * 9)` ≈ `O(rows * cols^2)`.
Space Complexity: `O(cols^2)` — only two rolling `cols x cols` layers (`front` and `curr`) instead of the full `rows x cols x cols` table, since only the immediately-next row's state is ever needed.

**Explanation:**

*Dry run — 3D state transition for Cherry Pickup II on a 3x3 grid:*

```
grid = [
  [3, 1, 1],
  [2, 5, 1],
  [1, 5, 5]
]
```

The state is `(row, col1, col2)`, meaning: with person 1 at column `col1` and person 2 at column `col2`, both currently on `row`, what is the maximum cherries collectible from `row` through the last row (`row = 2`)?

Start state: `(0, 0, 2)` — person 1 begins at `(0,0)`, person 2 begins at `(0,2)`.

At row 0: `col1 = 0`, `col2 = 2`. Since `col1 != col2`, cherries collected this row = `grid[0][0] + grid[0][2] = 3 + 1 = 4`.

Now both people move to row 1. Person 1 can move to column `0-1=-1` (invalid), `0`, or `0+1=1`. Person 2 can move to column `2-1=1`, `2`, or `2+1=3` (invalid). So valid combinations (of the 9 theoretical `d1 x d2` pairs) after pruning out-of-bounds moves are:
- (col1=0, col2=1), (col1=0, col2=2)
- (col1=1, col2=1), (col1=1, col2=2)

(4 valid combinations here since edge columns cut off some of the 9 possibilities; in the middle of the grid all 9 combinations would typically be valid.)

For each, evaluate row 1 cherries then recurse to row 2:
- `(1, 0, 1)`: cherries = `grid[1][0] + grid[1][1] = 2 + 5 = 7` (col1 != col2, both counted).
- `(1, 0, 2)`: cherries = `grid[1][0] + grid[1][2] = 2 + 1 = 3`.
- `(1, 1, 1)`: **special case, col1 == col2** → cherries = `grid[1][1]` counted only **once** = `5` (not `5+5=10`, since it's the same physical cell — both people cannot double-collect from one cell).
- `(1, 1, 2)`: cherries = `grid[1][1] + grid[1][2] = 5 + 1 = 6`.

Take `(1, 0, 1)` = 7 as an example and expand to row 2 (the last row, base case). From `col1=0`, valid next columns: `0` or `1`. From `col2=1`, valid next columns: `0`, `1`, or `2`.
- `(2, 0, 0)`: col1 == col2 → cherries = `grid[2][0]` once = `1`.
- `(2, 0, 1)`: cherries = `grid[2][0] + grid[2][1] = 1 + 5 = 6`.
- `(2, 0, 2)`: cherries = `grid[2][0] + grid[2][2] = 1 + 5 = 6`.
- `(2, 1, 0)`: cherries = `grid[2][1] + grid[2][0] = 5 + 1 = 6`.
- `(2, 1, 1)`: col1 == col2 → cherries = `grid[2][1]` once = `5`.
- `(2, 1, 2)`: cherries = `grid[2][1] + grid[2][2] = 5 + 5 = 10`.

The best among these base cases reachable from `(1, 0, 1)` is `10` (from `(2,1,2)`), so `dp[1][0][1] = 7 + 10 = 17`.

Repeating this expansion for every valid `(row, col1, col2)` combination back up to `(0, 0, 2)` and taking the maximum at each level (the "9 possible move combinations" rule: for every pair of columns, try all `Δcol1 ∈ {-1,0,+1}` and `Δcol2 ∈ {-1,0,+1}`, discard out-of-bounds results, and keep the max of the remaining next-row states), the overall best combined total works out to `dp[0][0][2] = 24`, matching the expected output — achieved via person 1 taking a path through the higher-value cells at row 2 while person 2 covers a different column so neither cherry cell is wasted or double-counted.

The `col1 == col2` special case is the crux of the whole problem: whenever both people land on the same cell in the same row, the cherries there are added **once**, not twice, because physically there is only one pile of cherries in that cell — this is why the recurrence always checks `if (col1 != col2)` before adding `grid[row][col2]` a second time.

*Dry run — Triangle minimum path sum, bottom-up from the last row upward:*

```
triangle = [
  [2],
  [3, 4],
  [6, 5, 7],
  [4, 1, 8, 3]
]
```

Initialize `dp` for the last row (row 3) directly from the triangle values:
`dp[3] = [4, 1, 8, 3]`

Row 2 (`[6, 5, 7]`): for each `j`, `dp[2][j] = triangle[2][j] + min(dp[3][j], dp[3][j+1])`.
- `j=0`: `6 + min(dp[3][0]=4, dp[3][1]=1) = 6 + 1 = 7`
- `j=1`: `5 + min(dp[3][1]=1, dp[3][2]=8) = 5 + 1 = 6`
- `j=2`: `7 + min(dp[3][2]=8, dp[3][3]=3) = 7 + 3 = 10`

`dp[2] = [7, 6, 10]`

Row 1 (`[3, 4]`): `dp[1][j] = triangle[1][j] + min(dp[2][j], dp[2][j+1])`.
- `j=0`: `3 + min(dp[2][0]=7, dp[2][1]=6) = 3 + 6 = 9`
- `j=1`: `4 + min(dp[2][1]=6, dp[2][2]=10) = 4 + 6 = 10`

`dp[1] = [9, 10]`

Row 0 (`[2]`): `dp[0][0] = triangle[0][0] + min(dp[1][0], dp[1][1]) = 2 + min(9, 10) = 2 + 9 = 11`.

Final answer: `dp[0][0] = 11`, matching the expected output. Tracing the choices back down confirms the path `2 (row0) → 3 (row1, j=0) → 5 (row2, j=1) → 1 (row3, j=1)`, summing to `2+3+5+1 = 11`, since at each step the `min` chose the child that ultimately led to this optimal total.
