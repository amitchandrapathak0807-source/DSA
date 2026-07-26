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

**Approach:** This is the canonical backtracking template applied to grid traversal:
1. **Try a choice** — from the current cell, attempt to move in each allowed direction (Down, Left, Up, Right, in whatever priority order the problem wants the output sorted).
2. **Validity check (pruning)** — before recursing into a neighbor cell `(newRow, newCol)`, confirm it is in-bounds (`0 <= newRow < n` and `0 <= newCol < n`), that the cell is open (`grid[newRow][newCol] == 1`), and that it has not already been visited in the current path (`!visited[newRow][newCol]`). If any check fails, that branch is pruned immediately — no recursive call is made.
3. **Mark visited / place the move** — mark the cell visited and append the move character to the path string.
4. **Recurse** — call the same function on the neighbor. If the neighbor is the destination, record the accumulated path as one valid answer.
5. **Undo (backtrack)** — after the recursive call returns (regardless of whether it found paths or not), unmark the cell (`visited[newRow][newCol] = false`) and remove the move character from the path, so that the cell is free to be tried again via a different route from a different ancestor.

This mark-recurse-unmark cycle is what allows the same cell to be explored as part of many different candidate paths without contaminating sibling branches.

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

**Time Complexity:** `O(4^(n*n))` in the absolute worst case, since at every cell up to 4 directions can be tried and a path can theoretically wander through all `n*n` cells before dead-ending. In practice, the "not visited" and "cell is open" checks prune the vast majority of branches almost immediately, so real runtimes on typical mazes are far smaller than the theoretical bound.
**Space Complexity:** `O(n*n)` for the visited matrix and the recursion stack depth (path length is bounded by the number of cells), plus the space needed to store the output paths.

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

**Approach:** Same backtracking skeleton, adapted to a 2D character grid with word-index matching:
1. **Try a starting cell** — for every cell `(r, c)` in the board, attempt to match `word[0]` there and launch a DFS.
2. **Validity check (pruning)** — at each recursive step matching `word[k]` at cell `(r, c)`: the cell must be in bounds, must not already be used in the current path, and `board[r][c]` must equal `word[k]`. If any of these fail, prune (return false) without recursing further.
3. **Mark visited by mutating the board** — instead of a separate boolean matrix, a common idiomatic trick is to temporarily overwrite `board[r][c]` with a sentinel character (e.g. `'#'`) that cannot match any letter, which cheaply marks the cell as "in use" without extra memory.
4. **Recurse** — try all 4 neighboring directions looking for `word[k+1]`. If `k` reaches `word.Length - 1` with a match, the word is found.
5. **Undo (backtrack)** — after exploring all neighbors from `(r, c)`, restore `board[r][c]` to its original character before returning, so the cell is available again for other starting points or other branches of the search.

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

**Time Complexity:** `O(n * m * 4^L)` where `n * m` is the board size and `L` is the length of `word`. Each of the `n*m` starting cells can branch into up to 4 directions at each of the `L` steps in the worst case. The immediate character-mismatch pruning (a wrong letter aborts a branch in `O(1)`) means that on real boards with varied letters, the vast majority of the `4^L` branches are cut off after just one or two characters, so actual performance is usually close to linear in board size.
**Space Complexity:** `O(L)` for the recursion stack (depth equals word length); no extra visited matrix is needed since the board itself is mutated and restored.

## 3. M-Coloring Problem

**Problem Statement:** Given an undirected graph with `V` vertices (represented, for example, as an adjacency matrix) and an integer `M`, determine whether the graph's vertices can be colored using at most `M` colors such that no two adjacent vertices share the same color. Return `true`/`false`, and typically also produce one valid color assignment (an array mapping each vertex to a color index) if one exists.

**Example:**
- Input: `V = 4` vertices with edges `(0,1), (1,2), (2,3), (3,0), (0,2)` and `M = 3`
- Output: `true`, with one valid coloring `color = [1, 2, 1, 2]` (1-indexed colors)
- Explanation: Vertex 0 gets color 1, vertex 1 (adjacent to 0) gets color 2, vertex 2 (adjacent to 0 and 1) needs a third distinct... but since 2 is adjacent to 0 (color 1) it can reuse color 1, and vertex 3 (adjacent to 2 and 0, both color 1) gets color 2. No two adjacent vertices end up with the same color, and only 2 of the allowed 3 colors were needed, so a valid coloring exists.

