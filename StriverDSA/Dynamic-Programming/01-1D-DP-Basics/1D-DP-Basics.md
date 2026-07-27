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

**Logic (Steps):**
1. Define the state: `f(n)` = number of distinct ways to climb `n` remaining steps.
2. Base cases: `f(0) = 1` (one way — do nothing, you're already at the top), `f(n) = 0` for `n < 0` (invalid, overshot).
3. Recurrence/transition: `f(n) = f(n-1) + f(n-2)` — take 1 stair then solve the rest, or take 2 stairs then solve the rest.
4. No iteration order needed yet since this is plain top-down recursion; each call branches into two smaller calls until a base case is hit.

```csharp
public int ClimbStairsRecursive(int n)
{
    if (n == 0) return 1;
    if (n < 0) return 0;
    return ClimbStairsRecursive(n - 1) + ClimbStairsRecursive(n - 2);
}
```

Time Complexity: O(2^n), Space Complexity: O(n) recursion stack.

**Walkthrough:** For `n = 4`: `f(4) = f(3) + f(2)`. Expanding, `f(3) = f(2)+f(1) = 2+1 = 3`, `f(2) = f(1)+f(0) = 1+1 = 2`. So `f(4) = 3 + 2 = 5`, matching the expected output `5`.

**Approach 2 — Memoization (Top-Down DP):** Cache the number of ways for each `n` so it's computed only once.

**Logic (Steps):**
1. State: `memo[n]` stores the already-computed value of `f(n)` once solved, initialized to `-1` (unsolved).
2. Base cases are unchanged: `n == 0` returns `1`, `n < 0` returns `0`.
3. Before recursing, check `memo[n]`; if already computed, return it immediately instead of recomputing.
4. Otherwise compute `Solve(n-1) + Solve(n-2)`, store it in `memo[n]`, and return it — guaranteeing each `n` is computed only once.

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

**Walkthrough:** For `n = 4`: `Solve(4)` calls `Solve(3)` and `Solve(2)`. `Solve(2)` computes `Solve(1)+Solve(0) = 1+1 = 2`, cached as `memo[2]=2`. `Solve(3)` computes `Solve(2)+Solve(1)`; `Solve(2)` is now cached so it returns `2` instantly, giving `memo[3] = 2+1 = 3`. Finally `Solve(4) = memo[3] + memo[2] = 3 + 2 = 5`, matching the expected output `5`.

**Approach 3 — Tabulation (Bottom-Up DP):** Build `dp[i]` = ways to reach step `i`, starting from `dp[0] = 1`.

**Logic (Steps):**
1. State: `dp[i]` = number of ways to reach step `i`.
2. Base cases: `dp[0] = 1`, `dp[1] = 1`.
3. Transition: `dp[i] = dp[i-1] + dp[i-2]` for `i >= 2`.
4. Iteration order: fill `dp` left to right from `i = 2` to `n`, since each entry depends only on the two entries before it.

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

**Walkthrough:** For `n = 4`: `dp[0]=1, dp[1]=1, dp[2]=dp[1]+dp[0]=2, dp[3]=dp[2]+dp[1]=3, dp[4]=dp[3]+dp[2]=5`. Returns `dp[4] = 5`, matching the expected output `5`.

**Approach 4 — Space Optimization:** Only the previous two values are needed at any point.

**Logic (Steps):**
1. State: replace the `dp` array with two rolling variables, `prev2` (= `dp[i-2]`) and `prev1` (= `dp[i-1]`), initialized to `dp[0]=1` and `dp[1]=1`.
2. Transition: at each step compute `curr = prev1 + prev2`.
3. Iteration order: loop `i` from `2` to `n`, after computing `curr` shift the window forward (`prev2 = prev1`, `prev1 = curr`).
4. After the loop, `prev1` holds `dp[n]`, the final answer.

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

**Walkthrough:** For `n = 4`: start `prev2=1, prev1=1`. `i=2: curr=2, prev2=1, prev1=2`. `i=3: curr=3, prev2=2, prev1=3`. `i=4: curr=5, prev2=3, prev1=5`. Loop ends, return `prev1 = 5`, matching the expected output `5`.

---

## 2. Frog Jump

**Problem Statement:** A frog is on step `0` of `n` steps, with `height[i]` denoting the height of step `i`. From step `i`, the frog can jump to step `i+1` or step `i+2`, paying an energy cost equal to `|height[i] - height[j]|` for a jump to step `j`. Find the minimum total energy to reach step `n-1`.

**Example:**
- Input: `height = [10, 20, 30, 10]`
- Output: `20`
- Explanation: Jump `0 → 1` costs `|10-20|=10`, then `1 → 3` costs `|20-10|=10`, total `20`. This beats jumping `0→1→2→3` (`10+10+20=40`) or `0→2→3` (`20+20=40`).

**Approach 1 — Recursion:** From step `n-1`, the minimum cost is the smaller of coming from `n-2` or `n-3` (guarding index bounds), plus the corresponding jump cost.

**Logic (Steps):**
1. State: `Solve(i)` = minimum energy cost to reach step `i` from step `0`.
2. Base case: `Solve(0) = 0` (already there, no cost).
3. Transition: `Solve(i) = min(Solve(i-1) + |height[i]-height[i-1]|, Solve(i-2) + |height[i]-height[i-2]|)`, guarding the two-step option when `i <= 1`.
4. The answer is `Solve(n-1)`, found by recursing from the last index down to the base case.

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

**Walkthrough:** For `height = [10, 20, 30, 10]`, `n = 4`: `Solve(3)` compares `Solve(2)+|10-30|` and `Solve(1)+|10-20|`. Expanding, `Solve(1) = Solve(0)+|20-10| = 10`, and `Solve(2) = min(Solve(1)+|30-20|, Solve(0)+|30-10|) = min(20, 20) = 20`. So `Solve(3) = min(20+20, 10+10) = min(40, 20) = 20`, matching the expected output `20`.

**Approach 2 — Memoization (Top-Down DP):** Cache the minimum cost to reach each index.

**Logic (Steps):**
1. State: `memo[i]` caches `Solve(i)` once computed, initialized to `-1`.
2. Base case unchanged: `Solve(0) = 0`.
3. Before computing, check `memo[i]`; if set, return it directly.
4. Otherwise compute the same `oneStep`/`twoStep` transition as the recursive approach, store the result in `memo[i]`, and return it — each index is now solved only once.

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

**Walkthrough:** For `height = [10, 20, 30, 10]`: `Solve(3)` triggers `Solve(2)` which triggers `Solve(1)`. `memo[1]=10`, `memo[2]=min(10+10, 0+20)=20`. Back in `Solve(3)`: `oneStep = memo[2]+20 = 40`, `twoStep = memo[1]+10 = 20`, so `memo[3] = 20`, matching the expected output `20`.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` = minimum cost to reach step `i`, starting from `dp[0] = 0`.

**Logic (Steps):**
1. State: `dp[i]` = minimum cost to reach step `i`.
2. Base case: `dp[0] = 0`.
3. Transition: for each `i` from `1` to `n-1`, `dp[i] = min(dp[i-1] + |height[i]-height[i-1]|, dp[i-2] + |height[i]-height[i-2]|)` (the second term only when `i > 1`).
4. Iteration order: left to right, since `dp[i]` only depends on the two entries before it. The answer is `dp[n-1]`.

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

**Walkthrough:** For `height = [10, 20, 30, 10]` (`n = 4`): `dp[0]=0`; `dp[1] = dp[0]+|20-10| = 10`; `dp[2] = min(dp[1]+|30-20|, dp[0]+|30-10|) = min(20,20) = 20`; `dp[3] = min(dp[2]+|10-30|, dp[1]+|10-20|) = min(40,20) = 20`. Final answer `dp[3] = 20`, matching the expected output — reached via `0 → 1 → 3`.

**Approach 4 — Space Optimization:** Only the previous two `dp` values are needed.

**Logic (Steps):**
1. State: replace `dp[]` with `prev2` (= `dp[i-2]`) and `prev1` (= `dp[i-1]`), both starting at `0` (`dp[0] = 0`).
2. Transition: at each `i`, compute `oneStep = prev1 + |height[i]-height[i-1]|` and `twoStep = prev2 + |height[i]-height[i-2]|` (when `i > 1`), then `curr = min(oneStep, twoStep)`.
3. Iteration order: loop `i` from `1` to `n-1`, shifting the window (`prev2 = prev1; prev1 = curr`) after each step.
4. After the loop, `prev1` holds `dp[n-1]`, the final answer.

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

**Walkthrough:** For `height = [10, 20, 30, 10]`: start `prev2=0, prev1=0`. `i=1: oneStep=10, curr=10, prev2=0, prev1=10`. `i=2: oneStep=20, twoStep=0+20=20, curr=20, prev2=10, prev1=20`. `i=3: oneStep=20+20=40, twoStep=10+10=20, curr=20, prev1=20`. Loop ends, return `20`, matching the expected output.

---

## 3. Frog Jump with K Distances

**Problem Statement:** Same setup as Frog Jump, but the frog standing on step `i` can jump to any step from `i+1` up to `i+k` (a jump of `1` to `k` steps), paying cost `|height[i] - height[j]|` for that jump. Find the minimum total energy to reach step `n-1`.

**Example:**
- Input: `height = [10, 30, 40, 50, 20], k = 3`
- Output: `30`
- Explanation: Jump `0 → 4` directly is not allowed (distance 4 > k=3). Best path is `0 → 1 → 4`: cost `|10-30| + |30-20| = 20 + 10 = 30`. (Direct `0 → 3 → 4` costs `40 + 30 = 70`, worse.)

**Approach 1 — Recursion:** From step `i`, try all jumps back by `1..k` and take the minimum.

**Logic (Steps):**
1. State: `Solve(i)` = minimum energy to reach step `i` from step `0`.
2. Base case: `Solve(0) = 0`.
3. Transition: `Solve(i) = min over j in [1..k], i-j>=0 of Solve(i-j) + |height[i]-height[i-j]|` — try every allowed jump distance and keep the cheapest.
4. The answer is `Solve(n-1)`.

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

**Walkthrough:** For `height = [10, 30, 40, 50, 20], k = 3`: `Solve(4)` tries `j=1,2,3`, i.e. `Solve(3)+|20-50|`, `Solve(2)+|20-40|`, `Solve(1)+|20-30|`. Expanding `Solve(1) = Solve(0)+|30-10| = 20`, so the `j=3` branch gives `20+10=30`. Working out the other branches yields larger costs, so the minimum is `30`, matching the expected output `30`.

**Approach 2 — Memoization (Top-Down DP):** Cache the minimum cost to reach each index.

**Logic (Steps):**
1. State: `memo[i]` caches `Solve(i)` once computed.
2. Base case: `Solve(0) = 0`.
3. Before computing, check `memo[i]`; if set, return it. Otherwise loop `j` from `1` to `k` (bounds-checked), taking the minimum of `Solve(i-j) + |height[i]-height[i-j]|`.
4. Store the result in `memo[i]` before returning, so each index is solved once.

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

**Walkthrough:** For `height = [10, 30, 40, 50, 20], k = 3`: `Solve(4)` needs `Solve(1), Solve(2), Solve(3)`, each computed once and cached (`memo[1]=20`, etc.). Reusing cached values, `Solve(4)` evaluates the same three branches as the recursive version and returns `30`, matching the expected output, but without recomputation across overlapping subcalls.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` by looking back up to `k` steps.

**Logic (Steps):**
1. State: `dp[i]` = minimum cost to reach step `i`.
2. Base case: `dp[0] = 0`.
3. Transition: for each `i` from `1` to `n-1`, loop `j` from `1` to `k` (bounds-checked) and take `dp[i] = min(dp[i-j] + |height[i]-height[i-j]|)`.
4. Iteration order: left to right, since each `dp[i]` only depends on up to `k` earlier entries. Answer is `dp[n-1]`.

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

**Walkthrough:** For `height = [10, 30, 40, 50, 20], k = 3`: `dp[0]=0`; `dp[1]=dp[0]+20=20`; `dp[2]=min(dp[1]+10, dp[0]+30)=min(30,30)=30`; `dp[3]=min(dp[2]+10, dp[1]+20, dp[0]+40)=min(40,40,40)=40`; `dp[4]=min(dp[3]+30, dp[2]+20, dp[1]+10)=min(70,50,30)=30`. Final answer `dp[4]=30`, matching the expected output.

**Approach 4 — Space Optimization:** We only need the last `k` computed values, so a sliding window of size `k` (a small array/deque) replaces the full `dp` array. This is not O(1) in general (it's O(k)), but it's the natural bound-reduction analogous to the two-variable trick when `k` is fixed.

**Logic (Steps):**
1. State: `window[idx]` holds `dp` values for the last `k` indices, addressed via `idx = i % k` (a circular buffer).
2. Base case: `window[0] = 0` represents `dp[0]`.
3. Transition: for each `i` from `1` to `n-1`, loop `j` from `1` to `k`, reading `dp[i-j]` as `window[(i-j) % k]`, and take the minimum cost as before.
4. Iteration order: left to right; write the result to `window[i % k]`, overwriting the entry that has fallen more than `k` steps behind. The answer is `window[(n-1) % k]`.

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

**Walkthrough:** For `height = [10, 30, 40, 50, 20], k = 3`: `window` cycles through indices `0,1,2` as `i` increases. `window[0]=0` (dp[0]), `window[1]=20` (dp[1]), `window[2]=30` (dp[2]), then `i=3` writes `window[0]=40` (dp[3], overwriting old dp[0]), `i=4` writes `window[1]=30` (dp[4]). The answer `window[(4)%3] = window[1] = 30` matches the expected output.

---

## 4. Maximum Sum of Non-Adjacent Elements (House Robber)

**Problem Statement:** Given an array of non-negative integers representing money in each house along a street, find the maximum sum of a subsequence such that no two chosen elements are adjacent in the array (a robber cannot rob two adjacent houses, as it triggers an alarm).

**Example:**
- Input: `nums = [2, 7, 9, 3, 1]`
- Output: `12`
- Explanation: Rob houses at index `0`, `2`, `4`: `2 + 9 + 1 = 12`. This beats robbing `1` and `3`: `7 + 3 = 10`.

**Approach 1 — Recursion:** At index `i`, either "take" `nums[i]` and skip to `i-2`, or "skip" it and move to `i-1`; take the max of both choices.

**Logic (Steps):**
1. State: `Solve(i)` = maximum robbable sum considering houses `0..i`.
2. Base cases: `Solve(-1) = 0` (no houses), `Solve(0) = nums[0]` (only one house, must take it).
3. Transition: `Solve(i) = max(nums[i] + Solve(i-2), Solve(i-1))` — either rob house `i` and add to the best from `i-2` (skipping the adjacent `i-1`), or skip house `i` entirely and take the best from `i-1`.
4. The answer is `Solve(n-1)`.

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

**Walkthrough:** For `nums = [2, 7, 9, 3, 1]`: `Solve(4)` compares `take=1+Solve(2)` vs `skip=Solve(3)`. Expanding, `Solve(2)=max(9+Solve(0), Solve(1))=max(9+2,7)=11`, so `take=1+11=12`. `Solve(3)=max(3+Solve(1), Solve(2))=max(3+7,11)=11`, so `skip=11`. `Solve(4)=max(12,11)=12`, matching the expected output `12`.

**Approach 2 — Memoization (Top-Down DP):** Cache the best result for each index.

**Logic (Steps):**
1. State: `memo[i]` caches `Solve(i)` once computed.
2. Base cases unchanged: `i < 0` returns `0`, `i == 0` returns `nums[0]`.
3. Before computing, check `memo[i]`; if set, return it immediately. Otherwise compute `take`/`skip` exactly as in the recursive version.
4. Store the result in `memo[i]` before returning.

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

**Walkthrough:** For `nums = [2, 7, 9, 3, 1]`: `Solve(4)` triggers `Solve(2)` and `Solve(3)`, which both need `Solve(1)` — computed once and cached as `memo[1]=7`. Using cached values, `memo[2]=11`, `memo[3]=11`, and `Solve(4)=max(1+11, 11)=12`, matching the expected output `12`, with `Solve(1)` never recomputed.

**Approach 3 — Tabulation (Bottom-Up DP):** Fill `dp[i]` = best sum achievable using houses `0..i`.

**Logic (Steps):**
1. State: `dp[i]` = maximum robbable sum using houses `0..i`.
2. Base case: `dp[0] = nums[0]`.
3. Transition: for `i` from `1` to `n-1`, `dp[i] = max(nums[i] + (i>1 ? dp[i-2] : 0), dp[i-1])`.
4. Iteration order: left to right, since `dp[i]` depends only on `dp[i-1]` and `dp[i-2]`. Answer is `dp[n-1]`.

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

**Walkthrough:** For `nums = [2, 7, 9, 3, 1]`: `dp[0]=2`; `dp[1]=max(7,2)=7`; `dp[2]=max(9+2,7)=11`; `dp[3]=max(3+7,11)=11`; `dp[4]=max(1+11,11)=12`. Final answer `dp[4]=12`, matching the expected output `12`.

**Approach 4 — Space Optimization:** Only the previous two `dp` values are needed.

**Logic (Steps):**
1. State: replace `dp[]` with `prev2` (= `dp[i-2]`, starting at `0`) and `prev1` (= `dp[i-1]`, starting at `dp[0]=nums[0]`).
2. Transition: at each `i`, compute `take = nums[i] + prev2`, `skip = prev1`, `curr = max(take, skip)`.
3. Iteration order: loop `i` from `1` to `n-1`, shifting the window (`prev2 = prev1; prev1 = curr`) after each step.
4. After the loop, `prev1` holds `dp[n-1]`, the final answer.

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

**Walkthrough:** For `nums = [2, 7, 9, 3, 1]`: start `prev2=0, prev1=2`. `i=1: curr=max(7,2)=7, prev2=2, prev1=7`. `i=2: curr=max(9+2,7)=11, prev2=7, prev1=11`. `i=3: curr=max(3+7,11)=11, prev2=11, prev1=11`. `i=4: curr=max(1+11,11)=12, prev1=12`. Loop ends, return `12`, matching the expected output.

---

## 5. House Robber II

**Problem Statement:** Same as House Robber, but the houses are arranged in a circle — the first and last houses are now adjacent to each other. Find the maximum sum of non-adjacent house values under this circular constraint.

**Example:**
- Input: `nums = [2, 3, 2]`
- Output: `3`
- Explanation: Because house `0` and house `2` are adjacent (circular), we cannot rob both. Robbing only house `1` gives `3`, which is better than robbing houses `0` and `2` (`2+2=4` is invalid since they're adjacent) — actually best valid choice is just house `1` = `3`, since `0` and `2` can't both be picked and picking either alone (`2`) is less than `3`.

**Approach 1 — Recursion:** Split into two linear subproblems — houses `[0 .. n-2]` (excluding the last house) and houses `[1 .. n-1]` (excluding the first house) — solve each with plain linear House Robber recursion, and return the max of the two results.

**Logic (Steps):**
1. State: `Solve(i, start)` = maximum robbable sum considering houses `start..i` (same recurrence as linear House Robber, just parameterized by a starting index).
2. Base cases: `Solve(start-1, start) = 0`, `Solve(start, start) = nums[start]`.
3. Transition: `Solve(i, start) = max(nums[i] + Solve(i-2, start), Solve(i-1, start))`.
4. Compute the answer as `max(SolveRange(0, n-2), SolveRange(1, n-1))` — since houses `0` and `n-1` are adjacent in the circle, at least one of these two ranges must exclude the "wrap-around" conflict, so the true optimum is the better of the two.

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

**Walkthrough:** For `nums = [2, 3, 2]`: `excludeLast = SolveRange(0, 1)` solves linear House Robber on `[2, 3]` → `max(3, 2) = 3`. `excludeFirst = SolveRange(1, 2)` solves linear House Robber on `[3, 2]` → `max(2, 3) = 3`. Result `max(3, 3) = 3`, matching the expected output `3`.

**Approach 2 — Memoization (Top-Down DP):** Run the memoized linear House Robber logic on each of the two ranges (each with its own memo array), and take the max.

**Logic (Steps):**
1. State: `memo1[i]` and `memo2[i]` cache `Solve(i, start)` for the two ranges independently (they must be separate arrays since the two ranges have different `start` values and results).
2. Base cases and transition are identical to the linear House Robber memoization.
3. Compute `excludeLast = Solve(n-2, 0, nums, memo1)` and `excludeFirst = Solve(n-1, 1, nums, memo2)`.
4. Return `max(excludeLast, excludeFirst)`.

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

**Walkthrough:** For `nums = [2, 3, 2]`: `Solve(1, 0, memo1)` computes the linear answer on `[2, 3]` = `3`. `Solve(2, 1, memo2)` computes the linear answer on `[3, 2]` = `3`. `max(3, 3) = 3`, matching the expected output `3`, with each subproblem inside each range solved only once thanks to its own memo array.

**Approach 3 — Tabulation (Bottom-Up DP):** Run the linear tabulated House Robber on `nums[0 .. n-2]` and again on `nums[1 .. n-1]`; return the max.

**Logic (Steps):**
1. State: `RobLinearTab(nums, start, end)` builds a local `dp[]` over the subrange `start..end`, exactly like linear House Robber tabulation, using `dp[0] = nums[start]` as the base case.
2. Transition: `dp[i] = max(nums[start+i] + (i>1 ? dp[i-2] : 0), dp[i-1])`.
3. Iteration order: left to right within each subrange.
4. Call `RobLinearTab` once on `[0, n-2]` and once on `[1, n-1]`, and return the larger of the two results.

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

**Walkthrough:** For `nums = [2, 3, 2]`: `RobLinearTab(nums, 0, 1)` builds `dp[0]=2, dp[1]=max(3,2)=3` → returns `3`. `RobLinearTab(nums, 1, 2)` builds `dp[0]=3, dp[1]=max(2,3)=3` → returns `3`. `max(3, 3) = 3`, matching the expected output `3`.

**Approach 4 — Space Optimization:** Run the O(1)-space linear House Robber twice (once per range) and take the max.

**Logic (Steps):**
1. State: `RobLinearOptimal(nums, start, end)` uses the same two-rolling-variable trick as linear House Robber (`prev2`, `prev1`) restricted to the subrange `start..end`.
2. Base case: `prev1 = nums[start]`, `prev2 = 0`.
3. Transition: for `i` from `start+1` to `end`, `curr = max(nums[i]+prev2, prev1)`, then shift the window.
4. Call it once on `[0, n-2]` and once on `[1, n-1]`, returning the max of the two — same O(1)-space benefit as linear House Robber, applied twice.

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

**Walkthrough:** For `nums = [2, 3, 2]`: `RobLinearOptimal(nums, 0, 1)` starts `prev1=2, prev2=0`, then `i=1: curr=max(3+0,2)=3` → returns `3`. `RobLinearOptimal(nums, 1, 2)` starts `prev1=3, prev2=0`, then `i=2: curr=max(2+0,3)=3` → returns `3`. `max(3, 3) = 3`, matching the expected output `3`.

Because houses `0` and `n-1` are adjacent in the circle, a valid robbery plan can never include both — so the optimum always falls into one of two mutually exclusive scenarios: exclude house `n-1` and solve linearly on `nums[0..n-2]`, or exclude house `0` and solve linearly on `nums[1..n-1]`. Taking the max of both guarantees every valid circular arrangement is covered, reusing the linear House Robber solution from problem 4 unchanged, just invoked twice.
