# Dynamic Programming — DP on Squares

## 1. Maximal Square

### 1. Maximal Square

**Problem Statement:**
Given a binary matrix (`rows x cols`) filled with `0`s and `1`s, find the largest square submatrix that contains only `1`s and return its **area** (side length squared).

**Example:**
- Input:
```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```
- Output: `4`
- Explanation: The largest all-ones square has side length `2` (e.g. the block formed by rows 1-2, columns 2-3, or rows 1-2, columns 3-4). Side length `2` gives area `2 * 2 = 4`. No all-ones square of side `3` exists in this matrix.

**Brute Force Approach:**
For every cell, treat it as the top-left corner of a candidate square and try to expand the side length as far as possible, checking every cell inside the square each time.

```csharp
public class MaximalSquareBruteForce
{
    public int MaximalSquare(int[][] matrix)
    {
        if (matrix == null || matrix.Length == 0 || matrix[0].Length == 0)
            return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int maxSide = 0;

        // Try every cell as the top-left corner of a square.
        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                if (matrix[i][j] != 1) continue;

                // Try to grow the square side by side.
                int maxPossibleSide = Math.Min(rows - i, cols - j);
                int side = 1;

                for (; side <= maxPossibleSide; side++)
                {
                    if (!IsAllOnesSquare(matrix, i, j, side))
                        break;
                }

                // 'side' overshot by one on break (or equals maxPossibleSide + 1 when loop completed).
                maxSide = Math.Max(maxSide, side - 1);
            }
        }

        return maxSide * maxSide;
    }

    // Checks every cell of the square with top-left (r, c) and given side length.
    private bool IsAllOnesSquare(int[][] matrix, int r, int c, int side)
    {
        for (int x = r; x < r + side; x++)
        {
            for (int y = c; y < c + side; y++)
            {
                if (matrix[x][y] != 1) return false;
            }
        }
        return true;
    }
}
```

Time Complexity: `O(rows * cols * min(rows, cols)^2)` — for each cell we may check squares of growing size, each check costing up to `O(side^2)`.
Space Complexity: `O(1)` extra space (no auxiliary DP table used).

**Optimized DP Approach:**
Define `dp[i][j]` = side length of the largest all-ones square whose **bottom-right** corner is at cell `(i, j)`.

Recurrence:
- If `matrix[i][j] == 0`, then `dp[i][j] = 0` (no square can end here).
- If `matrix[i][j] == 1`:
  - If `i == 0` or `j == 0` (first row/column), `dp[i][j] = 1` (a lone cell is a valid `1x1` square, and there is no room to extend a bigger square since a neighboring row/column is missing).
  - Otherwise, `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`.

For the **Maximal Square** problem, the final answer is `(max value in dp table)^2`, since the max `dp[i][j]` directly gives the largest square's side length.

```csharp
public class MaximalSquareDP
{
    public int MaximalSquare(int[][] matrix)
    {
        if (matrix == null || matrix.Length == 0 || matrix[0].Length == 0)
            return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int[,] dp = new int[rows, cols];
        int maxSide = 0;

        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                if (matrix[i][j] == 1)
                {
                    if (i == 0 || j == 0)
                    {
                        dp[i, j] = 1; // first row/column: a single 1 forms a 1x1 square
                    }
                    else
                    {
                        dp[i, j] = 1 + Math.Min(dp[i - 1, j],
                                       Math.Min(dp[i, j - 1], dp[i - 1, j - 1]));
                    }

                    maxSide = Math.Max(maxSide, dp[i, j]);
                }
                else
                {
                    dp[i, j] = 0;
                }
            }
        }

        return maxSide * maxSide;
    }
}
```

Time Complexity: `O(rows * cols)` — each cell is processed once with O(1) work.
Space Complexity: `O(rows * cols)` for the DP table. This can be optimized further to `O(cols)` by keeping only two rolling rows (the "previous row" and "current row"), since `dp[i][j]` only ever depends on the row directly above and the current row's previous column — the full 2D table is not strictly necessary.

**Explanation:**
Let's dry-run the DP fill on the example matrix (rows 0-3, cols 0-4):

```
Matrix:
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```

Row 0: every `1` in the first row can only be a `1x1` square (no row above it), so `dp[0][j] = matrix[0][j]`.
```
dp row 0: 1 0 1 0 0
```

Row 1:
- `dp[1][0] = 1` (first column, matrix value is 1).
- `dp[1][1] = 0` (matrix value is 0).
- `dp[1][2] = 1` (matrix[1][2]=1, but neighbors dp[0][2]=1, dp[1][1]=0, dp[0][1]=0 → min is 0 → `1+0=1`).
- `dp[1][3] = 1` (matrix=1, neighbors dp[0][3]=0, dp[1][2]=1, dp[0][2]=1 → min=0 → `1+0=1`).
- `dp[1][4] = 1` (matrix=1, neighbors dp[0][4]=0, dp[1][3]=1, dp[0][3]=0 → min=0 → `1+0=1`).
```
dp row 1: 1 0 1 1 1
```

