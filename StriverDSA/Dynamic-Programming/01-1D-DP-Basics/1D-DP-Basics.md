# Dynamic Programming — 1D DP Basics

## Concept: Dynamic Programming

Dynamic Programming (DP) is a technique for solving problems that have two properties:

1. **Overlapping Subproblems** — the same smaller subproblem is solved again and again during recursion.
2. **Optimal Substructure** — the optimal solution to the problem can be built from optimal solutions of its subproblems.

Instead of recomputing the same subproblem repeatedly, DP stores ("remembers") the result the first time it is computed and reuses it later. There are three progressively more efficient ways to apply this idea. We'll illustrate all three using the classic **Fibonacci** example: `fib(n) = fib(n-1) + fib(n-2)`, with `fib(0) = 0` and `fib(1) = 1`.

### 1. Plain Recursion (no DP)

```
                fib(5)
              /        \
         fib(4)          fib(3)
        /      \         /     \
    fib(3)    fib(2)  fib(2)   fib(1)
    /    \    /   \    /   \
 fib(2) fib(1) fib(1) fib(0) fib(1) fib(0)
  /  \
fib(1) fib(0)
```

Notice `fib(3)` is computed twice, `fib(2)` three times, `fib(1)` five times — these are **overlapping subproblems**. Plain recursion recomputes each of them from scratch every time, giving exponential time complexity `O(2^n)`.

### 2. Memoization (Top-Down DP)

We keep the same recursive structure but add an array (or dictionary) `memo[]` that caches the result of each subproblem the first time it's solved. Before recursing, we check if the answer is already cached; if so, we return it instantly instead of recomputing.

```csharp
int Fib(int n, int[] memo)
{
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];
    return memo[n] = Fib(n - 1, memo) + Fib(n - 2, memo);
}
```

Now every subproblem `fib(0) … fib(n)` is computed exactly once — `O(n)` time — at the cost of `O(n)` extra memo space plus `O(n)` recursion stack space.

### 3. Tabulation (Bottom-Up DP)

Instead of recursing top-down, we build the answer iteratively from the base cases upward, filling a `dp[]` array in order.

```csharp
int Fib(int n)
{
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[0] = 0;
    dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}
```

This avoids recursion entirely (no call-stack overhead) while keeping `O(n)` time and `O(n)` space.

### 4. Space-Optimized Tabulation

Since `dp[i]` only ever depends on the previous two values `dp[i-1]` and `dp[i-2]`, we don't need to store the whole array — just two running variables.

```csharp
int Fib(int n)
{
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++)
    {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

This gives `O(n)` time with only `O(1)` extra space.

This four-stage progression — **recursion → memoization → tabulation → space optimization** — is the general recipe we will apply to every 1D DP problem below.

---

## 1. Climbing Stairs

**Problem Statement:** You are climbing a staircase with `n` steps. Each time you can climb either 1 or 2 steps. Count the number of distinct ways you can reach the top.

**Example:**
- Input: `n = 4`
- Output: `5`
- Explanation: The 5 ways are `1+1+1+1`, `1+1+2`, `1+2+1`, `2+1+1`, `2+2`.

**Approach 1 — Recursion:** At each step, either take 1 stair or 2 stairs, and sum the ways from both choices. Base cases: 0 steps left = 1 way, negative steps = 0 ways (invalid).

```csharp
public int ClimbStairsRecursive(int n)
{
    if (n == 0) return 1;
    if (n < 0) return 0;
    return ClimbStairsRecursive(n - 1) + ClimbStairsRecursive(n - 2);
}
```

Time Complexity: O(2^n), Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization (Top-Down DP):** Cache the number of ways for each `n` so it's computed only once.

```csharp
public int ClimbStairsMemo(int n)
{
    int[] memo = new int[n + 1];
    Array.Fill(memo, -1);
    return Solve(n, memo);
}

