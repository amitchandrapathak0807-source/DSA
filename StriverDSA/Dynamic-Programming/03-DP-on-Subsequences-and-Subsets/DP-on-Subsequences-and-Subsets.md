# Dynamic Programming — DP on Subsequences and Subsets

## Concept: Pick/Not-Pick DP on Subsets

Every problem in this section is built on the exact same recursion tree you already
know from the **Recursion** topic: for every index `i` in the array you have exactly
two choices —

1. **Pick** `arr[i]` (only if it doesn't violate a constraint such as a remaining
   target going negative), and recurse on `(i - 1, target - arr[i])`.
2. **Not-Pick** `arr[i]`, and recurse on `(i - 1, target)`.

That is the entire "subsequence generation" pattern from recursion — nothing new is
introduced conceptually. The only difference is *what we do with the two recursive
calls*: instead of just generating/printing all `2^n` subsequences, we combine the
two results with an `OR` (does a subset exist?), a `SUM` (how many subsets exist?),
or a `MAX` (what is the best value obtainable?).

Because the recursion is called repeatedly with the **same `(index, target)` pair**
along different branches of the tree, plain recursion recomputes the same
sub-problem exponentially many times — `O(2^n)`. The fix is the standard DP
recipe:

- Identify the **changing parameters** of the recursive call — here they are
  `index` and `target` (or `remaining sum`, `remaining capacity`, etc.).
- Create a memo table `dp[index][target]` sized `(n) x (target + 1)`.
- Before doing any work, check whether `dp[index][target]` has already been
  computed; if so, return it directly.
- Otherwise compute it via the same pick/not-pick recursion and **store** it in
  `dp[index][target]` before returning.

This single change — memoizing on `(index, target)` — collapses the search space
from `O(2^n)` down to `O(n * target)`, because there are only `n * (target + 1)`
distinct `(index, target)` pairs, and each is computed once.

From there:

- **Tabulation** removes recursion entirely by building the same `dp` table
  bottom-up (from the smallest sub-problems, i.e. `index = 0`, up to
  `index = n - 1`), using iteration instead of function calls.
- **Space Optimization** notices that `dp[index][*]` only ever depends on
  `dp[index - 1][*]`, i.e. only the *previous row* is needed. So the 2D table
  collapses into two 1D arrays (`prev`/`cur`), which can then be collapsed
  further into a single 1D array if the target dimension is iterated
  **from high to low**.

Every one of the six problems below is a thin variation on this same
"does a subset exist with sum X" / "how many subsets have sum X" DP. Once you
internalize the pick/not-pick recursion + `dp[index][target]` memo table, all six
problems (and the classic 0/1 Knapsack) become the same 20 lines of code with
minor tweaks to what gets combined at each step.

---

## 1. Subset Sum Equal to Target

**Problem Statement:**
Given an array `arr` of `n` non-negative integers and an integer `target`,
determine whether there exists a subset of `arr` whose elements sum exactly to
`target`.