Row 2:
- `dp[2][0] = 1` (first column, matrix=1).
- `dp[2][1] = 1` (matrix=1, first-row/col rule doesn't apply since j=1,i=2 both >0: neighbors dp[1][1]=0, dp[2][0]=1, dp[1][0]=1 → min=0 → `1+0=1`).
- `dp[2][2] = 2` (matrix=1, neighbors dp[1][2]=1, dp[2][1]=1, dp[1][1]=0 → min=0 → `1+0=1`)... let's recompute carefully: neighbors are top=dp[1][2]=1, left=dp[2][1]=1, top-left=dp[1][1]=0 → min(1,1,0)=0 → dp[2][2]=1+0=1.

  Correction — recompute row 2 fully and carefully:
```
- dp[2][0]: matrix=1, first col -> dp=1
- dp[2][1]: matrix=1, top=dp[1][1]=0, left=dp[2][0]=1, topleft=dp[1][0]=1 -> min=0 -> dp=1
- dp[2][2]: matrix=1, top=dp[1][2]=1, left=dp[2][1]=1, topleft=dp[1][1]=0 -> min=0 -> dp=1
- dp[2][3]: matrix=1, top=dp[1][3]=1, left=dp[2][2]=1, topleft=dp[1][2]=1 -> min=1 -> dp=2
- dp[2][4]: matrix=1, top=dp[1][4]=1, left=dp[2][3]=2, topleft=dp[1][3]=1 -> min=1 -> dp=2
```
```
dp row 2: 1 1 1 2 2
```

Row 3:
```
- dp[3][0]: matrix=1, first col -> dp=1
- dp[3][1]: matrix=0 -> dp=0
- dp[3][2]: matrix=0 -> dp=0
- dp[3][3]: matrix=1, top=dp[2][3]=2, left=dp[3][2]=0, topleft=dp[2][2]=1 -> min=0 -> dp=1
- dp[3][4]: matrix=0 -> dp=0
```
```
dp row 3: 1 0 0 1 0
```

Final DP table:
```
1 0 1 0 0
1 0 1 1 1
1 1 1 2 2
1 0 0 1 0
```

The maximum value in the table is `2`, occurring at `dp[2][3]` and `dp[2][4]`. This means the largest all-ones square has side length `2`. Why does `dp[i][j] = 1 + min(top, left, top-left)` work? To extend a square of side `k` ending at `(i,j)` to side `k+1`, you need the entire top edge, the entire left edge, and the top-left corner cell all to already support a square of side at least `k`. If any one of the three neighboring squares (top, left, top-left) is smaller, that smaller square becomes the bottleneck — you cannot form a bigger square than the weakest of the three supporting squares, plus the current cell itself. That's exactly why we take the `min` of the three neighbors and add `1`.

Answer for **Maximal Square**: `maxSide = 2` → area `= 2 * 2 = 4`, matching the expected output.

---

## 2. Count Square Submatrices with All Ones

### 2. Count Square Submatrices with All Ones

**Problem Statement:**
Given a binary matrix, count the total number of square submatrices (of any size, `1x1`, `2x2`, `3x3`, ...) that are made entirely of `1`s.

**Example:**
- Input:
```
1 0 1 0 0
1 0 1 1 1
1 1 1 1 1
1 0 0 1 0
```
- Output: `15`
- Explanation: Counting all-ones squares of every size: there are `13` squares of size `1x1` (one per `1` cell in the matrix) and `2` squares of size `2x2` — no `3x3` or larger all-ones square exists anywhere in this matrix. Total = `13 + 2 = 15`.

**Brute Force Approach:**
For each cell as a potential top-left corner, try every possible side length and check whether all the cells inside that square are `1`s; if so, count it as one valid square.

```csharp
public class CountSquaresBruteForce
{
    public int CountSquares(int[][] matrix)
    {
        if (matrix == null || matrix.Length == 0 || matrix[0].Length == 0)
            return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int count = 0;

        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                int maxPossibleSide = Math.Min(rows - i, cols - j);

                // Try every side length starting at this top-left corner.
                for (int side = 1; side <= maxPossibleSide; side++)
                {
                    if (IsAllOnesSquare(matrix, i, j, side))
                        count++;
                    else
                        break; // once a side fails, larger sides will also fail
                }
            }
        }

        return count;
    }

    private bool IsAllOnesSquare(int[][] matrix, int r, int c, int side)
    {
        for (int x = r; x < r + side; x++)
        {
            for (int y = c; y < c + side; y++)
            {
                if (matrix[x][y] != 1) return false;
            }
        }
        return true;
    }
}
```

Time Complexity: `O(rows * cols * min(rows, cols)^2)` in the worst case (dense matrix of all 1s), since for each cell we may validate squares of growing side length, each costing up to `O(side^2)`.
Space Complexity: `O(1)` extra space.

**Optimized DP Approach:**
Use the same `dp[i][j]` definition as before: the side length of the largest all-ones square with bottom-right corner at `(i, j)`.

```
dp[i][j] = 0                                                  if matrix[i][j] == 0
dp[i][j] = 1                                                  if matrix[i][j] == 1 and (i == 0 or j == 0)
dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])      otherwise
```

The key insight for **Count Squares**: `dp[i][j]` itself equals the number of distinct all-ones squares that have their bottom-right corner exactly at `(i, j)` — a cell with `dp[i][j] = k` is the bottom-right corner of exactly `k` valid squares, of sizes `1x1, 2x2, ..., k x k`, all ending at that same cell. So the total count of all-ones squares in the matrix is simply the **sum of every value** in the `dp` table.

```csharp
public class CountSquaresDP
{
    public int CountSquares(int[][] matrix)
    {
        if (matrix == null || matrix.Length == 0 || matrix[0].Length == 0)
            return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int[,] dp = new int[rows, cols];
        int totalCount = 0;

        for (int i = 0; i < rows; i++)
        {
            for (int j = 0; j < cols; j++)
            {
                if (matrix[i][j] == 1)
                {
                    if (i == 0 || j == 0)
                    {
                        dp[i, j] = 1; // 1x1 square, no room to extend
                    }
                    else
                    {
                        dp[i, j] = 1 + Math.Min(dp[i - 1, j],
                                       Math.Min(dp[i, j - 1], dp[i - 1, j - 1]));
                    }

                    totalCount += dp[i, j]; // dp[i][j] = number of squares ending at (i, j)
                }
                // matrix[i][j] == 0 contributes dp[i,j] = 0 (default) and adds nothing to totalCount
            }
        }

        return totalCount;
    }
}
```

Time Complexity: `O(rows * cols)` — one pass over the matrix, O(1) work per cell.
Space Complexity: `O(rows * cols)` for the DP table. As with Maximal Square, this can be reduced to `O(cols)` by keeping only two rolling rows (previous and current), since each cell's computation only needs the row above and the current row's previous entry — accumulating `totalCount` as you go still works correctly without storing the entire 2D table.

**Explanation:**
Reusing the DP table computed for the same example matrix:

```
Matrix:                      DP Table:
1 0 1 0 0                    1 0 1 0 0
1 0 1 1 1                    1 0 1 1 1
1 1 1 1 1                    1 1 1 2 2
1 0 0 1 0                    1 0 0 1 0
```

Why does `dp[i][j]` equal the *count* of squares ending at `(i,j)`, not just the *side* of the largest one? Because if the largest square ending at `(i,j)` has side `k`, that same bottom-right corner is automatically also the bottom-right corner of a smaller `(k-1)x(k-1)` square (just shrink from the top-left), and a `(k-2)x(k-2)` square, and so on down to a `1x1` square — all of these are valid, distinct, all-ones squares nested inside one another, all ending at `(i,j)`. That gives exactly `k` squares for a cell with `dp[i][j] = k`. This is the same reasoning as why `1 + min(top, left, top-left)` works for the max-square recurrence: the min of the three neighbors is the bottleneck that determines how far the square can be extended, and every smaller extension up to that bottleneck is also automatically valid.

Summing every value in the DP table:
```
Row 0: 1 + 0 + 1 + 0 + 0 = 2
Row 1: 1 + 0 + 1 + 1 + 1 = 4
Row 2: 1 + 1 + 1 + 2 + 2 = 7
Row 3: 1 + 0 + 0 + 1 + 0 = 2

Total = 2 + 4 + 7 + 2 = 15
```

Breaking this down by square size confirms the same total a different way: there are `13` cells in the matrix with value `1` (each is trivially a `1x1` all-ones square), and there are exactly `2` cells whose `dp` value is `2` — namely `dp[2][3]` and `dp[2][4]` — meaning `2` distinct `2x2` all-ones squares exist. No cell reaches `dp = 3`, so there is no `3x3` all-ones square anywhere in the matrix. Adding these up: `13` (from all `1x1` squares) `+ 2` (from all `2x2` squares) `= 15`, which exactly matches the direct sum of the DP table. This confirms why simply summing every entry of the DP table works in one pass — each `dp[i][j] = k` cell is silently bundling in `k` nested squares (of sizes `1x1` through `k x k`) all ending at that same bottom-right corner, so the plain sum never double-counts and never misses a square.

Answer for **Count Square Submatrices with All Ones**: sum of the DP table = total number of all-ones squares of every size in the matrix = **15**.
