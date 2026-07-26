# Binary Search — Binary Search on 2D Arrays

## 1. Find the Row with Maximum Number of 1s

### 1. Find the Row with Maximum Number of 1s

**Problem Statement:**
Given a binary matrix (each cell is either `0` or `1`) where every row is sorted in non-decreasing order (all `0`s followed by all `1`s), find the index of the row that contains the maximum number of `1`s. If multiple rows have the same maximum count, return the row with the smallest index. If no row contains a `1`, return `-1`.

**Example:**
- Input:
```
[[0,0,0,1],
 [0,1,1,1],
 [0,0,1,1]]
```
- Output: `1`
- Explanation: Row 0 has one `1`, row 1 has three `1`s, row 2 has two `1`s. Row 1 has the maximum count, so the answer is `1`.

**Brute Force Approach:**
Scan every cell of the matrix, count the number of `1`s in each row using a simple linear traversal, and keep track of the row with the maximum count.

```csharp
public int RowWithMax1sBrute(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int maxCount = -1;
    int maxRowIndex = -1;

    for (int i = 0; i < rows; i++)
    {
        int countOnes = 0;
        for (int j = 0; j < cols; j++)
        {
            if (matrix[i, j] == 1)
            {
                countOnes++;
            }
        }

        if (countOnes > maxCount)
        {
            maxCount = countOnes;
            maxRowIndex = i;
        }
    }

    return maxRowIndex;
}
```

Time Complexity: `O(n * m)` — every cell of the matrix is visited once.
Space Complexity: `O(1)` — no extra space used.

**Optimized Approach:**
Since each row is sorted, the count of `1`s in a row equals `cols - (index of first 1)`. We can binary search for the "lower bound" of `1` in each row instead of scanning it linearly. This reduces the per-row cost from `O(m)` to `O(log m)`.

```csharp
public int RowWithMax1sOptimized(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int maxCount = -1;
    int maxRowIndex = -1;

    for (int i = 0; i < rows; i++)
    {
        int firstOneIndex = LowerBoundOfOne(matrix, i, cols);
        int countOnes = (firstOneIndex == -1) ? 0 : cols - firstOneIndex;

        if (countOnes > maxCount)
        {
            maxCount = countOnes;
            maxRowIndex = i;
        }
    }

    return maxRowIndex;
}

// Binary search for the first occurrence of 1 in row 'row'
private int LowerBoundOfOne(int[,] matrix, int row, int cols)
{
    int low = 0, high = cols - 1;
    int firstOneIndex = -1;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        if (matrix[row, mid] == 1)
        {
            firstOneIndex = mid;
            high = mid - 1; // search left half for an earlier 1
        }
        else
        {
            low = mid + 1;
        }
    }

    return firstOneIndex;
}
```

Time Complexity: `O(n * log m)` — for each of the `n` rows we binary search over `m` columns.
Space Complexity: `O(1)` — only a constant number of extra variables are used.

**Explanation:**
Row-wise binary search works because each row is individually sorted (`0`s then `1`s), so the index of the first `1` can be found with a standard "find lower bound of 1" binary search: if `matrix[row, mid] == 1`, the first `1` could be at `mid` or earlier, so we record it and move `high = mid - 1`; otherwise we move `low = mid + 1`. Once we know the first `1`'s index for a row, the count of `1`s in that row is simply `cols - firstOneIndex`. Comparing this count across all rows (keeping the first row seen for ties, due to `>` not `>=`) gives the row with the maximum number of `1`s in `O(n log m)` instead of `O(n * m)`.

---

## 2. Search in a Row-wise and Column-wise Sorted Matrix

### 2. Search in a Row-wise and Column-wise Sorted Matrix

**Problem Statement:**
Given an `n x m` matrix where every row is sorted left to right in increasing order and every column is sorted top to bottom in increasing order (but the first element of a row is not necessarily greater than the last element of the previous row), determine whether a given `target` exists in the matrix. This is the classic "Search a 2D Matrix II" problem.