**Example:**
- Input: `arr = [1, 2, 3, 3]`, `target = 6`
- Output: `true`
- Explanation: The subset `{1, 2, 3}` (using the first 3) sums to `6`. Also
  `{3, 3}` sums to `6`.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class SubsetSumRecursion
{
    public bool SubsetSumExists(int[] arr, int target)
    {
        return Solve(arr.Length - 1, target, arr);
    }

    private bool Solve(int index, int target, int[] arr)
    {
        if (target == 0) return true;
        if (index == 0) return arr[0] == target;

        bool notPick = Solve(index - 1, target, arr);
        bool pick = false;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr);

        return pick || notPick;
    }
}
```
Time Complexity: O(2^n), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class SubsetSumMemoization
{
    public bool SubsetSumExists(int[] arr, int target)
    {
        int n = arr.Length;
        int[,] dp = new int[n, target + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= target; j++)
                dp[i, j] = -1; // -1 = not computed

        return Solve(n - 1, target, arr, dp) == 1;
    }

    private int Solve(int index, int target, int[] arr, int[,] dp)
    {
        if (target == 0) return 1;
        if (index == 0) return arr[0] == target ? 1 : 0;
        if (dp[index, target] != -1) return dp[index, target];

        int notPick = Solve(index - 1, target, arr, dp);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr, dp);

        return dp[index, target] = (pick == 1 || notPick == 1) ? 1 : 0;
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(n*target) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class SubsetSumTabulation
{
    public bool SubsetSumExists(int[] arr, int target)
    {
        int n = arr.Length;
        bool[,] dp = new bool[n, target + 1];

        // Base cases
        for (int i = 0; i < n; i++)
            dp[i, 0] = true; // sum 0 is always achievable (empty subset)

        if (arr[0] <= target)
            dp[0, arr[0]] = true;

        for (int index = 1; index < n; index++)
        {
            for (int t = 1; t <= target; t++)
            {
                bool notPick = dp[index - 1, t];
                bool pick = false;
                if (arr[index] <= t)
                    pick = dp[index - 1, t - arr[index]];

                dp[index, t] = pick || notPick;
            }
        }

        return dp[n - 1, target];
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(n*target).

**Approach 4 — Space Optimization:**
```csharp
public class SubsetSumSpaceOptimized
{
    public bool SubsetSumExists(int[] arr, int target)
    {
        int n = arr.Length;
        bool[] prev = new bool[target + 1];

        prev[0] = true;
        if (arr[0] <= target)
            prev[arr[0]] = true;

        for (int index = 1; index < n; index++)
        {
            bool[] cur = new bool[target + 1];
            cur[0] = true;

            for (int t = 1; t <= target; t++)
            {
                bool notPick = prev[t];
                bool pick = false;
                if (arr[index] <= t)
                    pick = prev[t - arr[index]];

                cur[t] = pick || notPick;
            }
            prev = cur;
        }

        return prev[target];
    }

