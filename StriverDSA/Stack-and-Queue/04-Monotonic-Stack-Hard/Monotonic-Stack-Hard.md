# Stack and Queue — Monotonic Stack Problems (Hard)

## 1. Trapping Rain Water

### 1. Trapping Rain Water

**Problem Statement:**
Given `n` non-negative integers representing an elevation map where the width of each bar is 1, compute how much water it can trap after raining. Water trapped above a bar at index `i` is determined by `min(maxLeft, maxRight) - height[i]`, where `maxLeft` and `maxRight` are the tallest bars to the left and right of (and including) index `i`. If this value is negative, no water is trapped at that index.

**Example:**
- Input: `height = [0,1,0,2,1,0,1,3,2,1,2,1]`
- Output: `6`
- Explanation: Water accumulates above indices 2, 4, 5, 6, 8, 9 (the "valleys" between taller bars). Summing the trapped amounts at every index gives a total of 6 units of water.

**Brute Force Approach:**
For every index `i`, scan all elements to its left to find `maxLeft` and all elements to its right to find `maxRight`. The trapped water at `i` is `min(maxLeft, maxRight) - height[i]` (clamped at 0). This requires two nested scans for every index.

```csharp
public class Solution {
    public int TrapBruteForce(int[] height) {
        int n = height.Length;
        int totalWater = 0;

        for (int i = 0; i < n; i++) {
            int maxLeft = 0;
            for (int left = 0; left <= i; left++) {
                maxLeft = Math.Max(maxLeft, height[left]);
            }

            int maxRight = 0;
            for (int right = i; right < n; right++) {
                maxRight = Math.Max(maxRight, height[right]);
            }

            int waterAtI = Math.Min(maxLeft, maxRight) - height[i];
            if (waterAtI > 0) {
                totalWater += waterAtI;
            }
        }

        return totalWater;
    }
}
```

Time Complexity: O(n^2) — for each of the n indices we do an O(n) scan left and right.
Space Complexity: O(1) — no extra data structures used.

**Optimized Approach:**
Use the two-pointer technique with running `leftMax` and `rightMax` values instead of prefix/suffix arrays (which would take O(n) extra space). Start `left = 0` and `right = n - 1`. At each step, move the pointer whose side has the smaller current height, because the water level on that side is bounded by the smaller of the two maxes. This avoids ever needing to know the exact max on the far side.

```csharp
public class Solution {
    public int Trap(int[] height) {
        int n = height.Length;
        if (n == 0) return 0;

        int left = 0, right = n - 1;
        int leftMax = 0, rightMax = 0;
        int totalWater = 0;

        while (left < right) {
            if (height[left] <= height[right]) {
                leftMax = Math.Max(leftMax, height[left]);
                totalWater += leftMax - height[left];
                left++;
            } else {
                rightMax = Math.Max(rightMax, height[right]);
                totalWater += rightMax - height[right];
                right--;
            }
        }

        return totalWater;
    }
}
```

Time Complexity: O(n) — each pointer traverses the array once, and the two pointers together cover all n elements exactly once.
Space Complexity: O(1) — only a handful of scalar variables (`left`, `right`, `leftMax`, `rightMax`) are used; no auxiliary arrays.

**Explanation:**
The two-pointer trick works because whichever side currently has the smaller height is guaranteed to have its water level capped by `leftMax` (or `rightMax`) on that side — the taller bar on the opposite side can never be the limiting factor. For example, if `height[left] <= height[right]`, then even if there is an even taller bar somewhere between `left` and `right` on the right side, it doesn't matter: the water above index `left` is still bounded by `min(leftMax, someRightSideHeight)`, and since `height[right] >= height[left]` and `rightMax` (whatever it ends up being) is at least `height[right]`, we know `rightMax >= height[left]`. So `leftMax` alone determines the trapped water at `left`. This lets us safely advance the `left` pointer using only `leftMax`, without ever computing the true `rightMax` for that position. The pointers converge from both ends, each index processed exactly once.

## 2. Largest Rectangle in a Histogram

### 2. Largest Rectangle in a Histogram

**Problem Statement:**
Given an array `heights` representing the heights of bars in a histogram where each bar has width 1, find the area of the largest rectangle that can be formed within the histogram's bounds.

