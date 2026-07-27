# Recursion — Backtracking on Grids and Boards

## 1. Rat in a Maze

**Problem Statement:** Given an `n x n` grid (maze) of `0`s and `1`s, where `1` denotes an open cell and `0` denotes a blocked cell, find all possible paths for a rat starting at the top-left cell `(0, 0)` to reach the bottom-right cell `(n-1, n-1)`. The rat can move in some set of allowed directions (commonly Down, Left, Right, Up — sometimes restricted to just Down and Right). The rat cannot step on blocked cells or revisit a cell already used in the current path.

**Example:**
- Input:
  ```
  1 0 0 0
  1 1 0 1
  1 1 0 0
  0 1 1 1
  ```
- Output: `["DDRDRR", "DRDDRR"]`
- Explanation: Starting at `(0,0)`, two distinct sequences of moves (D = Down, R = Right, and so on) reach `(3,3)` while only passing through cells marked `1` and never revisiting a cell. Every other combination of moves either walks into a `0` cell, walks off the grid, or loops back onto a visited cell, so those branches are abandoned.

**Backtracking Approach:** This is the canonical backtracking template applied to grid traversal — try each direction, prune invalid moves, recurse, then undo the move so the cell is free for other paths.

**Logic (Steps):**
1. **Base case** — if the current cell `(row, col)` equals the destination `(n-1, n-1)`, record the accumulated `path` string into `result` and return.
2. **Mark visited** — mark the current cell visited before exploring its neighbors.
3. **Try a choice** — from the current cell, attempt to move in each allowed direction (Down, Left, Right, Up, in that priority order).
4. **Validity check (pruning)** — before recursing into a neighbor cell `(newRow, newCol)`, confirm it is in-bounds (`0 <= newRow < n` and `0 <= newCol < n`), that the cell is open (`grid[newRow][newCol] == 1`), and that it has not already been visited in the current path (`!visited[newRow][newCol]`). If any check fails, that branch is pruned immediately — no recursive call is made.
5. **Place the move / recurse** — append the move character to `path`, call `Solve` on the neighbor.
6. **Undo (backtrack)** — after the recursive call returns, remove the move character from `path`; once all 4 directions have been tried from the current cell, unmark it (`visited[row, col] = false`) so it can be reused by a different route from a different ancestor.

```csharp
public class RatInMaze
{
    private static readonly int[] dRow = { 1, 0, 0, -1 }; // Down, Left, Right, Up
    private static readonly int[] dCol = { 0, -1, 1, 0 };
    private static readonly char[] dChar = { 'D', 'L', 'R', 'U' };

    public IList<string> FindPaths(int[][] grid)
    {
        var result = new List<string>();
        int n = grid.Length;

        if (n == 0 || grid[0][0] == 0 || grid[n - 1][n - 1] == 0)
            return result;

        bool[,] visited = new bool[n, n];
        var path = new StringBuilder();

        Solve(grid, 0, 0, n, visited, path, result);
        return result;
    }

    private void Solve(int[][] grid, int row, int col, int n,
                        bool[,] visited, StringBuilder path, List<string> result)
    {
        // Base case: reached destination
        if (row == n - 1 && col == n - 1)
        {
            result.Add(path.ToString());
            return;
        }

        visited[row, col] = true; // mark current cell as part of this path

        for (int dir = 0; dir < 4; dir++)
        {
            int newRow = row + dRow[dir];
            int newCol = col + dCol[dir];

            // Validity / pruning check: bounds + open cell + not already visited
            if (IsValid(grid, newRow, newCol, n, visited))
            {
                path.Append(dChar[dir]);          // choose
                Solve(grid, newRow, newCol, n, visited, path, result); // recurse
                path.Remove(path.Length - 1, 1);  // undo the choice
            }
        }

        visited[row, col] = false; // unmark before returning to caller (backtrack)
    }

    private bool IsValid(int[][] grid, int row, int col, int n, bool[,] visited)
    {
        return row >= 0 && row < n && col >= 0 && col < n
            && grid[row][col] == 1 && !visited[row, col];
    }
}
```

Time Complexity: `O(4^(n*n))` in the absolute worst case, since at every cell up to 4 directions can be tried and a path can theoretically wander through all `n*n` cells before dead-ending. In practice, the "not visited" and "cell is open" checks prune the vast majority of branches almost immediately, so real runtimes on typical mazes are far smaller than the theoretical bound.
Space Complexity: `O(n*n)` for the visited matrix and the recursion stack depth (path length is bounded by the number of cells), plus the space needed to store the output paths.