    // Truly 1D version: iterate target high -> low so each item is used at most once per pass.
    public bool SubsetSumExists1D(int[] arr, int target)
    {
        int n = arr.Length;
        bool[] dp = new bool[target + 1];
        dp[0] = true;

        for (int index = 0; index < n; index++)
        {
            for (int t = target; t >= arr[index]; t--)
            {
                if (dp[t - arr[index]])
                    dp[t] = true;
            }
        }

        return dp[target];
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(target).

---

## 2. Partition Equal Subset Sum

**Problem Statement:**
Given an array `arr` of `n` positive integers, determine whether it can be
partitioned into two subsets such that the sum of elements in both subsets is
equal.

**Example:**
- Input: `arr = [1, 5, 11, 5]`
- Output: `true`
- Explanation: The array can be partitioned as `[1, 5, 5]` and `[11]`, both
  summing to `11`.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class PartitionEqualSubsetSumRecursion
{
    public bool CanPartition(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        if (totalSum % 2 != 0) return false; // odd total can never split evenly

        int target = totalSum / 2;
        return Solve(arr.Length - 1, target, arr);
    }

    private bool Solve(int index, int target, int[] arr)
    {
        if (target == 0) return true;
        if (index == 0) return arr[0] == target;

        bool notPick = Solve(index - 1, target, arr);
        bool pick = false;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr);

        return pick || notPick;
    }
}
```
Time Complexity: O(2^n), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class PartitionEqualSubsetSumMemoization
{
    public bool CanPartition(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;
        if (totalSum % 2 != 0) return false;

        int target = totalSum / 2;
        int n = arr.Length;
        int[,] dp = new int[n, target + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= target; j++)
                dp[i, j] = -1;

        return Solve(n - 1, target, arr, dp) == 1;
    }

    private int Solve(int index, int target, int[] arr, int[,] dp)
    {
        if (target == 0) return 1;
        if (index == 0) return arr[0] == target ? 1 : 0;
        if (dp[index, target] != -1) return dp[index, target];

        int notPick = Solve(index - 1, target, arr, dp);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr, dp);

        return dp[index, target] = (pick == 1 || notPick == 1) ? 1 : 0;
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(n*target) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class PartitionEqualSubsetSumTabulation
{
    public bool CanPartition(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;
        if (totalSum % 2 != 0) return false;

        int target = totalSum / 2;
        int n = arr.Length;
        bool[,] dp = new bool[n, target + 1];

        for (int i = 0; i < n; i++)
            dp[i, 0] = true;

        if (arr[0] <= target)
            dp[0, arr[0]] = true;

        for (int index = 1; index < n; index++)
        {
            for (int t = 1; t <= target; t++)
            {
                bool notPick = dp[index - 1, t];
                bool pick = false;
                if (arr[index] <= t)
                    pick = dp[index - 1, t - arr[index]];

                dp[index, t] = pick || notPick;
            }
        }

        return dp[n - 1, target];
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(n*target).

**Approach 4 — Space Optimization:**
```csharp
public class PartitionEqualSubsetSumSpaceOptimized
{
    public bool CanPartition(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;
        if (totalSum % 2 != 0) return false;

        int target = totalSum / 2;
        int n = arr.Length;
        bool[] dp = new bool[target + 1];
        dp[0] = true;

        for (int index = 0; index < n; index++)
        {
            for (int t = target; t >= arr[index]; t--)
            {
                if (dp[t - arr[index]])
                    dp[t] = true;
            }
        }

        return dp[target];
    }
}
```
Time Complexity: O(n*target), Space Complexity: O(target).

---

## 3. Partition a Set Into Two Subsets With Minimum Absolute Sum Difference

**Problem Statement:**
Given an array `arr` of `n` non-negative integers, partition it into two subsets
`S1` and `S2` such that `|sum(S1) - sum(S2)|` is minimized. Return that minimum
absolute difference.

**Example:**
- Input: `arr = [1, 6, 11, 5]`
- Output: `1`
- Explanation: `S1 = {1, 5, 6} (sum 12)`, `S2 = {11} (sum 11)`,
  `|12 - 11| = 1`, which is the minimum possible.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class MinSubsetSumDifferenceRecursion
{
    public int MinDifference(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        int n = arr.Length;
        int minDiff = int.MaxValue;

        // Try every possible sum s for S1 from 0..totalSum, keep the ones that are achievable.
        for (int s = 0; s <= totalSum; s++)
        {
            if (Solve(n - 1, s, arr))
            {
                int diff = Math.Abs(totalSum - 2 * s);
                minDiff = Math.Min(minDiff, diff);
            }
        }
        return minDiff;
    }

    private bool Solve(int index, int target, int[] arr)
    {
        if (target == 0) return true;
        if (index == 0) return arr[0] == target;

        bool notPick = Solve(index - 1, target, arr);
        bool pick = false;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr);

        return pick || notPick;
    }
}
```
Time Complexity: O(2^n * totalSum), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class MinSubsetSumDifferenceMemoization
{
    public int MinDifference(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        int n = arr.Length;
        int[,] dp = new int[n, totalSum + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= totalSum; j++)
                dp[i, j] = -1;

        int minDiff = int.MaxValue;
        for (int s = 0; s <= totalSum; s++)
        {
            if (Solve(n - 1, s, arr, dp) == 1)
            {
                int diff = Math.Abs(totalSum - 2 * s);
                minDiff = Math.Min(minDiff, diff);
            }
        }
        return minDiff;
    }

    private int Solve(int index, int target, int[] arr, int[,] dp)
    {
        if (target == 0) return 1;
        if (index == 0) return arr[0] == target ? 1 : 0;
        if (dp[index, target] != -1) return dp[index, target];

        int notPick = Solve(index - 1, target, arr, dp);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr, dp);

        return dp[index, target] = (pick == 1 || notPick == 1) ? 1 : 0;
    }
}
```
Time Complexity: O(n*totalSum), Space Complexity: O(n*totalSum) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class MinSubsetSumDifferenceTabulation
{
    public int MinDifference(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        int n = arr.Length;
        bool[,] dp = new bool[n, totalSum + 1];

        for (int i = 0; i < n; i++)
            dp[i, 0] = true;

        if (arr[0] <= totalSum)
            dp[0, arr[0]] = true;

        for (int index = 1; index < n; index++)
        {
            for (int t = 1; t <= totalSum; t++)
            {
                bool notPick = dp[index - 1, t];
                bool pick = false;
                if (arr[index] <= t)
                    pick = dp[index - 1, t - arr[index]];

                dp[index, t] = pick || notPick;
            }
        }

        // Scan the last row for every achievable sum s and find the one closest to totalSum/2.
        int minDiff = int.MaxValue;
        for (int s = 0; s <= totalSum; s++)
        {
            if (dp[n - 1, s])
            {
                int diff = Math.Abs(totalSum - 2 * s);
                minDiff = Math.Min(minDiff, diff);
            }
        }
        return minDiff;
    }
}
```
Time Complexity: O(n*totalSum), Space Complexity: O(n*totalSum).

