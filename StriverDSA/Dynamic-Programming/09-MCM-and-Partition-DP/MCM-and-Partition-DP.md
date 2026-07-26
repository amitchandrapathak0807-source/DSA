# Dynamic Programming — Matrix Chain Multiplication and Partition DP

## Concept: Partition DP (MCM Pattern)

Partition DP is a recursion/DP pattern that operates over a **range** `[i, j]` (of a matrix-dimension array, a string, an array of numbers, etc.) instead of a single index. The core idea:

1. Define a state `dp(i, j)` = the answer (min cost, max value, count, etc.) for the sub-problem restricted to the range `[i, j]`.
2. To compute `dp(i, j)`, try **every possible partition point `k`** inside (or bounding) the range that splits it into a left part and a right part.
3. For each `k`, recursively solve the left sub-range and the right sub-range, then **combine** the two results with some cost/value associated with performing the split at `k`.
4. Take the best (min/max/sum of counts) over all choices of `k`.

In pseudocode:

```text
dp(i, j):
    if range [i, j] is a base case: return base value
    best = identity (∞ for min, -∞ for max, 0 for count, ...)
    for k in partitionPointsBetween(i, j):
        left  = dp(i, k)
        right = dp(k+1, j)      // or dp(k, j), depending on problem
        cost  = combine(left, right, k)
        best  = better(best, cost)
    return best
```

This is exactly the structure used to solve **Matrix Chain Multiplication** (partition the chain of matrices at the last multiplication performed), but the same skeleton — "loop over a partition point inside a range-based recursion" — solves a much wider family of problems: cutting a stick optimally, bursting balloons, evaluating boolean expressions, counting palindrome-partition cuts, and more. What changes between problems is only:

- What the range represents (matrix indices, string indices, array indices, cut positions).
- What "combine" means (sum of multiplication costs, sum of two products, multiplying counts, etc.).
- Whether we're minimizing, maximizing, or counting.

Because `dp(i, j)` depends on smaller ranges, these problems are naturally solved with recursion → memoization (top-down) → tabulation (bottom-up, iterating range length from small to large).

---

## 1. Matrix Chain Multiplication

**Problem Statement:** Given an array `arr` of size `n+1` representing the dimensions of `n` matrices, where the `i`-th matrix has dimensions `arr[i-1] x arr[i]`, find the minimum number of scalar multiplications needed to multiply the entire chain of matrices together. Matrix multiplication is associative, so the order in which we multiply matters for cost but not for the final result.

**Example:**
- Input: `arr = [40, 20, 30, 10, 30]`
- Output: `26000`
- Explanation: There are 4 matrices: `A1 (40x20)`, `A2 (20x30)`, `A3 (30x10)`, `A4 (10x30)`. Multiplying them in the order `((A1(A2A3))A4)` costs 26000 scalar multiplications, which is the minimum over all possible parenthesizations.

**Approach 1 — Recursion:** the range `[i, j]` recursion trying every partition point.

```csharp
public class MatrixChainRecursive
{
    public int MatrixChainOrder(int[] arr)
    {
        int n = arr.Length;
        // i and j index into arr; matrices considered are (i..j)
        return Solve(arr, 1, n - 1);
    }

    private int Solve(int[] arr, int i, int j)
    {
        if (i >= j) return 0; // 0 or 1 matrix, no multiplication needed

        int minCost = int.MaxValue;
        for (int k = i; k < j; k++)
        {
            int cost = Solve(arr, i, k) + Solve(arr, k + 1, j)
                       + arr[i - 1] * arr[k] * arr[j];
            minCost = Math.Min(minCost, cost);
        }
        return minCost;
    }
}
```