**Walkthrough:** Using the 3x3 grid `[[1,0,0],[1,1,0],[0,1,1]]`, start at `(0,0)`, destination `(2,2)`.
- At `(0,0)`: mark visited, `path=""`. Try Down → `(1,0)` valid (`grid=1`, unvisited). Choose it, `path="D"`.
- At `(1,0)`: mark visited. Down → `(2,0)` has `grid=0`, prune. Left → out of bounds, prune. Right → `(1,1)` valid. Choose it, `path="DR"`.
- At `(1,1)`: mark visited. Down → `(2,1)` valid. Choose it, `path="DRD"`.
- At `(2,1)`: mark visited. Down → out of bounds, prune. Left → `(2,0)` has `grid=0`, prune. Right → `(2,2)` valid and is the destination! Choose it, `path="DRDR"`.
- At `(2,2)`: base case hit (`row==n-1 && col==n-1`) → `result.Add("DRDR")`.
- Backtrack: unmark `(2,2)`, remove 'R' → back at `(2,1)`. No other direction works (Up revisits `(1,1)`) → unmark `(2,1)`, remove 'D' → back at `(1,1)`.
- At `(1,1)`: remaining directions (Left, Right, Up) all prune (visited or `grid=0`) → unmark `(1,1)`, remove 'D' → back at `(1,0)`, then unwind fully back to `(0,0)` where no other direction is valid.
- Final `result = ["DRDR"]`, matching the single expected path.

---

## 2. Word Search

**Problem Statement:** Given an `m x n` grid of characters `board` and a string `word`, return `true` if `word` exists in the grid. The word must be constructed from letters of sequentially adjacent cells, where "adjacent" cells are horizontally or vertically neighboring. The same cell may not be used more than once within one word.

**Example:**
- Input:
  ```
  board = [
    ['A','B','C','E'],
    ['S','F','C','S'],
    ['A','D','E','E']
  ]
  word = "ABCCED"
  ```
- Output: `true`
- Explanation: Starting at `board[0][0] = 'A'`, moving right to `'B'`, right to `'C'`, down to `'C'`, down to `'E'`, left to `'D'` spells `"ABCCED"` using each cell at most once, so the word is found.

**Backtracking Approach:** Same backtracking skeleton as the maze problem, adapted to a 2D character grid with word-index matching, using the board itself as the visited marker.

**Logic (Steps):**
1. **Try a starting cell** — for every cell `(r, c)` in the board, attempt to match `word[0]` there and launch a DFS via `Backtrack`.
2. **Base case** — if `index == word.Length`, every character has matched, so return `true`.
3. **Validity check (pruning)** — at each recursive step matching `word[index]` at cell `(r, c)`: the cell must be in bounds and `board[r][c]` must equal `word[index]`. If either fails, prune (return false) without recursing further.
4. **Mark visited by mutating the board** — save `board[r][c]` as `original`, then temporarily overwrite it with the sentinel `'#'`, which cannot match any letter and so cheaply marks the cell as "in use" without extra memory.
5. **Recurse** — try all 4 neighboring directions (down, up, right, left) looking for `word[index+1]`, short-circuiting with `||` as soon as one succeeds.
6. **Undo (backtrack)** — after exploring all neighbors from `(r, c)`, restore `board[r][c] = original` before returning `found`, so the cell is available again for other starting points or other branches of the search.

```csharp
public class WordSearch
{
    public bool Exist(char[][] board, string word)
    {
        if (board == null || board.Length == 0 || string.IsNullOrEmpty(word))
            return false;

        int rows = board.Length, cols = board[0].Length;

        for (int r = 0; r < rows; r++)
        {
            for (int c = 0; c < cols; c++)
            {
                if (Backtrack(board, word, r, c, 0))
                    return true;
            }
        }
        return false;
    }

    private bool Backtrack(char[][] board, string word, int r, int c, int index)
    {
        // Base case: matched every character of the word
        if (index == word.Length)
            return true;

        int rows = board.Length, cols = board[0].Length;

        // Validity / pruning check: bounds + character match (also excludes
        // already-visited cells, since they hold the '#' sentinel)
        if (r < 0 || r >= rows || c < 0 || c >= cols || board[r][c] != word[index])
            return false;

        char original = board[r][c];
        board[r][c] = '#'; // mark visited by mutating the board in place

        bool found = Backtrack(board, word, r + 1, c, index + 1) ||
                     Backtrack(board, word, r - 1, c, index + 1) ||
                     Backtrack(board, word, r, c + 1, index + 1) ||
                     Backtrack(board, word, r, c - 1, index + 1);

        board[r][c] = original; // undo: restore original character (backtrack)

        return found;
    }
}
```