**Example:**
- Input: `matrix = [[1,4,7,11],[2,5,8,12],[3,6,9,16],[10,13,14,17]]`, `target = 5`
- Output: `true`
- Explanation: `5` is present at `matrix[1,1]`.

**Brute Force Approach:**
Simply scan every cell of the matrix and check if it equals the target.

```csharp
public bool SearchMatrixIIBrute(int[,] matrix, int target)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            if (matrix[i, j] == target)
            {
                return true;
            }
        }
    }

    return false;
}
```

Time Complexity: `O(n * m)` — every cell may be visited.
Space Complexity: `O(1)` — no extra space used.

**Optimized Approach:**
Start from the top-right corner (or bottom-left corner). At every step, if the current element equals the target, return `true`. If the current element is greater than the target, the entire column below it can be eliminated (move left). If it is smaller than the target, the entire row to its left can be eliminated (move down). This uses the sorted property in both dimensions to eliminate one row or one column per comparison.

```csharp
public bool SearchMatrixIIOptimized(int[,] matrix, int target)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int row = 0;
    int col = cols - 1; // start at top-right corner

    while (row < rows && col >= 0)
    {
        int current = matrix[row, col];

        if (current == target)
        {
            return true;
        }
        else if (current > target)
        {
            col--; // eliminate this column
        }
        else
        {
            row++; // eliminate this row
        }
    }

    return false;
}
```

Time Complexity: `O(n + m)` — in the worst case we move down `n` times and left `m` times, a total of at most `n + m` steps.
Space Complexity: `O(1)` — only pointers `row` and `col` are used.

**Explanation:**
Dry run on `matrix = [[1,4,7,11],[2,5,8,12],[3,6,9,16],[10,13,14,17]]`, `target = 5`:
- Start at `row = 0, col = 3` → `matrix[0,3] = 11`. `11 > 5`, so move left: `col = 2`.
- `matrix[0,2] = 7`. `7 > 5`, so move left: `col = 1`.
- `matrix[0,1] = 4`. `4 < 5`, so move down: `row = 1`.
- `matrix[1,1] = 5`. `5 == 5` → return `true`.

The technique works because starting at the top-right corner, the current cell is the largest in its row and the smallest in its column. If the current value is greater than the target, the target cannot be anywhere in the current column (since everything below is even larger), so we safely discard the whole column by decrementing `col`. If the current value is smaller than the target, the target cannot be anywhere in the current row (everything to the left is even smaller), so we safely discard the whole row by incrementing `row`. Each comparison eliminates exactly one row or one column, bounding the total work at `O(n + m)`.

---

## 3. Search in a Row-wise Sorted Matrix Where Each Row's First Element > Previous Row's Last Element

### 3. Search in a Row-wise Sorted Matrix Where Each Row's First Element > Previous Row's Last Element

**Problem Statement:**
Given an `n x m` matrix where each row is sorted in increasing order, and the first element of every row is strictly greater than the last element of the previous row, determine whether a given `target` exists in the matrix. Because of this property, the entire matrix can be treated as one flattened, fully sorted 1D array of size `n * m`. This is the classic "Search a 2D Matrix I" problem.

**Example:**
- Input: `matrix = [[1,3,5,7],[10,11,16,20],[23,30,34,60]]`, `target = 3`
- Output: `true`
- Explanation: Flattened, the matrix is `[1,3,5,7,10,11,16,20,23,30,34,60]`. `3` exists at flattened index `1`, which maps back to `matrix[0,1]`.

**Brute Force Approach:**
Scan every cell of the matrix (or every element of an equivalent flattened traversal) and check for equality with the target.

```csharp
public bool SearchMatrixIBrute(int[,] matrix, int target)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            if (matrix[i, j] == target)
            {
                return true;
            }
        }
    }

    return false;
}
```

Time Complexity: `O(n * m)` — every cell may be visited.
Space Complexity: `O(1)` — no extra space used.