**Example:**
- Input: `heights = [2,1,5,6,2,3]`
- Output: `10`
- Explanation: The largest rectangle has height 5 and spans indices 2 to 3 (bars of height 5 and 6), giving area `5 * 2 = 10`. (Alternative: height 2 spanning indices 2-5 gives 2*4=8, which is smaller.)

**Brute Force Approach:**
For every bar `i`, expand left and right as far as possible while the neighboring bars are `>= heights[i]`. This determines the maximal width for a rectangle of height `heights[i]`. Track the maximum area found.

```csharp
public class Solution {
    public int LargestRectangleAreaBruteForce(int[] heights) {
        int n = heights.Length;
        int maxArea = 0;

        for (int i = 0; i < n; i++) {
            int left = i;
            while (left > 0 && heights[left - 1] >= heights[i]) {
                left--;
            }

            int right = i;
            while (right < n - 1 && heights[right + 1] >= heights[i]) {
                right++;
            }

            int width = right - left + 1;
            int area = heights[i] * width;
            maxArea = Math.Max(maxArea, area);
        }

        return maxArea;
    }
}
```

Time Complexity: O(n^2) — worst case (e.g., strictly increasing or all-equal heights) each bar's left/right expansion can scan almost the entire array.
Space Complexity: O(1) — no extra data structures beyond scalars.

**Optimized Approach:**
Use a monotonic increasing stack that stores indices of bars whose heights are in non-decreasing order. When we encounter a bar shorter than the bar at the top of the stack, we know the top bar can no longer extend further right, so we pop it and compute the maximal rectangle using that popped bar's height, with the width determined by the current index and the new stack top (the previous smaller element).

```csharp
public class Solution {
    public int LargestRectangleArea(int[] heights) {
        int n = heights.Length;
        Stack<int> stack = new Stack<int>(); // stores indices, heights[] increasing
        int maxArea = 0;

        for (int i = 0; i <= n; i++) {
            // Use height 0 as a sentinel when i == n to flush remaining stack
            int currentHeight = (i == n) ? 0 : heights[i];

            while (stack.Count > 0 && heights[stack.Peek()] >= currentHeight) {
                int height = heights[stack.Pop()];
                int width = stack.Count == 0 ? i : i - stack.Peek() - 1;
                maxArea = Math.Max(maxArea, height * width);
            }

            stack.Push(i);
        }

        return maxArea;
    }
}
```

Time Complexity: O(n) — each index is pushed onto the stack exactly once and popped at most once, giving amortized O(1) work per index.
Space Complexity: O(n) — in the worst case (strictly increasing heights) the stack holds all n indices before any pops occur.

**Explanation (Dry run on `[2,1,5,6,2,3]`):**

Stack holds indices; we track `heights[stack.Peek()]` for comparisons. Sentinel height 0 appended at `i = 6`.

- `i=0`, height=2: stack empty, push 0. Stack: `[0]`
- `i=1`, height=1: `heights[0]=2 >= 1`, pop 0 → height=2, stack empty so width=`i`=1, area=`2*1=2`. maxArea=2. Stack empty, push 1. Stack: `[1]`
- `i=2`, height=5: `heights[1]=1 < 5`, no pop. Push 2. Stack: `[1,2]`
- `i=3`, height=6: `heights[2]=5 < 6`, no pop. Push 3. Stack: `[1,2,3]`
- `i=4`, height=2: `heights[3]=6 >= 2`, pop 3 → height=6, width=`i - stack.Peek() - 1 = 4-2-1=1`, area=`6*1=6`. maxArea=6.
  Continue: `heights[2]=5 >= 2`, pop 2 → height=5, width=`4-1-1=2`, area=`5*2=10`. maxArea=10.
  Continue: `heights[1]=1 < 2`, stop popping. Push 4. Stack: `[1,4]`
- `i=5`, height=3: `heights[4]=2 < 3`, no pop. Push 5. Stack: `[1,4,5]`
- `i=6`, height=0 (sentinel): pop 5 → height=3, width=`6-4-1=1`, area=`3*1=3`. maxArea stays 10.
  pop 4 → height=2, width=`6-1-1=4`, area=`2*4=8`. maxArea stays 10.
  pop 1 → height=1, stack empty so width=`i`=6, area=`1*6=6`. maxArea stays 10.
  Stack empty, push 6.