Time Complexity: `O(n * m * 4^L)` where `n * m` is the board size and `L` is the length of `word`. Each of the `n*m` starting cells can branch into up to 4 directions at each of the `L` steps in the worst case. The immediate character-mismatch pruning (a wrong letter aborts a branch in `O(1)`) means that on real boards with varied letters, the vast majority of the `4^L` branches are cut off after just one or two characters, so actual performance is usually close to linear in board size.
Space Complexity: `O(L)` for the recursion stack (depth equals word length); no extra visited matrix is needed since the board itself is mutated and restored.

**Walkthrough:** Using `board = [['A','B','C','E'],['S','F','C','S'],['A','D','E','E']]` and `word = "ABCCED"`.
- `Exist` loops over all cells as potential starts; `(0,0) = 'A'` matches `word[0]`, so `Backtrack(board, word, 0, 0, 0)` is called.
- `index=0`: `board[0][0]='A'==word[0]` → mark `board[0][0]='#'`, try neighbors for `index=1`.
- Right neighbor `(0,1)='B'==word[1]` → mark `'#'`, try neighbors for `index=2`.
- Right neighbor `(0,2)='C'==word[2]` → mark `'#'`, try neighbors for `index=3` (need `'C'` again).
- Down neighbor `(1,2)='C'==word[3]` → mark `'#'`, try neighbors for `index=4` (need `'E'`).
- Down neighbor `(2,2)='E'==word[4]` → mark `'#'`, try neighbors for `index=5` (need `'D'`).
- Left neighbor `(2,1)='D'==word[5]` → mark `'#'`, try neighbors for `index=6`.
- `index==word.Length (6)` → base case returns `true` immediately, which short-circuits all the `||` chains back up the stack — none of the boards get restored because the function returns before the undo line executes on the success path, but that's fine since the caller (`Exist`) already has its answer.
- If instead the down neighbor at `(1,2)` had NOT been `'C'` (a wrong branch), that recursive call would return `false` at the mismatch check, and the code would restore `board[1][2]` back to its original letter (backtrack) before trying the next of the 4 directions.
- Final result: `Exist` returns `true`, matching the expected output.

---

## 3. M-Coloring Problem

**Problem Statement:** Given an undirected graph with `V` vertices (represented, for example, as an adjacency matrix) and an integer `M`, determine whether the graph's vertices can be colored using at most `M` colors such that no two adjacent vertices share the same color. Return `true`/`false`, and typically also produce one valid color assignment (an array mapping each vertex to a color index) if one exists.

**Example:**
- Input: `V = 4` vertices with edges `(0,1), (1,2), (2,3), (3,0), (0,2)` and `M = 3`
- Output: `true`, with one valid coloring `color = [1, 2, 1, 2]` (1-indexed colors)
- Explanation: Vertex 0 gets color 1, vertex 1 (adjacent to 0) gets color 2, vertex 2 (adjacent to 0 and 1) needs a third distinct... but since 2 is adjacent to 0 (color 1) it can reuse color 1, and vertex 3 (adjacent to 2 and 0, both color 1) gets color 2. No two adjacent vertices end up with the same color, and only 2 of the allowed 3 colors were needed, so a valid coloring exists.

**Backtracking Approach:** Backtracking over vertices in order, trying every color at each step and checking adjacency conflicts before committing.

**Logic (Steps):**
1. **Base case** — if `vertex == v`, every vertex has been successfully colored, so return `true`.
2. **Try a choice** — for the current `vertex`, try assigning each color `c` from `1` to `m` in turn.
3. **Validity check (pruning)** — call `IsSafe`, which scans every other vertex `i` and confirms that if `graph[vertex, i]` is an edge, `color[i]` is not already the candidate color. If any adjacent vertex shares the candidate color, this color choice is invalid and is skipped without recursing.
4. **Place the value / recurse** — assign `color[vertex] = c`, then recurse into `SolveColoring(..., vertex + 1)`. If it returns `true`, propagate `true` immediately.
5. **Undo (backtrack)** — if the recursive call fails, reset `color[vertex] = 0` and try the next candidate color; if all `m` colors have been tried and none work, return `false` so the caller (the previous vertex in the call stack) backtracks and tries a different color for itself.