**Optimized Approach:**
Since the whole matrix behaves like one sorted 1D array of size `n * m`, apply standard binary search on the virtual index range `[0, n*m - 1]`. Convert a virtual index `idx` back to matrix coordinates using `row = idx / cols` and `col = idx % cols`.

```csharp
public bool SearchMatrixIOptimized(int[,] matrix, int target)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int low = 0;
    int high = rows * cols - 1;

    while (low <= high)
    {
        int mid = low + (high - low) / 2;

        int row = mid / cols;
        int col = mid % cols;
        int midValue = matrix[row, col];

        if (midValue == target)
        {
            return true;
        }
        else if (midValue < target)
        {
            low = mid + 1;
        }
        else
        {
            high = mid - 1;
        }
    }

    return false;
}
```

Time Complexity: `O(log(n * m))` — a single binary search over `n * m` virtual elements.
Space Complexity: `O(1)` — only a constant number of extra variables are used.

**Explanation:**
Because each row's first element is strictly greater than the previous row's last element, and each row itself is sorted, reading the matrix row by row (left to right, top to bottom) produces one continuously increasing sequence. Any virtual index `idx` in `[0, n*m - 1]` maps uniquely to `matrix[idx / cols, idx % cols]` — dividing by the number of columns gives which row `idx` falls into, and the remainder gives the position within that row. Standard binary search is then applied directly on this virtual index range exactly as it would be on a plain 1D sorted array, giving `O(log(n*m))` time without ever materializing a flattened array.

---

## 4. Find Peak Element in a 2D Matrix

### 4. Find Peak Element in a 2D Matrix

**Problem Statement:**
A peak element in a 2D matrix is an element that is strictly greater than all of its (up to four) neighbors — the elements directly above, below, left, and right of it. Elements outside the matrix boundary are considered to be `-infinity`. Given a matrix of distinct integers, find the position `(row, col)` of any one peak element. It is guaranteed that a peak always exists.

**Example:**
- Input:
```
[[10, 20, 15],
 [21, 30, 14],
 [7,  16, 32]]
```
- Output: `(1, 1)` (value `30`)
- Explanation: `30`'s neighbors are `20` (up), `14` (right), `21` (left), `16` (down) — all smaller than `30`, so `(1,1)` is a peak. (`(2,2)` with value `32` is also a valid peak since its only neighbors are `16` and `14`.)

**Brute Force Approach:**
For every cell, compare it against its up/down/left/right neighbors (treating out-of-bounds as `-infinity`) and return the first cell for which all existing neighbors are smaller.

```csharp
public (int row, int col) FindPeakGridBrute(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            int current = matrix[i, j];

            int up = (i - 1 >= 0) ? matrix[i - 1, j] : int.MinValue;
            int down = (i + 1 < rows) ? matrix[i + 1, j] : int.MinValue;
            int left = (j - 1 >= 0) ? matrix[i, j - 1] : int.MinValue;
            int right = (j + 1 < cols) ? matrix[i, j + 1] : int.MinValue;

            if (current > up && current > down && current > left && current > right)
            {
                return (i, j);
            }
        }
    }

    return (-1, -1); // should not happen if a peak is guaranteed
}
```

Time Complexity: `O(n * m)` — every cell may be checked against its neighbors.
Space Complexity: `O(1)` — no extra space used.

**Optimized Approach:**
Binary search on columns. For a chosen middle column `mid`, find the row index of the maximum element in that column (`O(n)` scan). Compare that maximum element with its left neighbor (`mid - 1` column, same row) and right neighbor (`mid + 1` column, same row). If it is greater than both, it is a 2D peak. If the left neighbor is greater, a peak must exist in the left half of columns, so search `low = mid`'s left side (`high = mid - 1`). Otherwise a peak must exist in the right half, so search `low = mid + 1`. This halves the column search space each time, and each step does an `O(n)` column-max scan.

