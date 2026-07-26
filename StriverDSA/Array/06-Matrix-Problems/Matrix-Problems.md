# Array — Matrix Problems

## 1. Set Matrix Zeroes

### 1. Set Matrix Zeroes

**Problem Statement:** Given an `m x n` matrix, if an element is `0`, set its entire row and column to `0`. This must be done in place, and ideally the extra space used should be `O(1)` (excluding the output/input matrix itself).

For example, if any cell in the matrix contains `0`, every cell that shares that cell's row or that cell's column must also become `0`.

**Example:**
- Input: `[[1,1,1],[1,0,1],[1,1,1]]`
- Output: `[[1,0,1],[0,0,0],[1,0,1]]`
- Explanation: The `0` sits at row 1, column 1. So all of row 1 becomes `0`, and all of column 1 becomes `0`. Every other cell that is not in row 1 or column 1 stays unchanged.

**Brute Force Approach:** Use two auxiliary arrays, `rows[]` and `cols[]`, of size `m` and `n` respectively, initialized to `false`. Scan the matrix once — whenever `matrix[i][j] == 0`, mark `rows[i] = true` and `cols[j] = true`. Then scan the matrix a second time, and set `matrix[i][j] = 0` whenever `rows[i]` or `cols[j]` is `true`.

```csharp
public int[,] SetZeroesBrute(int[,] matrix)
{
    int m = matrix.GetLength(0);
    int n = matrix.GetLength(1);

    bool[] rows = new bool[m];
    bool[] cols = new bool[n];

    // First pass: record which rows/cols must be zeroed
    for (int i = 0; i < m; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (matrix[i, j] == 0)
            {
                rows[i] = true;
                cols[j] = true;
            }
        }
    }

    // Second pass: apply the zeroing
    for (int i = 0; i < m; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (rows[i] || cols[j])
            {
                matrix[i, j] = 0;
            }
        }
    }

    return matrix;
}
```

**Time Complexity:** `O(m * n)` — two full passes over the matrix, each visiting every cell once.
**Space Complexity:** `O(m + n)` — for the `rows[]` and `cols[]` marker arrays.

**Optimized Approach:** Instead of separate `rows[]`/`cols[]` arrays, reuse the matrix's own first row and first column as the marker arrays. `matrix[i][0]` marks whether row `i` needs zeroing, and `matrix[0][j]` marks whether column `j` needs zeroing. The catch: cell `matrix[0][0]` is shared by both row 0 and column 0, so we need one separate boolean variable, `col0`, to track whether column 0 itself must be zeroed (since `matrix[0][0]` alone cannot represent both "row 0 needs zero" and "col 0 needs zero").

Steps:
1. Use a variable `col0 = 1` (true) initially.
2. Traverse the matrix from `(0,0)` to `(m-1, n-1)`. For every `matrix[i][j] == 0`, set `matrix[i][0] = 0` and: if `j == 0`, set `col0 = 0`; else set `matrix[0][j] = 0`.
3. Traverse the matrix again, this time from `(m-1, n-1)` backward to `(0,1)` — i.e., skip column 0 — and for each cell `(i, j)` with `j >= 1`, if `matrix[i][0] == 0` or `matrix[0][j] == 0`, set `matrix[i][j] = 0`.
4. Finally, check `matrix[0][0]`: if it is `0`, zero out the entire first row. Check `col0`: if it is `0`, zero out the entire first column.

```csharp
public int[,] SetZeroesOptimized(int[,] matrix)
{
    int m = matrix.GetLength(0);
    int n = matrix.GetLength(1);
    int col0 = 1; // tracks whether column 0 must be zeroed

    // Step 1: use row 0 and col 0 as marker arrays
    for (int i = 0; i < m; i++)
    {
        for (int j = 0; j < n; j++)
        {
            if (matrix[i, j] == 0)
            {
                matrix[i, 0] = 0;
                if (j != 0)
                {
                    matrix[0, j] = 0;
                }
                else
                {
                    col0 = 0;
                }
            }
        }
    }

    // Step 2: zero cells based on markers, skipping row 0 and col 0
    for (int i = m - 1; i >= 0; i--)
    {
        for (int j = n - 1; j >= 1; j--)
        {
            if (matrix[i, 0] == 0 || matrix[0, j] == 0)
            {
                matrix[i, j] = 0;
            }
        }

        // Step 3: handle column 0 using the separate col0 flag
        if (col0 == 0)
        {
            matrix[i, 0] = 0;
        }
    }

    return matrix;
}
```

**Time Complexity:** `O(m * n)` — a first full pass to mark, and a second full pass (in reverse) to apply zeroes. Still linear in the number of cells, just done with two passes instead of two-plus-an-extra-array.
**Space Complexity:** `O(1)` — no extra array is allocated; the matrix's own first row/column plus a single `col0` variable serve as the markers.