Final `maxArea = 10`, matching the expected output (the rectangle formed by bars of height 5 and 6 at indices 2-3).

## 3. Maximal Rectangle in a Binary Matrix

### 3. Maximal Rectangle in a Binary Matrix

**Problem Statement:**
Given a binary matrix `matrix` filled with `0`s and `1`s, find the area of the largest rectangle containing only `1`s.

**Example:**
- Input:
```
matrix = [
  ['1','0','1','0','0'],
  ['1','0','1','1','1'],
  ['1','1','1','1','1'],
  ['1','0','0','1','0']
]
```
- Output: `6`
- Explanation: The largest rectangle of all 1s is formed using rows 1-2 and columns 2-3 (a 2x3... actually a 3x2 block), yielding area 6. Specifically, columns 2-3 across rows 0-2 form a solid rectangle of 1s with area `3*2=6`.

**Brute Force Approach:**
For every possible pair of rows (top boundary) and every possible pair of columns (left/right boundary), or more simply, for every top-left corner and every bottom-right corner, check whether the sub-rectangle formed consists entirely of 1s, and track the maximum area. This is extremely expensive.

```csharp
public class Solution {
    public int MaximalRectangleBruteForce(char[][] matrix) {
        if (matrix.Length == 0 || matrix[0].Length == 0) return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int maxArea = 0;

        for (int r1 = 0; r1 < rows; r1++) {
            for (int c1 = 0; c1 < cols; c1++) {
                if (matrix[r1][c1] == '0') continue;

                for (int r2 = r1; r2 < rows; r2++) {
                    for (int c2 = c1; c2 < cols; c2++) {
                        bool allOnes = true;

                        for (int r = r1; r <= r2 && allOnes; r++) {
                            for (int c = c1; c <= c2 && allOnes; c++) {
                                if (matrix[r][c] == '0') {
                                    allOnes = false;
                                }
                            }
                        }

                        if (allOnes) {
                            int area = (r2 - r1 + 1) * (c2 - c1 + 1);
                            maxArea = Math.Max(maxArea, area);
                        }
                    }
                }
            }
        }

        return maxArea;
    }
}
```

Time Complexity: O(rows^3 * cols^3) in the worst case — choosing all pairs of corners is O(rows^2 * cols^2), and verifying each candidate rectangle costs up to O(rows * cols). This is impractically slow for anything beyond tiny inputs.
Space Complexity: O(1) — no extra storage beyond scalars.

**Optimized Approach:**
Reduce the 2D problem to `rows` applications of the 1D "Largest Rectangle in a Histogram" problem. Maintain a `heights` array of length `cols`. Process the matrix row by row: for each row, update `heights[c]` to be `heights[c] + 1` if `matrix[row][c] == '1'`, or reset to `0` if `matrix[row][c] == '0'` (since a 0 breaks any vertical bar of 1s at that column). After updating `heights` for the current row, run the O(n) "Largest Rectangle in Histogram" algorithm on it — the result represents the largest all-1s rectangle whose bottom edge is the current row. Track the maximum across all rows.

```csharp
public class Solution {
    public int MaximalRectangle(char[][] matrix) {
        if (matrix.Length == 0 || matrix[0].Length == 0) return 0;

        int rows = matrix.Length;
        int cols = matrix[0].Length;
        int[] heights = new int[cols];
        int maxArea = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                heights[c] = (matrix[r][c] == '1') ? heights[c] + 1 : 0;
            }

            maxArea = Math.Max(maxArea, LargestRectangleArea(heights));
        }

        return maxArea;
    }

    private int LargestRectangleArea(int[] heights) {
        int n = heights.Length;
        Stack<int> stack = new Stack<int>();
        int maxArea = 0;

        for (int i = 0; i <= n; i++) {
            int currentHeight = (i == n) ? 0 : heights[i];

            while (stack.Count > 0 && heights[stack.Peek()] >= currentHeight) {
                int height = heights[stack.Pop()];
                int width = stack.Count == 0 ? i : i - stack.Peek() - 1;
                maxArea = Math.Max(maxArea, height * width);
            }

            stack.Push(i);
        }

        return maxArea;
    }
}
```