```csharp
public class MColoring
{
    public bool GraphColoring(bool[,] graph, int m, int v)
    {
        int[] color = new int[v]; // 0 means uncolored
        return SolveColoring(graph, m, v, color, 0);
    }

    private bool SolveColoring(bool[,] graph, int m, int v, int[] color, int vertex)
    {
        // Base case: all vertices successfully colored
        if (vertex == v)
            return true;

        for (int c = 1; c <= m; c++)
        {
            if (IsSafe(graph, color, v, vertex, c))
            {
                color[vertex] = c; // choose this color

                if (SolveColoring(graph, m, v, color, vertex + 1)) // recurse
                    return true;

                color[vertex] = 0; // undo: this color didn't lead to a solution
            }
        }

        return false; // no color works for this vertex given prior choices
    }

    private bool IsSafe(bool[,] graph, int[] color, int v, int vertex, int candidateColor)
    {
        // Validity check: no adjacent vertex may already have this color
        for (int i = 0; i < v; i++)
        {
            if (graph[vertex, i] && color[i] == candidateColor)
                return false;
        }
        return true;
    }
}
```

Time Complexity: `O(M^V)` in the worst case, since each of the `V` vertices can be tried with up to `M` colors, and the `IsSafe` adjacency check costs `O(V)` per attempt, giving `O(V * M^V)` overall. The adjacency-conflict pruning cuts off a branch the moment two adjacent vertices would clash, so dense or highly constrained graphs (which is most real inputs) are resolved far faster than the raw exponential bound suggests, because entire subtrees of color assignments become invalid after very few vertices are placed.
Space Complexity: `O(V)` for the `color[]` array and the recursion stack (depth equals `V`), plus `O(V^2)` if the graph itself is stored as an adjacency matrix.