Time Complexity: O(2^n) (exponential — every partition explored without reuse); Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class MatrixChainMemo
{
    public int MatrixChainOrder(int[] arr)
    {
        int n = arr.Length;
        int[,] memo = new int[n, n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                memo[i, j] = -1;

        return Solve(arr, 1, n - 1, memo);
    }

    private int Solve(int[] arr, int i, int j, int[,] memo)
    {
        if (i >= j) return 0;
        if (memo[i, j] != -1) return memo[i, j];

        int minCost = int.MaxValue;
        for (int k = i; k < j; k++)
        {
            int cost = Solve(arr, i, k, memo) + Solve(arr, k + 1, j, memo)
                       + arr[i - 1] * arr[k] * arr[j];
            minCost = Math.Min(minCost, cost);
        }
        return memo[i, j] = minCost;
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

**Approach 3 — Tabulation:** iterating range length from small to large.

```csharp
public class MatrixChainTabulation
{
    public int MatrixChainOrder(int[] arr)
    {
        int n = arr.Length;
        int[,] dp = new int[n, n];

        for (int len = 2; len <= n - 1; len++)
        {
            for (int i = 1; i <= n - len; i++)
            {
                int j = i + len - 1;
                dp[i, j] = int.MaxValue;
                for (int k = i; k < j; k++)
                {
                    int cost = dp[i, k] + dp[k + 1, j]
                               + arr[i - 1] * arr[k] * arr[j];
                    dp[i, j] = Math.Min(dp[i, j], cost);
                }
            }
        }
        return dp[1, n - 1];
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

---

## 2. Minimum Cost to Cut a Stick

**Problem Statement:** Given a stick of length `n` and an array `cuts` containing positions at which cuts must be made, find the minimum total cost to perform all the cuts in some order. The cost of a single cut is the length of the stick segment being cut at that moment. You may choose the order of cuts to minimize total cost.

**Example:**
- Input: `n = 7`, `cuts = [1, 3, 4, 5]`
- Output: `16`
- Explanation: Cutting in order `3, 5, 1, 4` gives costs `7 + 4 + 3 + 2 = 16`, which is optimal. Cutting at 3 first costs 7 (full stick), then at 5 costs 4 (segment 3–7), etc.

**Approach 1 — Recursion:** the range `[i, j]` recursion trying every partition point.

```csharp
public class StickCutRecursive
{
    public int MinCost(int n, int[] cuts)
    {
        int m = cuts.Length;
        int[] positions = new int[m + 2];
        positions[0] = 0;
        positions[m + 1] = n;
        Array.Copy(cuts, 0, positions, 1, m);
        Array.Sort(positions);

        return Solve(positions, 1, m);
    }

    // Solve cuts indexed [i..j] within 'positions', bounded by positions[i-1] and positions[j+1]
    private int Solve(int[] positions, int i, int j)
    {
        if (i > j) return 0; // no cuts left in this range

        int minCost = int.MaxValue;
        for (int k = i; k <= j; k++)
        {
            int cost = Solve(positions, i, k - 1) + Solve(positions, k + 1, j)
                       + (positions[j + 1] - positions[i - 1]);
            minCost = Math.Min(minCost, cost);
        }
        return minCost;
    }
}
```

Time Complexity: O(2^m) where `m` = number of cuts; Space Complexity: O(m) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class StickCutMemo
{
    public int MinCost(int n, int[] cuts)
    {
        int m = cuts.Length;
        int[] positions = new int[m + 2];
        positions[0] = 0;
        positions[m + 1] = n;
        Array.Copy(cuts, 0, positions, 1, m);
        Array.Sort(positions);

        int[,] memo = new int[m + 2, m + 2];
        for (int i = 0; i < m + 2; i++)
            for (int j = 0; j < m + 2; j++)
                memo[i, j] = -1;

        return Solve(positions, 1, m, memo);
    }

    private int Solve(int[] positions, int i, int j, int[,] memo)
    {
        if (i > j) return 0;
        if (memo[i, j] != -1) return memo[i, j];

        int minCost = int.MaxValue;
        for (int k = i; k <= j; k++)
        {
            int cost = Solve(positions, i, k - 1, memo) + Solve(positions, k + 1, j, memo)
                       + (positions[j + 1] - positions[i - 1]);
            minCost = Math.Min(minCost, cost);
        }
        return memo[i, j] = minCost;
    }
}
```

Time Complexity: O(m^3); Space Complexity: O(m^2).

**Approach 3 — Tabulation:** iterating range length from small to large.

```csharp
public class StickCutTabulation
{
    public int MinCost(int n, int[] cuts)
    {
        int m = cuts.Length;
        int[] positions = new int[m + 2];
        positions[0] = 0;
        positions[m + 1] = n;
        Array.Copy(cuts, 0, positions, 1, m);
        Array.Sort(positions);

        int[,] dp = new int[m + 2, m + 2];

        for (int len = 1; len <= m; len++)
        {
            for (int i = 1; i <= m - len + 1; i++)
            {
                int j = i + len - 1;
                dp[i, j] = int.MaxValue;
                for (int k = i; k <= j; k++)
                {
                    int cost = dp[i, k - 1] + dp[k + 1, j]
                               + (positions[j + 1] - positions[i - 1]);
                    dp[i, j] = Math.Min(dp[i, j], cost);
                }
            }
        }
        return dp[1, m];
    }
}
```

Time Complexity: O(m^3); Space Complexity: O(m^2).

---

## 3. Burst Balloons

**Problem Statement:** Given `n` balloons indexed `0` to `n-1`, each with a number on it stored in array `nums`, bursting balloon `i` gives coins equal to `nums[left] * nums[i] * nums[right]`, where `left` and `right` are the currently adjacent balloons (out-of-bounds balloons are treated as having value `1`). Find the maximum coins obtainable by bursting all balloons in some order.

**Example:**
- Input: `nums = [3, 1, 5, 8]`
- Output: `167`
- Explanation: Burst order `1, 5, 3, 8` gives `3*1*5 + 3*5*8 + 1*3*8 + 1*8*1 = 15 + 120 + 24 + 8 = 167`.

**Approach 1 — Recursion:** the range `[i, j]` recursion trying every partition point.

```csharp
public class BurstBalloonsRecursive
{
    public int MaxCoins(int[] nums)
    {
        int n = nums.Length;
        int[] balloons = new int[n + 2];
        balloons[0] = 1;
        balloons[n + 1] = 1;
        Array.Copy(nums, 0, balloons, 1, n);

        return Solve(balloons, 1, n);
    }

    // k = last balloon burst in range [i, j]; neighbors at that moment are i-1 and j+1
    private int Solve(int[] balloons, int i, int j)
    {
        if (i > j) return 0;

        int maxCoins = int.MinValue;
        for (int k = i; k <= j; k++)
        {
            int coins = balloons[i - 1] * balloons[k] * balloons[j + 1]
                        + Solve(balloons, i, k - 1) + Solve(balloons, k + 1, j);
            maxCoins = Math.Max(maxCoins, coins);
        }
        return maxCoins;
    }
}
```

Time Complexity: O(2^n); Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class BurstBalloonsMemo
{
    public int MaxCoins(int[] nums)
    {
        int n = nums.Length;
        int[] balloons = new int[n + 2];
        balloons[0] = 1;
        balloons[n + 1] = 1;
        Array.Copy(nums, 0, balloons, 1, n);

        int[,] memo = new int[n + 2, n + 2];
        for (int i = 0; i < n + 2; i++)
            for (int j = 0; j < n + 2; j++)
                memo[i, j] = -1;

        return Solve(balloons, 1, n, memo);
    }

    private int Solve(int[] balloons, int i, int j, int[,] memo)
    {
        if (i > j) return 0;
        if (memo[i, j] != -1) return memo[i, j];

        int maxCoins = int.MinValue;
        for (int k = i; k <= j; k++)
        {
            int coins = balloons[i - 1] * balloons[k] * balloons[j + 1]
                        + Solve(balloons, i, k - 1, memo) + Solve(balloons, k + 1, j, memo);
            maxCoins = Math.Max(maxCoins, coins);
        }
        return memo[i, j] = maxCoins;
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

**Approach 3 — Tabulation:** iterating range length from small to large.

```csharp
public class BurstBalloonsTabulation
{
    public int MaxCoins(int[] nums)
    {
        int n = nums.Length;
        int[] balloons = new int[n + 2];
        balloons[0] = 1;
        balloons[n + 1] = 1;
        Array.Copy(nums, 0, balloons, 1, n);

        int[,] dp = new int[n + 2, n + 2];

        for (int len = 1; len <= n; len++)
        {
            for (int i = 1; i <= n - len + 1; i++)
            {
                int j = i + len - 1;
                int best = int.MinValue;
                for (int k = i; k <= j; k++)
                {
                    int coins = balloons[i - 1] * balloons[k] * balloons[j + 1]
                                + dp[i, k - 1] + dp[k + 1, j];
                    best = Math.Max(best, coins);
                }
                dp[i, j] = best;
            }
        }
        return dp[1, n];
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

---

## 4. Evaluate Boolean Expression to True

**Problem Statement:** Given a boolean expression string containing symbols `T` (true), `F` (false), and operators `&` (AND), `|` (OR), `^` (XOR), count the number of ways to parenthesize the expression so that it evaluates to `True`. Since the count can be large, return it modulo `1000000007`.

**Example:**
- Input: `expr = "T|T&F^T"`
- Output: `4`
- Explanation: There are 4 ways to parenthesize the expression such that it evaluates to `True` (out of a total number of ways to fully parenthesize the operators between the operands).

**Approach 1 — Recursion:** the range `[i, j]` recursion trying every partition point (operator position).

```csharp
public class BooleanExprRecursive
{
    public int CountWays(string expr)
    {
        int n = expr.Length;
        return Solve(expr, 0, n - 1, true);
    }

    // isTrue: whether we want the count of ways to make expr[i..j] evaluate to True (true) or False (false)
    private int Solve(string expr, int i, int j, bool isTrue)
    {
        if (i > j) return 0;
        if (i == j)
        {
            if (isTrue) return expr[i] == 'T' ? 1 : 0;
            else return expr[i] == 'F' ? 1 : 0;
        }

        int ways = 0;
        for (int k = i + 1; k < j; k += 2) // operators are at odd offsets between operands
        {
            char op = expr[k];

            int leftTrue = Solve(expr, i, k - 1, true);
            int leftFalse = Solve(expr, i, k - 1, false);
            int rightTrue = Solve(expr, k + 1, j, true);
            int rightFalse = Solve(expr, k + 1, j, false);

            int totalWays = leftTrue * (rightTrue + rightFalse)
                             + leftFalse * (rightTrue + rightFalse); // full combinations (unused directly below)

            if (op == '&')
            {
                int trueWays = leftTrue * rightTrue;
                int falseWays = leftTrue * rightFalse + leftFalse * rightTrue + leftFalse * rightFalse;
                ways += isTrue ? trueWays : falseWays;
            }
            else if (op == '|')
            {
                int trueWays = leftTrue * rightTrue + leftTrue * rightFalse + leftFalse * rightTrue;
                int falseWays = leftFalse * rightFalse;
                ways += isTrue ? trueWays : falseWays;
            }
            else // '^'
            {
                int trueWays = leftTrue * rightFalse + leftFalse * rightTrue;
                int falseWays = leftTrue * rightTrue + leftFalse * rightFalse;
                ways += isTrue ? trueWays : falseWays;
            }
        }
        return ways;
    }
}
```

Time Complexity: O(2^n); Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class BooleanExprMemo
{
    private const int MOD = 1000000007;

    public int CountWays(string expr)
    {
        int n = expr.Length;
        // memo[i][j][0] = false-ways, memo[i][j][1] = true-ways; -1 = uncomputed
        long[,,] memo = new long[n, n, 2];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                for (int t = 0; t < 2; t++)
                    memo[i, j, t] = -1;

        return (int)Solve(expr, 0, n - 1, true, memo);
    }

    private long Solve(string expr, int i, int j, bool isTrue, long[,,] memo)
    {
        if (i > j) return 0;
        if (i == j)
        {
            if (isTrue) return expr[i] == 'T' ? 1 : 0;
            else return expr[i] == 'F' ? 1 : 0;
        }

        int t = isTrue ? 1 : 0;
        if (memo[i, j, t] != -1) return memo[i, j, t];

        long ways = 0;
        for (int k = i + 1; k < j; k += 2)
        {
            char op = expr[k];

            long leftTrue = Solve(expr, i, k - 1, true, memo);
            long leftFalse = Solve(expr, i, k - 1, false, memo);
            long rightTrue = Solve(expr, k + 1, j, true, memo);
            long rightFalse = Solve(expr, k + 1, j, false, memo);

            if (op == '&')
            {
                long trueWays = leftTrue * rightTrue % MOD;
                long falseWays = (leftTrue * rightFalse + leftFalse * rightTrue + leftFalse * rightFalse) % MOD;
                ways = (ways + (isTrue ? trueWays : falseWays)) % MOD;
            }
            else if (op == '|')
            {
                long trueWays = (leftTrue * rightTrue + leftTrue * rightFalse + leftFalse * rightTrue) % MOD;
                long falseWays = leftFalse * rightFalse % MOD;
                ways = (ways + (isTrue ? trueWays : falseWays)) % MOD;
            }
            else // '^'
            {
                long trueWays = (leftTrue * rightFalse + leftFalse * rightTrue) % MOD;
                long falseWays = (leftTrue * rightTrue + leftFalse * rightFalse) % MOD;
                ways = (ways + (isTrue ? trueWays : falseWays)) % MOD;
            }
        }
        return memo[i, j, t] = ways;
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

**Approach 3 — Tabulation:** iterating range length from small to large.

```csharp
public class BooleanExprTabulation
{
    private const int MOD = 1000000007;

    public int CountWays(string expr)
    {
        int n = expr.Length;
        long[,,] dp = new long[n, n, 2]; // dp[i, j, 0] = false ways, dp[i, j, 1] = true ways

        for (int i = 0; i < n; i++)
        {
            if (i % 2 == 0) // operand position
            {
                dp[i, i, 1] = expr[i] == 'T' ? 1 : 0;
                dp[i, i, 0] = expr[i] == 'F' ? 1 : 0;
            }
        }

        for (int len = 3; len <= n; len += 2) // ranges are between odd-length operand..operand spans
        {
            for (int i = 0; i + len - 1 < n; i += 2)
            {
                int j = i + len - 1;
                long trueWays = 0, falseWays = 0;

                for (int k = i + 1; k < j; k += 2)
                {
                    char op = expr[k];
                    long leftTrue = dp[i, k - 1, 1];
                    long leftFalse = dp[i, k - 1, 0];
                    long rightTrue = dp[k + 1, j, 1];
                    long rightFalse = dp[k + 1, j, 0];

                    if (op == '&')
                    {
                        trueWays = (trueWays + leftTrue * rightTrue) % MOD;
                        falseWays = (falseWays + leftTrue * rightFalse + leftFalse * rightTrue + leftFalse * rightFalse) % MOD;
                    }
                    else if (op == '|')
                    {
                        trueWays = (trueWays + leftTrue * rightTrue + leftTrue * rightFalse + leftFalse * rightTrue) % MOD;
                        falseWays = (falseWays + leftFalse * rightFalse) % MOD;
                    }
                    else // '^'
                    {
                        trueWays = (trueWays + leftTrue * rightFalse + leftFalse * rightTrue) % MOD;
                        falseWays = (falseWays + leftTrue * rightTrue + leftFalse * rightFalse) % MOD;
                    }
                }
                dp[i, j, 1] = trueWays % MOD;
                dp[i, j, 0] = falseWays % MOD;
            }
        }
        return (int)dp[0, n - 1, 1];
    }
}
```

Time Complexity: O(n^3); Space Complexity: O(n^2).

---

## 5. Palindrome Partitioning II

**Problem Statement:** Given a string `s`, partition it into substrings such that every substring is a palindrome, using the minimum number of cuts. Return the minimum number of cuts needed.

**Example:**
- Input: `s = "aab"`
- Output: `1`
- Explanation: Cutting once gives `"aa"` and `"b"`, both palindromes — the minimum possible number of cuts is 1.

**Approach 1 — Recursion:** the range `[i, j]` recursion trying every partition point.

```csharp
public class PalindromePartitionRecursive
{
    public int MinCut(string s)
    {
        int n = s.Length;
        return Solve(s, 0, n - 1);
    }

    private int Solve(string s, int i, int j)
    {
        if (i >= j || IsPalindrome(s, i, j)) return 0;

        int minCuts = int.MaxValue;
        for (int k = i; k < j; k++)
        {
            if (IsPalindrome(s, i, k))
            {
                int cuts = 1 + Solve(s, k + 1, j);
                minCuts = Math.Min(minCuts, cuts);
            }
        }
        return minCuts;
    }

    private bool IsPalindrome(string s, int i, int j)
    {
        while (i < j)
        {
            if (s[i] != s[j]) return false;
            i++; j--;
        }
        return true;
    }
}
```

Time Complexity: O(2^n * n) (exponential partitions, each with O(n) palindrome checks); Space Complexity: O(n) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class PalindromePartitionMemo
{
    public int MinCut(string s)
    {
        int n = s.Length;
        int[,] memo = new int[n, n];
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                memo[i, j] = -1;

        bool[,] isPalin = PrecomputePalindromes(s);
        return Solve(s, 0, n - 1, memo, isPalin);
    }

    private int Solve(string s, int i, int j, int[,] memo, bool[,] isPalin)
    {
        if (i >= j || isPalin[i, j]) return 0;
        if (memo[i, j] != -1) return memo[i, j];

        int minCuts = int.MaxValue;
        for (int k = i; k < j; k++)
        {
            if (isPalin[i, k])
            {
                int cuts = 1 + Solve(s, k + 1, j, memo, isPalin);
                minCuts = Math.Min(minCuts, cuts);
            }
        }
        return memo[i, j] = minCuts;
    }

    private bool[,] PrecomputePalindromes(string s)
    {
        int n = s.Length;
        bool[,] isPalin = new bool[n, n];
        for (int i = 0; i < n; i++) isPalin[i, i] = true;

        for (int len = 2; len <= n; len++)
        {
            for (int i = 0; i + len - 1 < n; i++)
            {
                int j = i + len - 1;
                if (s[i] == s[j] && (len == 2 || isPalin[i + 1, j - 1]))
                    isPalin[i, j] = true;
            }
        }
        return isPalin;
    }
}
```

Time Complexity: O(n^2) precompute + O(n^2) DP states with O(n) transition = O(n^3) overall (O(n^2) for palindrome table itself); Space Complexity: O(n^2).

**Approach 3 — Tabulation:** iterating range length from small to large, using a 1D "min cuts for prefix ending at i" table (the standard efficient tabulation for this problem) combined with the precomputed palindrome table.

```csharp
public class PalindromePartitionTabulation
{
    public int MinCut(string s)
    {
        int n = s.Length;
        bool[,] isPalin = PrecomputePalindromes(s);

        // dp[i] = minimum cuts needed for s[0..i]
        int[] dp = new int[n];

        for (int i = 0; i < n; i++)
        {
            if (isPalin[0, i])
            {
                dp[i] = 0;
                continue;
            }

            dp[i] = int.MaxValue;
            for (int k = 0; k < i; k++)
            {
                if (isPalin[k + 1, i] && dp[k] != int.MaxValue)
                {
                    dp[i] = Math.Min(dp[i], dp[k] + 1);
                }
            }
        }
        return dp[n - 1];
    }

    private bool[,] PrecomputePalindromes(string s)
    {
        int n = s.Length;
        bool[,] isPalin = new bool[n, n];
        for (int i = 0; i < n; i++) isPalin[i, i] = true;

        for (int len = 2; len <= n; len++)
        {
            for (int i = 0; i + len - 1 < n; i++)
            {
                int j = i + len - 1;
                if (s[i] == s[j] && (len == 2 || isPalin[i + 1, j - 1]))
                    isPalin[i, j] = true;
            }
        }
        return isPalin;
    }
}
```

Time Complexity: O(n^2) (palindrome precompute O(n^2) + cut DP O(n^2)); Space Complexity: O(n^2) for the palindrome table.

---

## 6. Partition Array for Maximum Sum

**Problem Statement:** Given an integer array `arr` and an integer `k`, partition the array into contiguous subarrays, each of length at most `k`. After partitioning, each subarray's values are all changed to the maximum value in that subarray. Return the largest possible sum of the resulting array.

**Example:**
- Input: `arr = [1, 15, 7, 9, 2, 5, 10]`, `k = 3`
- Output: `84`
- Explanation: Partition as `[1, 15, 7] | [9] | [2, 5, 10]` → becomes `[15, 15, 15, 9, 10, 10, 10]`, sum = 84.

**Approach 1 — Recursion:** range recursion trying every partition length (here the range is really a prefix/suffix boundary, and the "partition point" is how far the next chunk extends).

```csharp
public class MaxSumPartitionRecursive
{
    public int MaxSumAfterPartitioning(int[] arr, int k)
    {
        int n = arr.Length;
        return Solve(arr, 0, k);
    }

    // Solve for suffix starting at index i, choosing chunk lengths up to k
    private int Solve(int[] arr, int i, int k)
    {
        int n = arr.Length;
        if (i == n) return 0;

        int maxSum = int.MinValue;
        int currMax = int.MinValue;
        int limit = Math.Min(k, n - i);

        for (int len = 1; len <= limit; len++)
        {
            currMax = Math.Max(currMax, arr[i + len - 1]);
            int sum = currMax * len + Solve(arr, i + len, k);
            maxSum = Math.Max(maxSum, sum);
        }
        return maxSum;
    }
}
```

Time Complexity: O(k^n) in the worst case (exponential branching over chunk lengths, n/k levels of depth each branching k ways); Space Complexity: O(n/k) recursion stack.

**Approach 2 — Memoization:**

```csharp
public class MaxSumPartitionMemo
{
    public int MaxSumAfterPartitioning(int[] arr, int k)
    {
        int n = arr.Length;
        int[] memo = new int[n];
        for (int i = 0; i < n; i++) memo[i] = -1;

        return Solve(arr, 0, k, memo);
    }

    private int Solve(int[] arr, int i, int k, int[] memo)
    {
        int n = arr.Length;
        if (i == n) return 0;
        if (memo[i] != -1) return memo[i];

        int maxSum = int.MinValue;
        int currMax = int.MinValue;
        int limit = Math.Min(k, n - i);

        for (int len = 1; len <= limit; len++)
        {
            currMax = Math.Max(currMax, arr[i + len - 1]);
            int sum = currMax * len + Solve(arr, i + len, k, memo);
            maxSum = Math.Max(maxSum, sum);
        }
        return memo[i] = maxSum;
    }
}
```

Time Complexity: O(n * k); Space Complexity: O(n).

**Approach 3 — Tabulation:** iterating the range end from small to large.

```csharp
public class MaxSumPartitionTabulation
{
    public int MaxSumAfterPartitioning(int[] arr, int k)
    {
        int n = arr.Length;
        int[] dp = new int[n + 1]; // dp[i] = max sum achievable for suffix starting at index i
        dp[n] = 0;

        for (int i = n - 1; i >= 0; i--)
        {
            int maxSum = int.MinValue;
            int currMax = int.MinValue;
            int limit = Math.Min(k, n - i);

            for (int len = 1; len <= limit; len++)
            {
                currMax = Math.Max(currMax, arr[i + len - 1]);
                int sum = currMax * len + dp[i + len];
                maxSum = Math.Max(maxSum, sum);
            }
            dp[i] = maxSum;
        }
        return dp[0];
    }
}
```

Time Complexity: O(n * k); Space Complexity: O(n).

---

## Explanation

### Dry Run: Matrix Chain Multiplication on `[40, 20, 30, 10, 30]`

Matrices: `A1 (40x20)`, `A2 (20x30)`, `A3 (30x10)`, `A4 (10x30)`. We fill a table `dp[i][j]` = minimum cost to multiply matrices `Ai...Aj`, for chain lengths `len = 2, 3, 4`.

**Length 2 (adjacent pairs):**
- `dp[1][2]` = cost of `A1*A2` = `40*20*30` = `24000`
- `dp[2][3]` = cost of `A2*A3` = `20*30*10` = `6000`
- `dp[3][4]` = cost of `A3*A4` = `30*10*30` = `9000`

**Length 3:**
- `dp[1][3]` (multiply `A1 A2 A3`): try `k=1`: `dp[1][1] + dp[2][3] + arr[0]*arr[1]*arr[3]` = `0 + 6000 + 40*20*10` = `6000+8000=14000`; try `k=2`: `dp[1][2] + dp[3][3] + arr[0]*arr[2]*arr[3]` = `24000+0+40*30*10=24000+12000=36000`. Minimum = `14000` at `k=1`.
- `dp[2][4]` (multiply `A2 A3 A4`): try `k=2`: `dp[2][2]+dp[3][4]+arr[1]*arr[2]*arr[4]` = `0+9000+20*30*30=9000+18000=27000`; try `k=3`: `dp[2][3]+dp[4][4]+arr[1]*arr[3]*arr[4]` = `6000+0+20*10*30=6000+6000=12000`. Minimum = `12000` at `k=3`.

**Length 4 (the full chain `A1 A2 A3 A4`):**
- `dp[1][4]`: try each split point `k = 1, 2, 3`:
  - `k=1`: `dp[1][1] + dp[2][4] + arr[0]*arr[1]*arr[4]` = `0 + 12000 + 40*20*30` = `12000 + 24000 = 36000`
  - `k=2`: `dp[1][2] + dp[3][4] + arr[0]*arr[2]*arr[4]` = `24000 + 9000 + 40*30*30` = `33000 + 36000 = 69000`
  - `k=3`: `dp[1][3] + dp[4][4] + arr[0]*arr[3]*arr[4]` = `14000 + 0 + 40*10*30` = `14000 + 12000 = 26000`
  - Minimum over all `k` is `26000`, achieved at `k=3`, i.e., parenthesization `((A1 A2 A3) A4)`, which itself uses `A1 A2 A3` optimally split at `k=1` as `(A1 (A2 A3))`.

Final answer: `dp[1][4] = 26000`, matching the expected output. This dry run shows exactly the partition-DP pattern: the table is filled by increasing range length, and at each `[i, j]` we scan every partition point `k` and keep the best combination of `dp[i][k] + dp[k+1][j] + cost(i, k, j)`.

### The Burst Balloons Trick: Think "Last", Not "First"

The naive idea is to pick the balloon burst **first** in a range and recurse on the two remaining pieces — but that fails, because once a balloon is burst, the two neighboring pieces are no longer independent (bursting order in one side can change what is adjacent on the other side when the boundary balloon is gone). It also makes the reward for the first burst depend on values that will change.

The fix is to think about which balloon is burst **last** in the range `[i, j]`. If balloon `k` is the *last* one to be burst in `[i, j]`, then at the moment it is burst, every other balloon in `[i, j]` has already been popped — so its immediate left and right neighbors are guaranteed to still be the *boundary* balloons `i-1` and `j+1` (whatever lies just outside the range), regardless of the order in which the rest of `[i, k-1]` and `[k+1, j]` were burst. This means:

- The reward for bursting `k` last is fixed: `balloons[i-1] * balloons[k] * balloons[j+1]`.
- The two sub-ranges `[i, k-1]` and `[k+1, j]` become **completely independent** sub-problems, because they will each be fully burst (in any order) before `k`, and neither range's outcome affects the other's boundary values.

This independence is exactly what the partition-DP recurrence needs (`dp(i,j) = max over k of dp(i,k-1) + dp(k+1,j) + cost`). Padding the array with `1` at both ends (`balloons[0] = balloons[n+1] = 1`) removes the need for special-casing out-of-bounds neighbors — multiplying by `1` is a no-op, so the same formula `balloons[i-1] * balloons[k] * balloons[j+1]` works uniformly even when `k` is at the very edge of the original array.

### Palindrome Partitioning II: Precomputing the Palindrome Table

The recursive/memoized solution to Palindrome Partitioning II checks, for every candidate partition point `k` inside `dp(i, j)`, whether `s[i..k]` is a palindrome. A naive `IsPalindrome(s, i, k)` check costs O(n) time by walking inward from both ends, and it gets called O(n) times per state across O(n^2) states — pushing the total complexity to O(n^3) or worse, and doing a lot of redundant re-scanning of the same substrings.

The optimization is to **precompute** a 2D boolean table `isPalin[i][j]` = "is `s[i..j]` a palindrome?" for all `i <= j`, in O(n^2) time, using the recurrence:

```text
isPalin[i][i] = true
isPalin[i][j] = (s[i] == s[j]) && (j - i < 2 || isPalin[i+1][j-1])
```

filled by increasing substring length (exactly the same "small ranges before large ranges" order used throughout partition DP). Once this table exists, every palindrome check inside the O(n^2) cut-counting DP becomes an O(1) array lookup instead of an O(n) scan, dropping the overall complexity for the cut DP itself to O(n^2) (dominated by the O(n^2) states each doing O(n) work in the two-table version, or O(n^2) total in the classic single-array `dp[i]` = min cuts for `s[0..i]` formulation). This precompute-a-lookup-table-then-reuse-it strategy is a common companion technique alongside partition DP whenever the "combine" step needs a fast yes/no or cost query about a sub-range.