```csharp
public (int row, int col) FindPeakGridOptimized(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int low = 0;
    int high = cols - 1;

    while (low <= high)
    {
        int midCol = low + (high - low) / 2;

        int maxRowIndex = 0;
        int maxValueInCol = matrix[0, midCol];
        for (int i = 1; i < rows; i++)
        {
            if (matrix[i, midCol] > maxValueInCol)
            {
                maxValueInCol = matrix[i, midCol];
                maxRowIndex = i;
            }
        }

        int leftValue = (midCol - 1 >= 0) ? matrix[maxRowIndex, midCol - 1] : int.MinValue;
        int rightValue = (midCol + 1 < cols) ? matrix[maxRowIndex, midCol + 1] : int.MinValue;

        if (maxValueInCol > leftValue && maxValueInCol > rightValue)
        {
            return (maxRowIndex, midCol);
        }
        else if (leftValue > maxValueInCol)
        {
            high = midCol - 1; // peak exists in the left half
        }
        else
        {
            low = midCol + 1; // peak exists in the right half
        }
    }

    return (-1, -1); // should not happen if a peak is guaranteed
}
```

Time Complexity: `O(n * log m)` — binary search over `m` columns (`log m` steps), and each step scans a column of `n` rows to find its maximum.
Space Complexity: `O(1)` — only a constant number of extra variables are used.