**Explanation:** Dry run on `[[1,1,1],[1,0,1],[1,1,1]]` (0-indexed rows/cols 0..2):

- Initial: `col0 = 1`.
- Pass 1 (marking): scan all cells. Only `matrix[1,1] == 0`. Since `j = 1 != 0`, set `matrix[1,0] = 0` and `matrix[0,1] = 0`. `col0` remains `1` because the zero was not in column 0.
  - Matrix now looks like: `[[1,0,1],[0,0,1],[1,1,1]]` (row 0 and col 0 now carry markers).
- Pass 2 (applying, iterating `i` from `2` down to `0`, and for each row `j` from `2` down to `1`, skipping `j=0`):
  - `i=2`: `matrix[2,0]=1`, `matrix[0,2]=1` → for `j=2`: neither marker is 0, cell stays `1`. For `j=1`: `matrix[0,1]=0` → set `matrix[2,1]=0`. Then check `col0==1`, so `matrix[2,0]` stays as is (`1`).
  - `i=1`: `matrix[1,0]=0` → for `j=2`: `matrix[1,0]==0` → set `matrix[1,2]=0`. For `j=1`: `matrix[1,0]==0` → set `matrix[1,1]=0` (already 0). Then `col0==1`, so `matrix[1,0]` stays `0` (this is correct, it was already marked and legitimately needs to be zero since it is in row 1).
  - `i=0`: for `j=2`: `matrix[0,2]=1`, `matrix[0,0]` not checked here (row 0 col 0 excluded from inner check target only via j>=1, but matrix[0,0] as a *marker source* is still valid) — `matrix[0,0]=1` and `matrix[0,2]=1`, no zero, so `matrix[0,2]` stays `1`... but wait we must also check `matrix[0,0]` for row 0's own marker in step 3 (handled after the loop, not shown as a per-row col0 step for row 0 explicitly — it's covered by the final explicit check below). For `j=1`: `matrix[0,1]=0` → set `matrix[0,1]=0` (already 0). `col0==1` so `matrix[0,0]` untouched.
- Step 4 (final check): `matrix[0,0] == 1`, so row 0 is **not** entirely zeroed beyond what markers already indicated. `col0 == 1`, so column 0 is **not** entirely zeroed either.
- Final matrix: `[[1,0,1],[0,0,0],[1,0,1]]`, which matches the expected output. The key subtlety is that `matrix[0][0]` doubles as the marker for "should row 0 be zeroed," while the separate `col0` variable is required because `matrix[0][0]` cannot simultaneously encode "should column 0 be zeroed" — using it for both would cause ambiguity when only one of row 0 or column 0 actually needs zeroing.

---

## 2. Rotate Matrix by 90 Degrees (In-Place)

### 2. Rotate Matrix by 90 Degrees (In-Place)

**Problem Statement:** Given an `n x n` square matrix, rotate it by 90 degrees clockwise, in place, without using another matrix (ideally `O(1)` extra space).

**Example:**
- Input: `[[1,2,3],[4,5,6],[7,8,9]]`
- Output: `[[7,4,1],[8,5,2],[9,6,3]]`
- Explanation: Rotating clockwise by 90 degrees means the first column (read bottom to top) becomes the first row, the second column (bottom to top) becomes the second row, and so on. Column `[7,4,1]` (bottom-to-top) becomes row `[7,4,1]`.

**Brute Force Approach:** Create a new matrix of the same size. For every cell `(i, j)` in the original matrix, place its value into the new matrix at position `(j, n-1-i)` (the formula for a 90-degree clockwise rotation). Then copy the new matrix back, or simply return it.

```csharp
public int[,] RotateBrute(int[,] matrix)
{
    int n = matrix.GetLength(0);
    int[,] rotated = new int[n, n];

    for (int i = 0; i < n; i++)
    {
        for (int j = 0; j < n; j++)
        {
            rotated[j, n - 1 - i] = matrix[i, j];
        }
    }

    return rotated;
}
```

**Time Complexity:** `O(n^2)` — every cell of the `n x n` matrix is visited and copied exactly once.
**Space Complexity:** `O(n^2)` — a brand-new `n x n` matrix is allocated to hold the rotated result.

**Optimized Approach:** Perform the rotation in place using two well-known steps:
1. **Transpose** the matrix: swap `matrix[i][j]` with `matrix[j][i]` for all `i < j`. This flips the matrix across its main diagonal, turning rows into columns.
2. **Reverse each row**: after transposing, reverse every individual row from left to right. The combination of transpose + row-reversal produces exactly a 90-degree clockwise rotation.