**Approach:** Backtracking over vertices, trying every color at each step:
1. **Try a choice** — process vertices in order `0, 1, ..., V-1`. For the current vertex, try assigning each color from `1` to `M` in turn.
2. **Validity check (pruning)** — before committing a color to the current vertex, check every other vertex that is adjacent to it (via the adjacency matrix/list) and confirm none of them already has that same color. If any adjacent vertex shares the candidate color, this color choice is invalid and is skipped without recursing.
3. **Place the value** — assign the candidate color to the current vertex's slot in the `color[]` array.
4. **Recurse** — move on to color the next vertex (`vertex + 1`). If all `V` vertices get colored successfully, a valid coloring has been found.
5. **Undo (backtrack)** — if the recursive call on the next vertex fails to find a valid coloring for the rest of the graph, reset the current vertex's color back to `0` (uncolored) and try the next candidate color; if all `M` colors have been tried and none work, return `false` so the caller (the previous vertex) backtracks and tries a different color for itself.

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

**Time Complexity:** `O(M^V)` in the worst case, since each of the `V` vertices can be tried with up to `M` colors, and the `IsSafe` adjacency check costs `O(V)` per attempt, giving `O(V * M^V)` overall. The adjacency-conflict pruning cuts off a branch the moment two adjacent vertices would clash, so dense or highly constrained graphs (which is most real inputs) are resolved far faster than the raw exponential bound suggests, because entire subtrees of color assignments become invalid after very few vertices are placed.
**Space Complexity:** `O(V)` for the `color[]` array and the recursion stack (depth equals `V`), plus `O(V^2)` if the graph itself is stored as an adjacency matrix.

## 4. Sudoku Solver

**Problem Statement:** Given a partially filled `9 x 9` Sudoku board (empty cells marked, e.g., with `'.'`), fill in the empty cells so that every row, every column, and each of the nine `3 x 3` sub-boxes contains all digits `1` through `9` exactly once. Assume the given board has exactly one solution.

**Example:**
- Input: a `9x9` board with most cells filled and a few `'.'` cells, e.g. row 0 = `"53..7...."`, and so on for the remaining 8 rows.
- Output: the same board with every `'.'` replaced by a digit `1-9` such that all Sudoku constraints hold, e.g. row 0 becomes `"534678912"`.
- Explanation: Each empty cell is filled with the unique digit that satisfies its row, column, and 3x3 box, discovered by trying digits and backtracking whenever a choice leads to a dead end elsewhere on the board.

**Approach:** Backtracking cell by cell over the empty positions:
1. **Try a choice** — scan the board for the next empty cell (`'.'`). For that cell, try digits `'1'` through `'9'` in order.
2. **Validity check (pruning)** — for each candidate digit, call `IsValid(board, row, col, digit)`, which checks three constraints: (a) the digit does not already appear anywhere in `row`, (b) the digit does not already appear anywhere in `col`, and (c) the digit does not already appear in the `3x3` box containing `(row, col)` (found via `boxRow = row - row % 3`, `boxCol = col - col % 3`). If any of the three checks fails, the digit is skipped.
3. **Place the value** — write the valid digit into `board[row][col]`.
4. **Recurse** — attempt to solve the rest of the board from the next empty cell. If the recursive call succeeds, propagate success upward immediately.
5. **Undo (backtrack)** — if no subsequent placement leads to a full solution, reset `board[row][col]` back to `'.'` and try the next candidate digit; if all 9 digits fail for this cell, return `false`, forcing the previous cell in the call stack to try its next digit instead.

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

**Time Complexity:** Roughly `O(9^(number of empty cells))` in the worst case, since each empty cell could in principle try up to 9 digits before the recursion resolves. In practice, `IsValid` prunes invalid digits in `O(1)`-ish amortized time (each check is `O(9)` over row/col/box), and as the board fills up, constraints cascade — most cells end up with only one or two legal digits — so real Sudoku puzzles solve almost instantly instead of exploring anywhere near the theoretical exponential bound.
**Space Complexity:** `O(1)` extra space beyond the board itself (aside from the recursion stack), since validity checks scan fixed-size rows/columns/boxes; recursion depth is bounded by the number of empty cells (at most 81).