**Walkthrough:** Using `V=4` with edges `(0,1), (1,2), (2,3), (3,0), (0,2)` and `M=3`.
- `vertex=0`: try `c=1`. `IsSafe` finds no colored neighbors yet → safe. `color=[1,0,0,0]`. Recurse to `vertex=1`.
- `vertex=1`: try `c=1`. Vertex 1 is adjacent to vertex 0 (`color[0]=1`) → conflict, unsafe. Try `c=2`. No adjacent vertex has color 2 → safe. `color=[1,2,0,0]`. Recurse to `vertex=2`.
- `vertex=2`: try `c=1`. Vertex 2 is adjacent to 0 (`color=1`, conflict) → unsafe. Try `c=2`. Vertex 2 is adjacent to 1 (`color=2`, conflict) → unsafe. Try `c=3`. No adjacent vertex has color 3 → safe. `color=[1,2,3,0]`. Recurse to `vertex=3`.
- `vertex=3`: try `c=1`. Vertex 3 is adjacent to 2 (`color=3`, fine) and 0 (`color=1`, conflict) → unsafe. Try `c=2`. Vertex 3 is adjacent to 2 (`color=3`, fine) and 0 (`color=1`, fine) → safe. `color=[1,2,3,2]`. Recurse to `vertex=4`.
- `vertex=4 == v` → base case, return `true` all the way up the call stack without ever needing to backtrack.
- `GraphColoring` returns `true` with a valid coloring found (colors used may differ slightly from the example's `[1,2,1,2]` depending on tie-breaking, but both are valid 3-colorings satisfying every edge constraint).

---

## 4. Sudoku Solver

**Problem Statement:** Given a partially filled `9 x 9` Sudoku board (empty cells marked, e.g., with `'.'`), fill in the empty cells so that every row, every column, and each of the nine `3 x 3` sub-boxes contains all digits `1` through `9` exactly once. Assume the given board has exactly one solution.

**Example:**
- Input: a `9x9` board with most cells filled and a few `'.'` cells, e.g. row 0 = `"53..7...."`, and so on for the remaining 8 rows.
- Output: the same board with every `'.'` replaced by a digit `1-9` such that all Sudoku constraints hold, e.g. row 0 becomes `"534678912"`.
- Explanation: Each empty cell is filled with the unique digit that satisfies its row, column, and 3x3 box, discovered by trying digits and backtracking whenever a choice leads to a dead end elsewhere on the board.

**Backtracking Approach:** Backtracking cell by cell over the empty positions, trying every digit and pruning against row/column/box constraints.

**Logic (Steps):**
1. **Find the next empty cell** — scan the board row by row, column by column, for the next cell where `board[row][col] == '.'`.
2. **Base case** — if the scan completes with no empty cell found, the board is fully and validly solved, so return `true`.
3. **Try a choice** — for that empty cell, try digits `'1'` through `'9'` in order.
4. **Validity check (pruning)** — for each candidate digit, call `IsValid(board, row, col, digit)`, which checks three constraints in one loop over `i = 0..8`: (a) the digit does not already appear anywhere in `row`, (b) the digit does not already appear anywhere in `col`, and (c) the digit does not already appear in the `3x3` box containing `(row, col)` (found via `boxRow = row - row % 3`, `boxCol = col - col % 3`). If any check fails, the digit is skipped.
5. **Place the value / recurse** — write the valid digit into `board[row][col]` and recurse on the rest of the board. If the recursive call succeeds, propagate success upward immediately.
6. **Undo (backtrack)** — if no subsequent placement leads to a full solution, reset `board[row][col]` back to `'.'` and try the next candidate digit; if all 9 digits fail for this cell, return `false`, forcing the previous cell in the call stack to try its next digit instead.

```csharp
public class SudokuSolver
{
    public void SolveSudoku(char[][] board)
    {
        Solve(board);
    }

    private bool Solve(char[][] board)
    {
        for (int row = 0; row < 9; row++)
        {
            for (int col = 0; col < 9; col++)
            {
                if (board[row][col] == '.') // find next empty cell
                {
                    for (char digit = '1'; digit <= '9'; digit++)
                    {
                        if (IsValid(board, row, col, digit))
                        {
                            board[row][col] = digit; // place the value

                            if (Solve(board)) // recurse on the rest of the board
                                return true;

                            board[row][col] = '.'; // undo: backtrack
                        }
                    }
                    return false; // no digit 1-9 works here; trigger backtracking above
                }
            }
        }
        return true; // no empty cells left: board is fully and validly solved
    }

    private bool IsValid(char[][] board, int row, int col, char digit)
    {
        int boxRow = row - row % 3;
        int boxCol = col - col % 3;

        for (int i = 0; i < 9; i++)
        {
            // Row check
            if (board[row][i] == digit) return false;
            // Column check
            if (board[i][col] == digit) return false;
            // 3x3 box check
            if (board[boxRow + i / 3][boxCol + i % 3] == digit) return false;
        }
        return true;
    }
}
```

Time Complexity: Roughly `O(9^(number of empty cells))` in the worst case, since each empty cell could in principle try up to 9 digits before the recursion resolves. In practice, `IsValid` prunes invalid digits in `O(1)`-ish amortized time (each check is `O(9)` over row/col/box), and as the board fills up, constraints cascade — most cells end up with only one or two legal digits — so real Sudoku puzzles solve almost instantly instead of exploring anywhere near the theoretical exponential bound.
Space Complexity: `O(1)` extra space beyond the board itself (aside from the recursion stack), since validity checks scan fixed-size rows/columns/boxes; recursion depth is bounded by the number of empty cells (at most 81).

**Walkthrough:** Using the example board where row 0 = `"53..7...."` and the target output row 0 = `"534678912"`.
- `Solve` scans and finds the first empty cell, `board[0][2] = '.'` (third character of row 0).
- Try digit `'1'`: `IsValid` checks row 0 (`'5'` and `'3'` present, no `'1'`) → passes row check; suppose column 2 and its 3x3 box also contain no `'1'` → `IsValid` returns `true`. Place `board[0][2] = '1'` and recurse.
- Deep in the recursion, further down the board, some later cell finds that every digit `1`-`9` fails `IsValid` (all conflict with existing digits) — that inner call returns `false`, propagating back.
- Back at `board[0][2]`, the placement of `'1'` is undone (`board[0][2] = '.'`) since it didn't lead to a full solution, and the next candidate is tried.
- Eventually digit `'4'` is tried at `board[0][2]`: `IsValid(board, 0, 2, '4')` computes `boxRow = 0 - 0%3 = 0`, `boxCol = 2 - 2%3 = 0`, checks row 0, column 2, and box (0,0)-(2,2) — none contain `'4'` → valid. Place `'4'`, and this choice (combined with all subsequent cells resolving consistently) leads all the way to a full board.
- `Solve` returns `true` at every level once the last empty cell is filled (no `'.'` remains), so no further backtracking occurs on the successful path.
- Final board matches the expected output, with row 0 becoming `"534678912"` — the `'.'` at index 2 correctly resolved to `'4'`.