**Approach 4 — Space Optimization:**
```csharp
public class MinSubsetSumDifferenceSpaceOptimized
{
    public int MinDifference(int[] arr)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        int n = arr.Length;
        bool[] dp = new bool[totalSum + 1];
        dp[0] = true;

        for (int index = 0; index < n; index++)
        {
            for (int t = totalSum; t >= arr[index]; t--)
            {
                if (dp[t - arr[index]])
                    dp[t] = true;
            }
        }

        int minDiff = int.MaxValue;
        for (int s = 0; s <= totalSum; s++)
        {
            if (dp[s])
            {
                int diff = Math.Abs(totalSum - 2 * s);
                minDiff = Math.Min(minDiff, diff);
            }
        }
        return minDiff;
    }
}
```
Time Complexity: O(n*totalSum), Space Complexity: O(totalSum).

---

## 4. Count Subsets with Sum Equal to K

**Problem Statement:**
Given an array `arr` of `n` non-negative integers and an integer `k`, count the
number of subsets whose elements sum exactly to `k`.

**Example:**
- Input: `arr = [1, 2, 2, 3]`, `k = 3`
- Output: `3`
- Explanation: The subsets that sum to `3` are `{1, 2}` (using the first `2`),
  `{1, 2}` (using the second `2`), and `{3}`.