Time Complexity: O(rows * cols) — building/updating the `heights` array for each row takes O(cols), and running the O(cols) histogram algorithm on each of the `rows` rows gives a total of O(rows * cols).
Space Complexity: O(cols) — for the `heights` array and the stack used inside the histogram helper (both bounded by the number of columns).

**Explanation:**
Each row of the binary matrix is treated as the "ground level" for a histogram, where `heights[c]` represents how many consecutive 1s are stacked vertically above (and including) row `r` at column `c`. As we move down row by row, this histogram evolves: a `1` extends the bar upward by one, and a `0` collapses the bar back to height 0 (since a 0 cell blocks any rectangle from extending through it). By running "Largest Rectangle in Histogram" on the `heights` array after processing each row, we find the largest all-1s rectangle whose bottom edge is exactly that row — because a rectangle in histogram terms (contiguous columns, bounded by the minimum height among them) corresponds precisely to a solid block of 1s in the matrix bounded below by the current row. Repeating this for every row and taking the overall maximum guarantees we consider every possible bottom edge, so the true maximal all-1s rectangle is found.

## 4. Sliding Window Maximum

### 4. Sliding Window Maximum

**Problem Statement:**
Given an array `nums` and an integer `k`, there is a sliding window of size `k` that moves from the very left of the array to the very right, one position at a time. For each window position, return the maximum element in that window.

**Example:**
- Input: `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`
- Output: `[3,3,5,5,6,7]`
- Explanation: Window `[1,3,-1]` → max 3; `[3,-1,-3]` → max 3; `[-1,-3,5]` → max 5; `[-3,5,3]` → max 5; `[5,3,6]` → max 6; `[3,6,7]` → max 7.

**Brute Force Approach:**
For each window starting position, scan all `k` elements in that window to find the maximum.

```csharp
public class Solution {
    public int[] MaxSlidingWindowBruteForce(int[] nums, int k) {
        int n = nums.Length;
        int[] result = new int[n - k + 1];

        for (int i = 0; i <= n - k; i++) {
            int windowMax = int.MinValue;
            for (int j = i; j < i + k; j++) {
                windowMax = Math.Max(windowMax, nums[j]);
            }
            result[i] = windowMax;
        }

        return result;
    }
}
```

Time Complexity: O(n * k) — there are `n - k + 1` windows, and each requires an O(k) scan to find the max.
Space Complexity: O(1) extra (excluding the output array).

**Optimized Approach:**
Use a monotonic decreasing deque (double-ended queue) that stores indices, where the corresponding values are always in decreasing order from front to back. The front of the deque always holds the index of the current window's maximum. For each new element: (1) remove indices from the back while their values are less than the current value (they can never be the max while the current element is still in the window), (2) remove the front index if it has fallen out of the window's left bound, (3) push the current index to the back, and (4) once the first window is complete, record the value at the front index as the max for that window.

```csharp
public class Solution {
    public int[] MaxSlidingWindow(int[] nums, int k) {
        int n = nums.Length;
        int[] result = new int[n - k + 1];
        LinkedList<int> deque = new LinkedList<int>(); // stores indices, values decreasing front->back

        for (int i = 0; i < n; i++) {
            // Remove indices that are out of the current window from the front
            if (deque.Count > 0 && deque.First.Value <= i - k) {
                deque.RemoveFirst();
            }

            // Remove indices from the back whose values are smaller than nums[i]
            while (deque.Count > 0 && nums[deque.Last.Value] < nums[i]) {
                deque.RemoveLast();
            }

            deque.AddLast(i);

            // Record the maximum once the window has reached size k
            if (i >= k - 1) {
                result[i - k + 1] = nums[deque.First.Value];
            }
        }

        return result;
    }
}
```

Time Complexity: O(n) — each index is added to the deque exactly once and removed at most once (from either front or back), giving amortized O(1) work per element.
Space Complexity: O(k) — the deque holds at most `k` indices at any time (it never grows unbounded because out-of-window and dominated indices are continuously evicted).

**Explanation (Dry run on `nums = [1,3,-1,-3,5,3,6,7]`, `k = 3`):**