```csharp
public int[,] RotateOptimized(int[,] matrix)
{
    int n = matrix.GetLength(0);

    // Step 1: Transpose the matrix in place (swap across main diagonal)
    for (int i = 0; i < n; i++)
    {
        for (int j = i + 1; j < n; j++)
        {
            int temp = matrix[i, j];
            matrix[i, j] = matrix[j, i];
            matrix[j, i] = temp;
        }
    }

    // Step 2: Reverse each row
    for (int i = 0; i < n; i++)
    {
        int left = 0;
        int right = n - 1;
        while (left < right)
        {
            int temp = matrix[i, left];
            matrix[i, left] = matrix[i, right];
            matrix[i, right] = temp;
            left++;
            right--;
        }
    }

    return matrix;
}
```

**Time Complexity:** `O(n^2)` — the transpose step touches roughly half the cells (`n*(n-1)/2` swaps), and the row-reversal step touches every cell once; both are linear in the number of cells, so overall it is `O(n^2)`.
**Space Complexity:** `O(1)` — all swaps happen directly within the original matrix; no auxiliary matrix is allocated.

**Explanation:** Dry run on `[[1,2,3],[4,5,6],[7,8,9]]` (`n = 3`):

- **Transpose step** (swap `matrix[i][j]` with `matrix[j][i]` for `i < j`):
  - `(0,1)` ↔ `(1,0)`: swap `2` and `4` → row 0 becomes `[1,4,3]`, row 1 becomes `[2,5,6]`.
  - `(0,2)` ↔ `(2,0)`: swap `3` and `7` → row 0 becomes `[1,4,7]`, row 2 becomes `[3,8,9]`.
  - `(1,2)` ↔ `(2,1)`: swap `6` and `8` → row 1 becomes `[2,5,8]`, row 2 becomes `[3,6,9]`.
  - After transpose: `[[1,4,7],[2,5,8],[3,6,9]]`.
- **Reverse each row step:**
  - Row 0 `[1,4,7]` reversed → `[7,4,1]`.
  - Row 1 `[2,5,8]` reversed → `[8,5,2]`.
  - Row 2 `[3,6,9]` reversed → `[9,6,3]`.
- Final matrix: `[[7,4,1],[8,5,2],[9,6,3]]`, which matches the expected output exactly. Intuitively, transposing turns the original first column `[1,4,7]` into the new first row `[1,4,7]`, and reversing that row flips it into `[7,4,1]` — which is precisely the original first column read bottom-to-top, confirming the clockwise rotation.

---

## 3. Print the Matrix in Spiral Order

### 3. Print the Matrix in Spiral Order

**Problem Statement:** Given an `m x n` matrix, return all elements of the matrix in spiral order — starting from the top-left corner, moving right across the top row, then down the right column, then left across the bottom row, then up the left column, and continuing to spiral inward until every element has been visited.

**Example:**
- Input: `[[1,2,3],[4,5,6],[7,8,9]]`
- Output: `[1,2,3,6,9,8,7,4,5]`
- Explanation: Start at top-left, go right across the top row (`1,2,3`), then down the right column (`6,9`), then left across the bottom row (`8,7`), then up the left column (`4`), then the only remaining element is the center (`5`).

**Brute Force Approach:** One brute-force-style way (still requiring `O(m*n)` extra space for the visited marks) is to use a `visited` boolean matrix and simulate movement with a direction vector, turning clockwise whenever the next cell is out of bounds or already visited. This avoids using boundary math but costs an extra `O(m*n)` marker matrix.

```csharp
public List<int> SpiralOrderBrute(int[,] matrix)
{
    int m = matrix.GetLength(0);
    int n = matrix.GetLength(1);
    List<int> result = new List<int>();
    bool[,] visited = new bool[m, n];

    // Right, Down, Left, Up
    int[] dRow = { 0, 1, 0, -1 };
    int[] dCol = { 1, 0, -1, 0 };

    int row = 0, col = 0, dir = 0;

    for (int count = 0; count < m * n; count++)
    {
        result.Add(matrix[row, col]);
        visited[row, col] = true;

        int nextRow = row + dRow[dir];
        int nextCol = col + dCol[dir];

        if (nextRow < 0 || nextRow >= m || nextCol < 0 || nextCol >= n || visited[nextRow, nextCol])
        {
            dir = (dir + 1) % 4; // turn clockwise
            nextRow = row + dRow[dir];
            nextCol = col + dCol[dir];
        }

        row = nextRow;
        col = nextCol;
    }

    return result;
}
```