## Explanation — Dry Runs

**Rat in a Maze dry run on a 3x3 grid:**

Consider the grid:
```
1 0 0
1 1 0
0 1 1
```
Start at `(0,0)`, destination `(2,2)`. Directions tried in order Down, Left, Right, Up.

- At `(0,0)`: mark visited, path = "". Try Down → `(1,0)` is valid (`grid=1`, unvisited). Choose it, path = "D".
- At `(1,0)`: mark visited. Try Down → `(2,0)` has `grid=0` → invalid, prune. Try Left → `(1,-1)` out of bounds → prune. Try Right → `(1,1)` valid (`grid=1`, unvisited). Choose it, path = "DR".
- At `(1,1)`: mark visited. Try Down → `(2,1)` valid (`grid=1`, unvisited). Choose it, path = "DRD".
- At `(2,1)`: mark visited. Try Down → `(3,1)` out of bounds → prune. Try Left → `(2,0)` has `grid=0` → prune. Try Right → `(2,2)` valid and is the destination! Choose it, path = "DRDR".
- At `(2,2)`: this is `(n-1, n-1)` → base case hit, record path "DRDR" into results.
- Backtrack: unmark `(2,2)`, remove 'R' from path → back to `(2,1)` with path "DRD". No more directions to try from `(2,1)` (Up would go back to `(1,1)`, already visited) → unmark `(2,1)`, remove 'D' from path → back to `(1,1)` with path "DR".
- At `(1,1)`: continue trying remaining directions after Down — Left → `(1,0)` already visited → prune. Right → `(1,2)` has `grid=0` → prune. Up → `(0,1)` has `grid=0` → prune. All options exhausted → unmark `(1,1)`, remove 'D' from path → back to `(1,0)` with path "D".
- At `(1,0)`: continue after Right — Up → `(0,0)` already visited → prune. All options exhausted → unmark `(1,0)`, remove 'R' then 'D' as the stack unwinds → back to `(0,0)`.
- At `(0,0)`: continue trying remaining directions after Down — Left/Right go out of bounds, Up goes out of bounds → all exhausted → unmark `(0,0)`, function returns.

Final result: `["DRDR"]` — one path found, with a dead end at `(1,1)` (after reaching it a second logical route) correctly abandoned via backtracking.

**Sudoku `IsValid(board, row, col, digit)` dry run:**

Suppose we are trying to place digit `'4'` at `row = 4, col = 5` on a partially filled board.

1. Compute box anchor: `boxRow = 4 - 4 % 3 = 3`, `boxCol = 5 - 5 % 3 = 3`. So the relevant 3x3 box spans rows 3-5, columns 3-5.
2. Loop `i` from 0 to 8:
   - Row check: examine `board[4][0], board[4][1], ..., board[4][8]`. Suppose `board[4][2] = '4'` already. As soon as `i = 2` is reached, `board[row][i] == digit` is true → return `false` immediately.
3. Because the row check already failed, the column check and box check are never reached for this attempt — the function short-circuits and reports the placement invalid.

Now suppose instead the row had no `'4'`, e.g. row 4 contains `"...4..."` was actually `"...5..."` (no 4 present):
1. Row check passes for all `i` (no `board[4][i] == '4'`).
2. Column check: examine `board[0][5], board[1][5], ..., board[8][5]`. Suppose none equal `'4'` → passes.
3. Box check: examine the 3x3 box starting at `(3,3)`, i.e., cells `(3,3),(3,4),(3,5),(4,3),(4,4),(4,5),(5,3),(5,4),(5,5)` via the index formula `board[boxRow + i/3][boxCol + i%3]`. Suppose none of these equal `'4'` → passes.
4. All three checks pass → `IsValid` returns `true`, and the solver proceeds to place `'4'` at `board[4][5]` and recurse into the next empty cell.