**Explanation:**
The correctness relies on the same "always move toward the larger side" idea as the 1D peak-finding binary search. At column `midCol`, we find the row with the maximum value in that column; this element is already greater than its up/down neighbors by construction (it's the column max). We only need to check its left and right neighbors. If it beats both, it's a full 2D peak. If the left neighbor is larger, then moving further left the values only need to keep increasing or a peak will be found strictly to the left of `midCol` — because the left neighbor is bigger than the current column's maximum, so it is definitely bigger than everything else in the current column too, guaranteeing a peak exists somewhere in the left half. Symmetric logic applies to the right side. This guarantees discarding half the columns at each step, giving `O(log m)` column-search steps, each costing `O(n)` to find a column maximum, for `O(n log m)` overall.

---

## 5. Find the Median of a Row-wise Sorted Matrix

### 5. Find the Median of a Row-wise Sorted Matrix

**Problem Statement:**
Given an `n x m` matrix where every row is individually sorted in non-decreasing order (but columns are not necessarily sorted), and the total number of elements `n * m` is odd, find the median of all the elements in the matrix.

**Example:**
- Input:
```
[[1, 3, 5],
 [2, 6, 9],
 [3, 6, 9]]
```
- Output: `5`
- Explanation: All elements sorted: `[1,2,3,3,5,6,6,9,9]` (9 elements). The median (5th element, 1-indexed) is `5`.

**Brute Force Approach:**
Copy all `n * m` elements of the matrix into a single array, sort that array, and pick the middle element.

```csharp
public int MedianOfMatrixBrute(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);

    int[] allElements = new int[rows * cols];
    int index = 0;

    for (int i = 0; i < rows; i++)
    {
        for (int j = 0; j < cols; j++)
        {
            allElements[index] = matrix[i, j];
            index++;
        }
    }

    Array.Sort(allElements);

    return allElements[(rows * cols) / 2]; // middle element (0-indexed) since n*m is odd
}
```

Time Complexity: `O(n * m * log(n * m))` — dominated by sorting all elements.
Space Complexity: `O(n * m)` — extra array to hold and sort all elements.

**Optimized Approach:**
Binary search on the value range `[minElement, maxElement]` of the matrix. For a candidate value `mid`, count how many elements in the entire matrix are `<= mid` by binary searching (upper bound) within each row and summing the counts — this works because each row is individually sorted. If that count is less than or equal to `(n*m)/2`, the median must be greater than `mid`, so search the right half. Otherwise, the median could be `mid` or something smaller, so search the left half (record `mid` as a candidate answer). The smallest value for which the count of elements `<= mid` exceeds `(n*m)/2` is the median.

```csharp
public int MedianOfMatrixOptimized(int[,] matrix)
{
    int rows = matrix.GetLength(0);
    int cols = matrix.GetLength(1);
    int totalElements = rows * cols;
    int desiredRank = totalElements / 2; // 0-indexed target rank for the median

    int low = int.MaxValue;
    int high = int.MinValue;

    // Determine the overall min and max values in the matrix
    for (int i = 0; i < rows; i++)
    {
        low = Math.Min(low, matrix[i, 0]);           // rows sorted, so first element is row min
        high = Math.Max(high, matrix[i, cols - 1]);  // last element is row max
    }

    while (low < high)
    {
        int mid = low + (high - low) / 2;

        int countLessEqual = CountElementsLessEqual(matrix, mid, rows, cols);

        if (countLessEqual <= desiredRank)
        {
            low = mid + 1; // median must be larger
        }
        else
        {
            high = mid; // mid could be the answer, keep it in range
        }
    }

    return low;
}

// For each row, binary search (upper bound) for count of elements <= value
private int CountElementsLessEqual(int[,] matrix, int value, int rows, int cols)
{
    int count = 0;

    for (int i = 0; i < rows; i++)
    {
        int low = 0, high = cols - 1;
        int rowCount = 0;

        while (low <= high)
        {
            int mid = low + (high - low) / 2;

            if (matrix[i, mid] <= value)
            {
                rowCount = mid + 1; // all elements up to and including mid qualify
                low = mid + 1;
            }
            else
            {
                high = mid - 1;
            }
        }

        count += rowCount;
    }

    return count;
}
```

Time Complexity: `O(n * log m * log(maxValue - minValue))` — the outer binary search on the value range runs `log(maxValue - minValue)` times, and each iteration counts elements `<= mid` by binary searching each of the `n` rows in `O(log m)`.
Space Complexity: `O(1)` — no extra array is used, only a constant number of variables.

**Explanation:**
Dry run the value-range binary search on `matrix = [[1,3,5],[2,6,9],[3,6,9]]` (`rows = 3, cols = 3, total = 9, desiredRank = 9/2 = 4`):
- `low = min(1,2,3) = 1`, `high = max(5,9,9) = 9`.
- Iteration 1: `mid = 5`. Count of elements `<= 5`: row0 → `1,3,5` → 3; row1 → `2` → 1; row2 → `3` → 1. Total = `5`. Since `5 > desiredRank(4)`, set `high = mid = 5`.
- Iteration 2: `low=1, high=5`, `mid = 3`. Count `<= 3`: row0 → `1,3` → 2; row1 → `2` → 1; row2 → `3` → 1. Total = `4`. Since `4 <= desiredRank(4)`, set `low = mid + 1 = 4`.
- Iteration 3: `low=4, high=5`, `mid = 4`. Count `<= 4`: row0 → `1,3` → 2; row1 → `2` → 1; row2 → `3` → 1. Total = `4`. Since `4 <= 4`, set `low = 5`.
- Now `low == high == 5`, loop ends. Return `5`, matching the expected median.

This works because for a fully sorted array of `n*m` (odd) elements, the median is the element at 0-indexed position `desiredRank = (n*m)/2` — i.e., the smallest value `v` such that at least `desiredRank + 1` elements of the array are `<= v`. Counting "elements `<= mid`" across the matrix is a monotonic, non-decreasing function of `mid` (as `mid` increases, the count can only stay the same or grow), which is exactly the precondition needed for binary search on answer: if the count is `<= desiredRank`, `mid` is too small to be the median (there aren't enough elements at or below it yet), so we move `low` up; otherwise `mid` is large enough to be a valid candidate (possibly the tightest one), so we pull `high` down to `mid` instead of `mid - 1`, ensuring we never skip past the true smallest qualifying value. When `low` converges with `high`, that value is guaranteed to be present in the matrix and is the median, because the counting function's "crossing point" from `<= desiredRank` to `> desiredRank` occurs precisely at the median's value in an odd-sized dataset.