> Note: when the array can contain `0`s, the base case needs a small tweak
> (each `0` doubles the count of subsets for the same target, because it can be
> either included or excluded without changing the sum). The implementations
> below handle that explicitly.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class CountSubsetsWithSumKRecursion
{
    public int CountSubsets(int[] arr, int k)
    {
        return Solve(arr.Length - 1, k, arr);
    }

    private int Solve(int index, int target, int[] arr)
    {
        if (index == 0)
        {
            if (target == 0 && arr[0] == 0) return 2; // pick or not-pick the 0
            if (target == 0 || arr[0] == target) return 1;
            return 0;
        }

        int notPick = Solve(index - 1, target, arr);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr);

        return pick + notPick;
    }
}
```
Time Complexity: O(2^n), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class CountSubsetsWithSumKMemoization
{
    public int CountSubsets(int[] arr, int k)
    {
        int n = arr.Length;
        int[,] dp = new int[n, k + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= k; j++)
                dp[i, j] = -1;

        return Solve(n - 1, k, arr, dp);
    }

    private int Solve(int index, int target, int[] arr, int[,] dp)
    {
        if (index == 0)
        {
            if (target == 0 && arr[0] == 0) return 2;
            if (target == 0 || arr[0] == target) return 1;
            return 0;
        }
        if (dp[index, target] != -1) return dp[index, target];

        int notPick = Solve(index - 1, target, arr, dp);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr, dp);

        return dp[index, target] = pick + notPick;
    }
}
```
Time Complexity: O(n*k), Space Complexity: O(n*k) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class CountSubsetsWithSumKTabulation
{
    public int CountSubsets(int[] arr, int k)
    {
        int n = arr.Length;
        int[,] dp = new int[n, k + 1];

        // Base case for index 0
        for (int t = 0; t <= k; t++)
        {
            if (t == 0 && arr[0] == 0) dp[0, t] = 2;
            else if (t == 0 || arr[0] == t) dp[0, t] = 1;
            else dp[0, t] = 0;
        }

        for (int index = 1; index < n; index++)
        {
            for (int t = 0; t <= k; t++)
            {
                int notPick = dp[index - 1, t];
                int pick = 0;
                if (arr[index] <= t)
                    pick = dp[index - 1, t - arr[index]];

                dp[index, t] = pick + notPick;
            }
        }

        return dp[n - 1, k];
    }
}
```
Time Complexity: O(n*k), Space Complexity: O(n*k).

**Approach 4 — Space Optimization:**
```csharp
public class CountSubsetsWithSumKSpaceOptimized
{
    public int CountSubsets(int[] arr, int k)
    {
        int n = arr.Length;
        int[] dp = new int[k + 1];

        for (int t = 0; t <= k; t++)
        {
            if (t == 0 && arr[0] == 0) dp[t] = 2;
            else if (t == 0 || arr[0] == t) dp[t] = 1;
            else dp[t] = 0;
        }

        for (int index = 1; index < n; index++)
        {
            for (int t = k; t >= arr[index]; t--)
            {
                dp[t] = dp[t] + dp[t - arr[index]];
            }
        }

        return dp[k];
    }
}
```
Time Complexity: O(n*k), Space Complexity: O(k).

---

## 5. Count Partitions with a Given Difference

**Problem Statement:**
Given an array `arr` of `n` non-negative integers and an integer `d`, count the
number of ways to partition the array into two subsets `S1` and `S2` such that
`sum(S1) - sum(S2) = d` (with `sum(S1) >= sum(S2)`).

**Example:**
- Input: `arr = [1, 1, 2, 3]`, `d = 1`
- Output: `3`
- Explanation: `totalSum = 7`, so `s1 = (7 + 1) / 2 = 4`. Counting subsets that
  sum to `4`: `{1, 3}` (first 1), `{1, 3}` (second 1), `{1, 1, 2}` — 3 ways.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class CountPartitionsWithDiffRecursion
{
    public int CountPartitions(int[] arr, int d)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        // sum(S1) - sum(S2) = d and sum(S1) + sum(S2) = totalSum
        // => sum(S1) = (totalSum + d) / 2
        if ((totalSum + d) % 2 != 0) return 0;
        if (totalSum < d) return 0;

        int s1 = (totalSum + d) / 2;
        return Solve(arr.Length - 1, s1, arr);
    }

    private int Solve(int index, int target, int[] arr)
    {
        if (index == 0)
        {
            if (target == 0 && arr[0] == 0) return 2;
            if (target == 0 || arr[0] == target) return 1;
            return 0;
        }

        int notPick = Solve(index - 1, target, arr);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr);

        return pick + notPick;
    }
}
```
Time Complexity: O(2^n), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class CountPartitionsWithDiffMemoization
{
    public int CountPartitions(int[] arr, int d)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        if ((totalSum + d) % 2 != 0) return 0;
        if (totalSum < d) return 0;

        int s1 = (totalSum + d) / 2;
        int n = arr.Length;
        int[,] dp = new int[n, s1 + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= s1; j++)
                dp[i, j] = -1;

        return Solve(n - 1, s1, arr, dp);
    }

    private int Solve(int index, int target, int[] arr, int[,] dp)
    {
        if (index == 0)
        {
            if (target == 0 && arr[0] == 0) return 2;
            if (target == 0 || arr[0] == target) return 1;
            return 0;
        }
        if (dp[index, target] != -1) return dp[index, target];

        int notPick = Solve(index - 1, target, arr, dp);
        int pick = 0;
        if (arr[index] <= target)
            pick = Solve(index - 1, target - arr[index], arr, dp);

        return dp[index, target] = pick + notPick;
    }
}
```
Time Complexity: O(n*s1), Space Complexity: O(n*s1) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class CountPartitionsWithDiffTabulation
{
    public int CountPartitions(int[] arr, int d)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        if ((totalSum + d) % 2 != 0) return 0;
        if (totalSum < d) return 0;

        int s1 = (totalSum + d) / 2;
        int n = arr.Length;
        int[,] dp = new int[n, s1 + 1];

        for (int t = 0; t <= s1; t++)
        {
            if (t == 0 && arr[0] == 0) dp[0, t] = 2;
            else if (t == 0 || arr[0] == t) dp[0, t] = 1;
            else dp[0, t] = 0;
        }

        for (int index = 1; index < n; index++)
        {
            for (int t = 0; t <= s1; t++)
            {
                int notPick = dp[index - 1, t];
                int pick = 0;
                if (arr[index] <= t)
                    pick = dp[index - 1, t - arr[index]];

                dp[index, t] = pick + notPick;
            }
        }

        return dp[n - 1, s1];
    }
}
```
Time Complexity: O(n*s1), Space Complexity: O(n*s1).

**Approach 4 — Space Optimization:**
```csharp
public class CountPartitionsWithDiffSpaceOptimized
{
    public int CountPartitions(int[] arr, int d)
    {
        int totalSum = 0;
        foreach (int x in arr) totalSum += x;

        if ((totalSum + d) % 2 != 0) return 0;
        if (totalSum < d) return 0;

        int s1 = (totalSum + d) / 2;
        int n = arr.Length;
        int[] dp = new int[s1 + 1];

        for (int t = 0; t <= s1; t++)
        {
            if (t == 0 && arr[0] == 0) dp[t] = 2;
            else if (t == 0 || arr[0] == t) dp[t] = 1;
            else dp[t] = 0;
        }

        for (int index = 1; index < n; index++)
        {
            for (int t = s1; t >= arr[index]; t--)
            {
                dp[t] = dp[t] + dp[t - arr[index]];
            }
        }

        return dp[s1];
    }
}
```
Time Complexity: O(n*s1), Space Complexity: O(s1).