Deque stores indices; values shown in brackets for clarity.

- `i=0`, val=1: deque empty. Push 0. Deque: `[0(1)]`
- `i=1`, val=3: front(0) not out of window. Back check: `nums[0]=1 < 3` → evict 0 from back. Deque empty, push 1. Deque: `[1(3)]`
- `i=2`, val=-1: front(1) not out of window. Back check: `nums[1]=3 < -1`? No, so no eviction. Push 2. Deque: `[1(3), 2(-1)]`. Window complete (i>=2): result[0] = `nums[front=1] = 3`. ✓ matches expected 3.
- `i=3`, val=-3: front check: `deque.First=1 <= i-k=0`? No (1>0). Back check: `nums[2]=-1 < -3`? No. Push 3. Deque: `[1(3), 2(-1), 3(-3)]`. result[1] = `nums[1] = 3`. ✓ matches expected 3.
- `i=4`, val=5: front check: `deque.First=1 <= i-k=1`? Yes → evict front 1. Deque: `[2(-1), 3(-3)]`. Back check: `nums[3]=-3 < 5` → evict 3; `nums[2]=-1 < 5` → evict 2. Deque empty, push 4. Deque: `[4(5)]`. result[2] = `nums[4] = 5`. ✓ matches expected 5.
- `i=5`, val=3: front check: `4 <= i-k=2`? No. Back check: `nums[4]=5 < 3`? No. Push 5. Deque: `[4(5), 5(3)]`. result[3] = `nums[4] = 5`. ✓ matches expected 5.
- `i=6`, val=6: front check: `4 <= i-k=3`? No. Back check: `nums[5]=3 < 6` → evict 5; deque now `[4(5)]`, `nums[4]=5 < 6` → evict 4. Deque empty, push 6. Deque: `[6(6)]`. result[4] = `nums[6] = 6`. ✓ matches expected 6.
- `i=7`, val=7: front check: `6 <= i-k=4`? No. Back check: `nums[6]=6 < 7` → evict 6. Deque empty, push 7. Deque: `[7(7)]`. result[5] = `nums[7] = 7`. ✓ matches expected 7.

Final result: `[3,3,5,5,6,7]`, matching the expected output exactly.

## 5. Maximum of Minimums of Every Window Size

### 5. Maximum of Minimums of Every Window Size

**Problem Statement:**
Given an array `arr` of size `n`, find the maximum of the minimum values for every possible window size from `1` to `n`. Specifically, for each window size `k` (from 1 to n), consider all contiguous subarrays of length `k`, compute the minimum of each such subarray, and take the maximum of those minimums. Return an array `answer` of size `n` where `answer[k-1]` is this maximum-of-minimums value for window size `k`.

**Example:**
- Input: `arr = [10, 20, 30, 50, 10, 70, 30]`
- Output: `[70, 30, 20, 20, 20, 10, 10]`
- Explanation: For window size 1, the best single element is 70 (max of all elements). For window size 2, the best window is `[20,30]` or similar with min 20... actually `[70]` alone doesn't apply; checking `[50,10]`→10, `[10,70]`→10, `[70,30]`→30 — best is 30. For window size 3, minimums include `[30,50,10]`→10, `[10,70,30]`→10, best across all windows of size 3 is 20 from `[20,30,50]`→20. This pattern continues, with larger windows generally producing smaller or equal max-of-min values as window size grows.

**Brute Force Approach:**
For every window size `k` from 1 to n, slide a window of that size across the array, compute the minimum of each window (via a nested scan), and track the maximum such minimum for that `k`.

```csharp
public class Solution {
    public int[] MaxOfMinBruteForce(int[] arr) {
        int n = arr.Length;
        int[] answer = new int[n + 1]; // 1-indexed by window size for clarity

        for (int k = 1; k <= n; k++) {
            int maxOfMin = int.MinValue;

            for (int i = 0; i <= n - k; i++) {
                int windowMin = int.MaxValue;
                for (int j = i; j < i + k; j++) {
                    windowMin = Math.Min(windowMin, arr[j]);
                }
                maxOfMin = Math.Max(maxOfMin, windowMin);
            }

            answer[k] = maxOfMin;
        }

        int[] result = new int[n];
        Array.Copy(answer, 1, result, 0, n);
        return result;
    }
}
```