private int Solve(int n, int[] memo)
{
    if (n == 0) return 1;
    if (n < 0) return 0;
    if (memo[n] != -1) return memo[n];
    return memo[n] = Solve(n - 1, memo) + Solve(n - 2, memo);
}
```

Time Complexity: O(n), Space Complexity: O(n) + O(n) recursion stack.

**Approach 3 — Tabulation (Bottom-Up DP):** Build `dp[i]` = ways to reach step `i`, starting from `dp[0] = 1`.

```csharp
public int ClimbStairsTab(int n)
{
    if (n == 0) return 1;
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i - 1] + dp[i - 2];
    return dp[n];
}
```

Time Complexity: O(n), Space Complexity: O(n).

**Approach 4 — Space Optimization:** Only the previous two values are needed at any point.

```csharp
public int ClimbStairsOptimal(int n)
{
    if (n == 0) return 1;
    int prev2 = 1, prev1 = 1; // dp[0], dp[1]
    for (int i = 2; i <= n; i++)
    {
        int curr = prev1 + prev2;
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Explanation:** Climbing stairs is structurally identical to Fibonacci: `ways(n) = ways(n-1) + ways(n-2)`, only the base cases differ slightly. Every technique from the concept section above (recursion → memo → tabulation → O(1) space) applies directly.

---

## 2. Frog Jump

**Problem Statement:** A frog is on step `0` of `n` steps, with `height[i]` denoting the height of step `i`. From step `i`, the frog can jump to step `i+1` or step `i+2`, paying an energy cost equal to `|height[i] - height[j]|` for a jump to step `j`. Find the minimum total energy to reach step `n-1`.

**Example:**
- Input: `height = [10, 20, 30, 10]`
- Output: `20`
- Explanation: Jump `0 → 1` costs `|10-20|=10`, then `1 → 3` costs `|20-10|=10`, total `20`. This beats jumping `0→1→2→3` (`10+10+20=40`) or `0→2→3` (`20+20=40`).

**Approach 1 — Recursion:** From step `n-1`, the minimum cost is the smaller of coming from `n-2` or `n-3` (guarding index bounds), plus the corresponding jump cost.

```csharp
public int FrogJumpRecursive(int n, int[] height)
{
    return Solve(n - 1, height);
}

private int Solve(int i, int[] height)
{
    if (i == 0) return 0;
    int oneStep = Solve(i - 1, height) + Math.Abs(height[i] - height[i - 1]);
    int twoStep = int.MaxValue;
    if (i > 1)
        twoStep = Solve(i - 2, height) + Math.Abs(height[i] - height[i - 2]);
    return Math.Min(oneStep, twoStep);
}
```

Time Complexity: O(2^n), Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization (Top-Down DP):** Cache the minimum cost to reach each index.

```csharp
public int FrogJumpMemo(int n, int[] height)
{
    int[] memo = new int[n];
    Array.Fill(memo, -1);
    return Solve(n - 1, height, memo);
}

private int Solve(int i, int[] height, int[] memo)
{
    if (i == 0) return 0;
    if (memo[i] != -1) return memo[i];

    int oneStep = Solve(i - 1, height, memo) + Math.Abs(height[i] - height[i - 1]);
    int twoStep = int.MaxValue;
    if (i > 1)
        twoStep = Solve(i - 2, height, memo) + Math.Abs(height[i] - height[i - 2]);

    return memo[i] = Math.Min(oneStep, twoStep);
}
```

Time Complexity: O(n), Space Complexity: O(n) + O(n) recursion stack.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` = minimum cost to reach step `i`, starting from `dp[0] = 0`.

```csharp
public int FrogJumpTab(int n, int[] height)
{
    int[] dp = new int[n];
    dp[0] = 0;
    for (int i = 1; i < n; i++)
    {
        int oneStep = dp[i - 1] + Math.Abs(height[i] - height[i - 1]);
        int twoStep = int.MaxValue;
        if (i > 1)
            twoStep = dp[i - 2] + Math.Abs(height[i] - height[i - 2]);
        dp[i] = Math.Min(oneStep, twoStep);
    }
    return dp[n - 1];
}
```

Time Complexity: O(n), Space Complexity: O(n).

**Approach 4 — Space Optimization:** Only the previous two `dp` values are needed.

```csharp
public int FrogJumpOptimal(int n, int[] height)
{
    int prev2 = 0; // dp[i-2]
    int prev1 = 0; // dp[i-1], dp[0] = 0
    for (int i = 1; i < n; i++)
    {
        int oneStep = prev1 + Math.Abs(height[i] - height[i - 1]);
        int twoStep = int.MaxValue;
        if (i > 1)
            twoStep = prev2 + Math.Abs(height[i] - height[i - 2]);

        int curr = Math.Min(oneStep, twoStep);
        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Explanation — Dry run of Tabulation:** Take `height = [10, 20, 30, 10]` (`n = 4`).

| i | height[i] | oneStep = dp[i-1] + \|h[i]-h[i-1]\| | twoStep = dp[i-2] + \|h[i]-h[i-2]\| | dp[i] = min |
|---|-----------|--------------------------------------|--------------------------------------|-------------|
| 0 | 10        | — (base case)                         | —                                      | `dp[0] = 0` |
| 1 | 20        | `dp[0] + \|20-10\| = 0 + 10 = 10`     | not applicable (i ≤ 1)                 | `dp[1] = 10` |
| 2 | 30        | `dp[1] + \|30-20\| = 10 + 10 = 20`    | `dp[0] + \|30-10\| = 0 + 20 = 20`      | `dp[2] = 20` |
| 3 | 10        | `dp[2] + \|10-30\| = 20 + 20 = 40`    | `dp[1] + \|10-20\| = 10 + 10 = 20`     | `dp[3] = 20` |

Final answer: `dp[3] = 20`, matching the expected output — reached via `0 → 1 → 3` (jump of 2 from step 1 to step 3).

---

## 3. Frog Jump with K Distances

**Problem Statement:** Same setup as Frog Jump, but the frog standing on step `i` can jump to any step from `i+1` up to `i+k` (a jump of `1` to `k` steps), paying cost `|height[i] - height[j]|` for that jump. Find the minimum total energy to reach step `n-1`.

**Example:**
- Input: `height = [10, 30, 40, 50, 20], k = 3`
- Output: `30`
- Explanation: Jump `0 → 4` directly is not allowed (distance 4 > k=3). Best path is `0 → 1 → 4`: cost `|10-30| + |30-20| = 20 + 10 = 30`. (Direct `0 → 3 → 4` costs `40 + 30 = 70`, worse.)

**Approach 1 — Recursion:** From step `i`, try all jumps back by `1..k` and take the minimum.

```csharp
public int FrogJumpKRecursive(int n, int k, int[] height)
{
    return Solve(n - 1, k, height);
}

private int Solve(int i, int k, int[] height)
{
    if (i == 0) return 0;
    int minCost = int.MaxValue;
    for (int j = 1; j <= k; j++)
    {
        if (i - j >= 0)
        {
            int cost = Solve(i - j, k, height) + Math.Abs(height[i] - height[i - j]);
            minCost = Math.Min(minCost, cost);
        }
    }
    return minCost;
}
```

Time Complexity: O(k^n), Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization (Top-Down DP):** Cache the minimum cost to reach each index.

```csharp
public int FrogJumpKMemo(int n, int k, int[] height)
{
    int[] memo = new int[n];
    Array.Fill(memo, -1);
    return Solve(n - 1, k, height, memo);
}

private int Solve(int i, int k, int[] height, int[] memo)
{
    if (i == 0) return 0;
    if (memo[i] != -1) return memo[i];

    int minCost = int.MaxValue;
    for (int j = 1; j <= k; j++)
    {
        if (i - j >= 0)
        {
            int cost = Solve(i - j, k, height, memo) + Math.Abs(height[i] - height[i - j]);
            minCost = Math.Min(minCost, cost);
        }
    }
    return memo[i] = minCost;
}
```

Time Complexity: O(n * k), Space Complexity: O(n) + O(n) recursion stack.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` by looking back up to `k` steps.

```csharp
public int FrogJumpKTab(int n, int k, int[] height)
{
    int[] dp = new int[n];
    dp[0] = 0;
    for (int i = 1; i < n; i++)
    {
        int minCost = int.MaxValue;
        for (int j = 1; j <= k; j++)
        {
            if (i - j >= 0)
            {
                int cost = dp[i - j] + Math.Abs(height[i] - height[i - j]);
                minCost = Math.Min(minCost, cost);
            }
        }
        dp[i] = minCost;
    }
    return dp[n - 1];
}
```

Time Complexity: O(n * k), Space Complexity: O(n).

**Approach 4 — Space Optimization:** We only need the last `k` computed values, so a sliding window of size `k` (a small array/deque) replaces the full `dp` array. This is not O(1) in general (it's O(k)), but it's the natural bound-reduction analogous to the two-variable trick when `k` is fixed.

```csharp
public int FrogJumpKOptimal(int n, int k, int[] height)
{
    // Sliding window of the last k dp values.
    int[] window = new int[k];
    window[0] = 0; // dp[0]

    for (int i = 1; i < n; i++)
    {
        int minCost = int.MaxValue;
        for (int j = 1; j <= k; j++)
        {
            if (i - j >= 0)
            {
                int prevValue = window[(i - j) % k];
                int cost = prevValue + Math.Abs(height[i] - height[i - j]);
                minCost = Math.Min(minCost, cost);
            }
        }
        window[i % k] = minCost;
    }
    return window[(n - 1) % k];
}
```

Time Complexity: O(n * k), Space Complexity: O(k).

**Explanation:** This problem generalizes Frog Jump (which is the special case `k = 2`). Instead of only checking `i-1` and `i-2`, we check all of `i-1 … i-k` and take the minimum cost among them. The recursion, memoization, and tabulation patterns are unchanged in structure — only the inner loop widens from a fixed 2 branches to `k` branches, which is why time complexity picks up a factor of `k`.

---

## 4. Maximum Sum of Non-Adjacent Elements (House Robber)

**Problem Statement:** Given an array of non-negative integers representing money in each house along a street, find the maximum sum of a subsequence such that no two chosen elements are adjacent in the array (a robber cannot rob two adjacent houses, as it triggers an alarm).

**Example:**
- Input: `nums = [2, 7, 9, 3, 1]`
- Output: `12`
- Explanation: Rob houses at index `0`, `2`, `4`: `2 + 9 + 1 = 12`. This beats robbing `1` and `3`: `7 + 3 = 10`.

**Approach 1 — Recursion:** At index `i`, either "take" `nums[i]` and skip to `i-2`, or "skip" it and move to `i-1`; take the max of both choices.

```csharp
public int RobRecursive(int[] nums)
{
    return Solve(nums.Length - 1, nums);
}

private int Solve(int i, int[] nums)
{
    if (i < 0) return 0;
    if (i == 0) return nums[0];

    int take = nums[i] + Solve(i - 2, nums);
    int skip = 0 + Solve(i - 1, nums);
    return Math.Max(take, skip);
}
```

Time Complexity: O(2^n), Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization (Top-Down DP):** Cache the best result for each index.

```csharp
public int RobMemo(int[] nums)
{
    int n = nums.Length;
    int[] memo = new int[n];
    Array.Fill(memo, -1);
    return Solve(n - 1, nums, memo);
}

private int Solve(int i, int[] nums, int[] memo)
{
    if (i < 0) return 0;
    if (i == 0) return nums[0];
    if (memo[i] != -1) return memo[i];

    int take = nums[i] + Solve(i - 2, nums, memo);
    int skip = 0 + Solve(i - 1, nums, memo);
    return memo[i] = Math.Max(take, skip);
}
```

Time Complexity: O(n), Space Complexity: O(n) + O(n) recursion stack.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` = best sum achievable using houses `0..i`.

```csharp
public int RobTab(int[] nums)
{
    int n = nums.Length;
    if (n == 0) return 0;

    int[] dp = new int[n];
    dp[0] = nums[0];
    for (int i = 1; i < n; i++)
    {
        int take = nums[i] + (i > 1 ? dp[i - 2] : 0);
        int skip = dp[i - 1];
        dp[i] = Math.Max(take, skip);
    }
    return dp[n - 1];
}
```

Time Complexity: O(n), Space Complexity: O(n).

**Approach 4 — Space Optimization:** Only the previous two `dp` values are needed.

```csharp
public int RobOptimal(int[] nums)
{
    int n = nums.Length;
    if (n == 0) return 0;

    int prev2 = 0;         // dp[i-2]
    int prev1 = nums[0];   // dp[i-1], dp[0] = nums[0]

    for (int i = 1; i < n; i++)
    {
        int take = nums[i] + prev2;
        int skip = prev1;
        int curr = Math.Max(take, skip);

        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Explanation:** The "take or skip" choice at every index makes this structurally similar to Fibonacci/Climbing Stairs but with a `max` combinator instead of a `sum`, and an extra `+nums[i]` payoff for the "take" branch. This is why the same recursion → memo → tabulation → O(1) space progression applies cleanly.

---

## 5. House Robber II

**Problem Statement:** Same as House Robber, but the houses are arranged in a circle — the first and last houses are now adjacent to each other. Find the maximum sum of non-adjacent house values under this circular constraint.

**Example:**
- Input: `nums = [2, 3, 2]`
- Output: `3`
- Explanation: Because house `0` and house `2` are adjacent (circular), we cannot rob both. Robbing only house `1` gives `3`, which is better than robbing houses `0` and `2` (`2+2=4` is invalid since they're adjacent) — actually best valid choice is just house `1` = `3`, since `0` and `2` can't both be picked and picking either alone (`2`) is less than `3`.

**Approach 1 — Recursion:** Split into two linear subproblems — houses `[0 .. n-2]` (excluding the last house) and houses `[1 .. n-1]` (excluding the first house) — solve each with plain linear House Robber recursion, and return the max of the two results.

```csharp
public int RobCircularRecursive(int[] nums)
{
    int n = nums.Length;
    if (n == 1) return nums[0];

    int excludeLast = SolveRange(0, n - 2, nums);
    int excludeFirst = SolveRange(1, n - 1, nums);
    return Math.Max(excludeLast, excludeFirst);
}

private int SolveRange(int start, int end, int[] nums)
{
    return Solve(end, start, nums);
}

private int Solve(int i, int start, int[] nums)
{
    if (i < start) return 0;
    if (i == start) return nums[i];

    int take = nums[i] + Solve(i - 2, start, nums);
    int skip = 0 + Solve(i - 1, start, nums);
    return Math.Max(take, skip);
}
```

Time Complexity: O(2^n), Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization (Top-Down DP):** Run the memoized linear House Robber logic on each of the two ranges (each with its own memo array), and take the max.

```csharp
public int RobCircularMemo(int[] nums)
{
    int n = nums.Length;
    if (n == 1) return nums[0];

    int[] memo1 = new int[n];
    int[] memo2 = new int[n];
    Array.Fill(memo1, -1);
    Array.Fill(memo2, -1);

    int excludeLast = Solve(n - 2, 0, nums, memo1);
    int excludeFirst = Solve(n - 1, 1, nums, memo2);
    return Math.Max(excludeLast, excludeFirst);
}

private int Solve(int i, int start, int[] nums, int[] memo)
{
    if (i < start) return 0;
    if (i == start) return nums[i];
    if (memo[i] != -1) return memo[i];

    int take = nums[i] + Solve(i - 2, start, nums, memo);
    int skip = 0 + Solve(i - 1, start, nums, memo);
    return memo[i] = Math.Max(take, skip);
}
```

Time Complexity: O(n), Space Complexity: O(n) + O(n) recursion stack.

**Approach 3 — Tabulation (Bottom-Up DP):** Run the linear tabulated House Robber on `nums[0 .. n-2]` and again on `nums[1 .. n-1]`; return the max.

```csharp
public int RobCircularTab(int[] nums)
{
    int n = nums.Length;
    if (n == 1) return nums[0];

    int excludeLast = RobLinearTab(nums, 0, n - 2);
    int excludeFirst = RobLinearTab(nums, 1, n - 1);
    return Math.Max(excludeLast, excludeFirst);
}

private int RobLinearTab(int[] nums, int start, int end)
{
    int len = end - start + 1;
    int[] dp = new int[len];
    dp[0] = nums[start];
    for (int i = 1; i < len; i++)
    {
        int take = nums[start + i] + (i > 1 ? dp[i - 2] : 0);
        int skip = dp[i - 1];
        dp[i] = Math.Max(take, skip);
    }
    return dp[len - 1];
}
```

Time Complexity: O(n), Space Complexity: O(n).

**Approach 4 — Space Optimization:** Run the O(1)-space linear House Robber twice (once per range) and take the max.

```csharp
public int RobCircularOptimal(int[] nums)
{
    int n = nums.Length;
    if (n == 1) return nums[0];

    int excludeLast = RobLinearOptimal(nums, 0, n - 2);
    int excludeFirst = RobLinearOptimal(nums, 1, n - 1);
    return Math.Max(excludeLast, excludeFirst);
}

private int RobLinearOptimal(int[] nums, int start, int end)
{
    int prev2 = 0;
    int prev1 = nums[start];

    for (int i = start + 1; i <= end; i++)
    {
        int take = nums[i] + prev2;
        int skip = prev1;
        int curr = Math.Max(take, skip);

        prev2 = prev1;
        prev1 = curr;
    }
    return prev1;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Explanation — The circular trick:** In House Robber II, houses `0` and `n-1` are adjacent, so a valid robbery plan can never include both. This means the optimal answer must fall into one of two mutually exclusive scenarios:

1. **House `n-1` is excluded entirely** — solve plain linear House Robber on the subarray `nums[0 .. n-2]`.
2. **House `0` is excluded entirely** — solve plain linear House Robber on the subarray `nums[1 .. n-1]`.

Since at least one of house `0` or house `n-1` must be left out of any valid selection, considering both scenarios and taking the maximum of the two guarantees we've covered every valid circular arrangement — no case is missed, and no invalid adjacent pair is ever considered. This reduces the circular problem to two independent calls of the linear House Robber solution developed in problem 4, reusing all four approaches (recursion, memoization, tabulation, and O(1)-space) unchanged, just invoked twice on two different ranges.