---

## 6. 0/1 Knapsack

**Problem Statement:**
Given `n` items, each with a `weight[i]` and a `value[i]`, and a knapsack of
capacity `W`, find the maximum total value that can be put into the knapsack
such that the sum of weights does not exceed `W`. Each item can be used at
most once (0/1 — either taken whole or not at all).

**Example:**
- Input: `weight = [1, 2, 4, 5]`, `value = [5, 4, 8, 6]`, `W = 5`
- Output: `13`
- Explanation: Taking items with weight `1` and `4` (values `5` and `8`) uses
  `5` capacity exactly and gives value `13`, which is the maximum possible.

**Approach 1 — Recursion (Pick/Not-Pick):**
```csharp
public class KnapsackRecursion
{
    public int MaxValue(int[] weight, int[] value, int W)
    {
        return Solve(weight.Length - 1, W, weight, value);
    }

    private int Solve(int index, int capacity, int[] weight, int[] value)
    {
        if (index == 0)
        {
            return weight[0] <= capacity ? value[0] : 0;
        }

        int notPick = Solve(index - 1, capacity, weight, value);
        int pick = int.MinValue;
        if (weight[index] <= capacity)
            pick = value[index] + Solve(index - 1, capacity - weight[index], weight, value);

        return Math.Max(pick, notPick);
    }
}
```
Time Complexity: O(2^n), Space Complexity: O(n).

**Approach 2 — Memoization:**
```csharp
public class KnapsackMemoization
{
    public int MaxValue(int[] weight, int[] value, int W)
    {
        int n = weight.Length;
        int[,] dp = new int[n, W + 1];
        for (int i = 0; i < n; i++)
            for (int j = 0; j <= W; j++)
                dp[i, j] = -1;

        return Solve(n - 1, W, weight, value, dp);
    }

    private int Solve(int index, int capacity, int[] weight, int[] value, int[,] dp)
    {
        if (index == 0)
            return weight[0] <= capacity ? value[0] : 0;

        if (dp[index, capacity] != -1) return dp[index, capacity];

        int notPick = Solve(index - 1, capacity, weight, value, dp);
        int pick = int.MinValue;
        if (weight[index] <= capacity)
            pick = value[index] + Solve(index - 1, capacity - weight[index], weight, value, dp);

        return dp[index, capacity] = Math.Max(pick, notPick);
    }
}
```
Time Complexity: O(n*W), Space Complexity: O(n*W) + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class KnapsackTabulation
{
    public int MaxValue(int[] weight, int[] value, int W)
    {
        int n = weight.Length;
        int[,] dp = new int[n, W + 1];

        for (int cap = weight[0]; cap <= W; cap++)
            dp[0, cap] = value[0];

        for (int index = 1; index < n; index++)
        {
            for (int cap = 0; cap <= W; cap++)
            {
                int notPick = dp[index - 1, cap];
                int pick = int.MinValue;
                if (weight[index] <= cap)
                    pick = value[index] + dp[index - 1, cap - weight[index]];

                dp[index, cap] = Math.Max(pick, notPick);
            }
        }

        return dp[n - 1, W];
    }
}
```
Time Complexity: O(n*W), Space Complexity: O(n*W).

**Approach 4 — Space Optimization:**
```csharp
public class KnapsackSpaceOptimized
{
    public int MaxValue(int[] weight, int[] value, int W)
    {
        int n = weight.Length;
        int[] dp = new int[W + 1];

        for (int cap = weight[0]; cap <= W; cap++)
            dp[cap] = value[0];

        for (int index = 1; index < n; index++)
        {
            for (int cap = W; cap >= weight[index]; cap--)
            {
                dp[cap] = Math.Max(dp[cap], value[index] + dp[cap - weight[index]]);
            }
        }

        return dp[W];
    }
}
```
Time Complexity: O(n*W), Space Complexity: O(W).

---

## Explanation: How Problems 2–5 Reduce to Subset-Sum / Count-Subsets

All four of problems 2, 3, 4, and 5 are **not new DP problems** — they are the
exact same `dp[index][target]` subset-sum machinery from problem 1 (existence)
or problem 4 (counting), wrapped with a small amount of pre/post-processing.

- **Problem 2 — Partition Equal Subset Sum:** If the total array sum is odd, an
  equal split is impossible, so return `false` immediately. Otherwise the array
  can be split into two equal halves if and only if a subset exists that sums
  to exactly `totalSum / 2` (the other half automatically sums to the same
  value). So the whole problem is just one call:
  `subsetSumExists(arr, totalSum / 2)`.

- **Problem 3 — Minimum Absolute Sum Difference:** Instead of asking "does a
  subset with one specific sum exist," we build the *entire* last row of the
  subset-sum tabulation table (`dp[n-1][s]` for every `s` from `0` to
  `totalSum`), which marks **every reachable subset sum**. If `S1` has sum `s`,
  then `S2` has sum `totalSum - s`, so the difference is
  `|totalSum - 2s|`. We scan all reachable `s` values and take the one that
  minimizes `|totalSum - 2s|` — i.e., the reachable sum closest to
  `totalSum / 2`.

- **Problem 4 — Count Subsets with Sum K:** This is the counting analogue of
  problem 1: instead of `OR`-ing the pick/not-pick branches (existence), we
  `SUM` them (count of ways). This is used directly as a subroutine by problem
  5, and its DP structure is what problem 5 calls under the hood.

- **Problem 5 — Count Partitions with a Given Difference:** We want subsets
  `S1, S2` with `sum(S1) - sum(S2) = d`. Combined with
  `sum(S1) + sum(S2) = totalSum`, solving the two simultaneous equations gives
  `s1 = (totalSum + d) / 2`. If `totalSum + d` is odd (no integer solution) or
  `d > totalSum` (impossible), the answer is `0`. Otherwise the answer is
  exactly `countSubsetsWithSumK(arr, s1)` — the same DP as problem 4, just
  called with a derived target.

So in practice, once you have written the subset-sum-exists DP and the
count-subsets-with-sum-k DP, problems 2, 3, and 5 require almost no new code —
just a formula to compute the right `target` to plug into an existing
subroutine, or (for problem 3) a scan over the full row of results.

### Dry Run: Why the 1D Space-Optimized DP Iterates `target` in Reverse

Take `arr = [2, 3]` and `target = 5`, using the space-optimized subset-sum DP
(`dp[0] = true`, all others `false` initially):

```
dp indices:  0     1     2     3     4     5
Initial:     T     F     F     F     F     F
```

**Processing `index = 0`, `arr[0] = 2`** (loop `t` from `5` down to `2`):

- `t = 5`: `dp[5 - 2] = dp[3] = F` → `dp[5]` stays `F`.
- `t = 4`: `dp[4 - 2] = dp[2] = F` → `dp[4]` stays `F`.
- `t = 3`: `dp[3 - 2] = dp[1] = F` → `dp[3]` stays `F`.
- `t = 2`: `dp[2 - 2] = dp[0] = T` → `dp[2] = T`.

```
After index 0:  0     1     2     3     4     5
                T     F     T     F     F     F