Time Complexity: O(n^3) — O(n) choices of `k`, times O(n) window start positions, times O(n) to compute each window's minimum.
Space Complexity: O(n) for the answer array.

**Optimized Approach:**
Reuse the "previous smaller element" and "next smaller element" boundaries computed via a monotonic stack (the same technique used in Largest Rectangle in Histogram). For each index `i`, let `left[i]` be the distance to the nearest strictly smaller element on the left (or `i` itself if none exists), and `right[i]` be the distance to the nearest strictly smaller element on the right (or `n - i - 1` if none exists). Then `windowLength = left[i] + right[i] + 1` is the size of the largest window in which `arr[i]` is the minimum. Update `answer[windowLength]` with `arr[i]` if it's larger than the current stored value (since `arr[i]` is a valid minimum for a window of that exact length). Finally, fill gaps: `answer[k]` should be at least `answer[k+1]`, because any minimum achievable with a window of size `k+1` is also achievable (or bettered) with a smaller window size `k` — so propagate values from larger `k` down to smaller `k`.

```csharp
public class Solution {
    public int[] MaxOfMin(int[] arr) {
        int n = arr.Length;
        int[] left = new int[n];  // distance to previous smaller element
        int[] right = new int[n]; // distance to next smaller element
        Stack<int> stack = new Stack<int>();

        // Compute "distance to previous smaller element" for each index
        for (int i = 0; i < n; i++) {
            while (stack.Count > 0 && arr[stack.Peek()] >= arr[i]) {
                stack.Pop();
            }
            left[i] = (stack.Count == 0) ? i : (i - stack.Peek() - 1);
            stack.Push(i);
        }

        stack.Clear();

        // Compute "distance to next smaller element" for each index
        for (int i = n - 1; i >= 0; i--) {
            while (stack.Count > 0 && arr[stack.Peek()] >= arr[i]) {
                stack.Pop();
            }
            right[i] = (stack.Count == 0) ? (n - i - 1) : (stack.Peek() - i - 1);
            stack.Push(i);
        }

        // answer[k] (1-indexed) = best (max) minimum achievable with window size k
        int[] answer = new int[n + 1];
        for (int k = 1; k <= n; k++) {
            answer[k] = 0;
        }

        for (int i = 0; i < n; i++) {
            int windowLength = left[i] + right[i] + 1;
            answer[windowLength] = Math.Max(answer[windowLength], arr[i]);
        }

        // Propagate: a smaller window can always match or beat a larger window's best minimum
        for (int k = n - 1; k >= 1; k--) {
            answer[k] = Math.Max(answer[k], answer[k + 1]);
        }

        int[] result = new int[n];
        Array.Copy(answer, 1, result, 0, n);
        return result;
    }
}
```

Time Complexity: O(n) — computing `left[]` and `right[]` each takes O(n) using a monotonic stack (each index pushed/popped once), and the subsequent fill/propagation loops are each O(n).
Space Complexity: O(n) — for the `left`, `right`, `answer` arrays and the stack.

**Explanation:**
This problem directly reuses the "previous smaller element" / "next smaller element" boundary computation that underlies Largest Rectangle in Histogram. In that problem, for each bar we found how far it could extend left and right before hitting a shorter bar — that same span is exactly the largest window in which the current element is the minimum value. If `arr[i]` can extend `left[i]` steps left and `right[i]` steps right before hitting a smaller element on either side, then `arr[i]` is the minimum of any window of length up to `left[i] + right[i] + 1` centered appropriately around it, and it is *the* minimum for the window of exactly that maximal length. We record `arr[i]` as a candidate answer for `windowLength = left[i] + right[i] + 1`. Since we want the best (maximum) minimum for each window size, we take the max over all elements that map to the same window length. Finally, because a window of size `k` can always be found inside (or matching) any valid configuration for size `k+1` that achieves a certain minimum, `answer[k]` must be at least `answer[k+1]` — so we propagate values backward from larger window sizes to smaller ones to fill any window sizes that had no element directly mapping to them. This entire process runs in O(n), a massive improvement over the O(n^3) brute force.