**Time Complexity:** `O(m * n)` — every cell is visited and added to the result exactly once.
**Space Complexity:** `O(m * n)` — for the `visited` marker matrix (in addition to the `O(m*n)` output list, which is unavoidable since it holds every element).

**Optimized Approach:** Use four boundary pointers — `top`, `bottom`, `left`, `right` — initialized to the outer edges of the matrix (`top = 0`, `bottom = m-1`, `left = 0`, `right = n-1`). Traverse in four directions, shrinking the corresponding boundary after each pass:
1. Traverse `left → right` along `row = top`, then do `top++`.
2. Traverse `top → bottom` along `col = right`, then do `right--`.
3. If `top <= bottom`, traverse `right → left` along `row = bottom`, then do `bottom--`.
4. If `left <= right`, traverse `bottom → top` along `col = left`, then do `left++`.

Repeat while `top <= bottom && left <= right`. No extra marker matrix is needed — just constant-space pointers.

```csharp
public List<int> SpiralOrderOptimized(int[,] matrix)
{
    int m = matrix.GetLength(0);
    int n = matrix.GetLength(1);
    List<int> result = new List<int>();

    int top = 0, bottom = m - 1;
    int left = 0, right = n - 1;

    while (top <= bottom && left <= right)
    {
        // 1. Left to right along the top row
        for (int j = left; j <= right; j++)
        {
            result.Add(matrix[top, j]);
        }
        top++;

        // 2. Top to bottom along the right column
        for (int i = top; i <= bottom; i++)
        {
            result.Add(matrix[i, right]);
        }
        right--;

        // 3. Right to left along the bottom row (only if a row remains)
        if (top <= bottom)
        {
            for (int j = right; j >= left; j--)
            {
                result.Add(matrix[bottom, j]);
            }
            bottom--;
        }

        // 4. Bottom to top along the left column (only if a column remains)
        if (left <= right)
        {
            for (int i = bottom; i >= top; i--)
            {
                result.Add(matrix[i, left]);
            }
            left++;
        }
    }

    return result;
}
```

**Time Complexity:** `O(m * n)` — each of the `m * n` elements is added to the result exactly once across all the directional sweeps combined.
**Space Complexity:** `O(1)` extra space — only the four boundary pointers are used (the output list itself is not counted as "extra" since returning all elements is the required result).

**Explanation:** Dry run on `[[1,2,3],[4,5,6],[7,8,9]]` (`m=3, n=3`), initial `top=0, bottom=2, left=0, right=2`:

- **Iteration 1** (`top(0) <= bottom(2)` and `left(0) <= right(2)`, so loop runs):
  - Step 1: left→right along row `top=0`: add `matrix[0,0]=1, matrix[0,1]=2, matrix[0,2]=3` → result `[1,2,3]`. Then `top++` → `top=1`.
  - Step 2: top→bottom along col `right=2`: add `matrix[1,2]=6, matrix[2,2]=9` → result `[1,2,3,6,9]`. Then `right--` → `right=1`.
  - Step 3: check `top(1) <= bottom(2)` → true. Right→left along row `bottom=2`: add `matrix[2,1]=8, matrix[2,0]=7` → result `[1,2,3,6,9,8,7]`. Then `bottom--` → `bottom=1`.
  - Step 4: check `left(0) <= right(1)` → true. Bottom→top along col `left=0`: add `matrix[1,0]=4` (loop from `i=bottom=1` down to `i=top=1`, just one element) → result `[1,2,3,6,9,8,7,4]`. Then `left++` → `left=1`.
- **Iteration 2** check: `top(1) <= bottom(1)` and `left(1) <= right(1)` → true, loop runs again:
  - Step 1: left→right along row `top=1`, from `j=left=1` to `j=right=1`: add `matrix[1,1]=5` → result `[1,2,3,6,9,8,7,4,5]`. Then `top++` → `top=2`.
  - Step 2: top→bottom along col `right=1`, from `i=top=2` to `i=bottom=1`: the loop condition `i <= bottom` (`2 <= 1`) is false, so nothing is added. Then `right--` → `right=0`.
  - Step 3: check `top(2) <= bottom(1)` → false, skip.
  - Step 4: check `left(1) <= right(0)` → false, skip.
- **Loop condition check:** `top(2) <= bottom(1)` → false, so the outer `while` loop terminates.
- Final result: `[1,2,3,6,9,8,7,4,5]`, matching the expected output. The boundaries shrink after each directional sweep (`top` grows, `bottom` shrinks, `right` shrinks, `left` grows), and the guards in steps 3 and 4 (`if top <= bottom` / `if left <= right`) prevent re-visiting a row or column that has already been fully consumed by an earlier step within the same iteration — this is essential for matrices that are not perfectly square or that have odd dimensions like the single-row/single-column case left in the center here.