```

**Processing `index = 1`, `arr[1] = 3`** (loop `t` from `5` down to `3`):

- `t = 5`: `dp[5 - 3] = dp[2] = T` → `dp[5] = T`.
- `t = 4`: `dp[4 - 3] = dp[1] = F` → `dp[4]` stays `F`.
- `t = 3`: `dp[3 - 3] = dp[0] = T` → `dp[3] = T`.

```
After index 1:  0     1     2     3     4     5
                T     F     T     T     F     T
```

Final answer: `dp[5] = true` — the subset `{2, 3}` sums to `5`. Correct.

**Why reverse iteration matters:** when processing `index = 1` (`arr[1] = 3`),
the update `dp[5] = dp[5] || dp[2]` is using `dp[2]`, which was set during
`index = 0`'s pass — i.e., it correctly represents "can I reach `2` using items
*before* index 1, then add item 1." If instead we iterated `t` **forward**
(low to high) within the *same* index's pass, then after setting `dp[3] = true`
(using item `3` alone), a later forward step for `t = 6` (if it existed) could
read `dp[3]` and think "reach `3`, then add another `3`" — using `arr[1] = 3`
**twice** in the same subset, which is invalid for 0/1 knapsack-style problems
where each item may be used at most once. Iterating `t` from high to low
guarantees that when we compute `dp[t]` using `dp[t - arr[index]]`, the value
`dp[t - arr[index]]` still reflects the state **before** `arr[index]` was
considered in this pass — exactly mirroring the 2D table's read of the
*previous row* (`dp[index - 1, t - arr[index]]`), never the current row.
