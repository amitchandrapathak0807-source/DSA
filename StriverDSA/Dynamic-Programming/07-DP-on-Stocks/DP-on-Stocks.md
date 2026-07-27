# Dynamic Programming — DP on Stocks

## Concept: State Machine for Stock Problems

Almost every "Buy and Sell Stock" variant can be modeled as the **same underlying state machine**:

```
State = (day, canBuy/holding, transactionsLeft)
```

- **day** — the current index into the prices array (0-based). At each day we make one decision.
- **canBuy / holding** — a boolean flag telling us whether we are currently allowed to *buy* (we hold no stock) or must *sell* (we already hold one share). This is what forces the buy → sell → buy → sell ordering; you can never buy two shares before selling the first one.
- **transactionsLeft** — how many more "buy+sell pairs" (or, in some formulations, how many more individual buy/sell actions) we are still allowed to perform. Unbounded-transaction variants drop this dimension entirely; "at most 2" or "at most K" variants keep it.

At every day, from every state, we have (at most) two choices:

1. **Do nothing** — move to `day + 1` with the exact same `canBuy`/`transactionsLeft`, profit unchanged.
2. **Act** (buy if `canBuy == true`, sell if `canBuy == false`) — flip `canBuy`, move to `day + 1`, and either subtract `prices[day]` (buying) or add `prices[day]` (selling) to the running profit. Selling is also the point where a "transaction" is considered complete, so `transactionsLeft` is decremented there (convention used throughout this file).

The recurrence, in its most general form:

```
f(day, canBuy, k) =
    if canBuy:
        max( f(day+1, canBuy, k),                      // skip
             -prices[day] + f(day+1, !canBuy, k) )      // buy
    else:
        max( f(day+1, canBuy, k),                       // skip
             prices[day] + f(day+1, !canBuy, k-1) )      // sell, uses one transaction
```

Base case: when `day == n` (array exhausted) or `k == 0` (no transactions left), the best achievable profit is `0`.

Every problem below is just this state machine with extra constraints bolted on:

| Problem | Extra constraint on the state machine |
|---|---|
| I — One Transaction | `k` capped at 1 |
| II — Unlimited Transactions | `k` dimension dropped entirely (always allowed) |
| III — At Most Two Transactions | `k` capped at 2 |
| IV — At Most K Transactions | `k` is a parameter |
| Cooldown | after selling, `canBuy` cannot become `true` again until one extra day has passed |
| Transaction Fee | a fixed `fee` is subtracted whenever a transaction completes (on sell) |

Keeping this single mental model in mind, the four "levels" of solving each problem are always:

1. **Pure recursion** on `(day, canBuy, [k])` — exponential, explores overlapping subproblems.
2. **Memoization** — cache the recursion results in a 2D/3D array indexed by the same state.
3. **Tabulation** — build the same array bottom-up, iterating `day` from `n-1` down to `0` (or `0` up to `n-1`, matching the recursion's base case).
4. **Space optimization** — since `day` only ever depends on `day + 1`, keep just the "next day" row (and, for K-transaction variants, roll `k` into a small array), dropping O(n) space to O(1) or O(k).

---

## 1. Best Time to Buy and Sell Stock I (One Transaction — greedy/DP)

**Problem Statement:**
Given an array `prices` where `prices[i]` is the price of a stock on day `i`, find the maximum profit achievable by buying on one day and selling on a later day. You may complete **at most one transaction** (i.e., buy once and sell once). If no profit is possible, return `0`.

**Example:**
- Input: `prices = [7,1,5,3,6,4]`
- Output: `5`
- Explanation: Buy on day 1 (price = 1), sell on day 4 (price = 6), profit = `6 - 1 = 5`. You cannot buy before day 1 and sell after day 4 for a larger profit; also note you cannot sell before you buy.

**Approach 1 — Recursion (state: day, holding):**

**Logic (Steps):**
1. State: `Solve(day, canBuy)` = best profit achievable from `day` onward, given whether we're allowed to buy (`canBuy`) or must sell.
2. Base case: `day == prices.Length` → 0 (no more days, nothing left to do).
3. If `canBuy`, choose the max of skipping (`Solve(day+1, true)`) or buying (`-prices[day] + Solve(day+1, false)`).
4. If not `canBuy` (holding a share), since only one transaction is allowed, selling ends everything immediately — its value is just `prices[day]` (no further recursion), compared against skipping (`Solve(day+1, false)`).

```csharp
public class Solution
{
    public int MaxProfitI(int[] prices)
    {
        return Solve(0, true, prices);
    }

    private int Solve(int day, bool canBuy, int[] prices)
    {
        if (day == prices.Length) return 0;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, false, prices);
            int skip = Solve(day + 1, true, prices);
            return Math.Max(buy, skip);
        }
        else
        {
            // Only one transaction allowed, so after selling we're done: profit is final.
            int sell = prices[day];
            int skip = Solve(day + 1, false, prices);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n) — two branches per day. Space Complexity: O(n) recursion stack.

**Walkthrough:** With `prices = [7,1,5,3,6,4]`, from `Solve(0, true)` the recursion explores buying at every day and selling at every later day. The branch that buys at day 1 (price 1, `canBuy=true`→false) and later sells at day 4 (price 6, contributing `6` directly since it's the final action) yields `-1 + 6 = 5`. No other buy/sell pair beats this, so the top call returns `5`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion as Approach 1, keyed on `(day, canBuy)` since these two values fully determine the sub-problem.
2. Check `dp[day, canBuy]` (initialized to `-1`); return the cached value if not `-1`.
3. Otherwise compute `result` exactly as before (max of buy/skip, or sell-now/skip).
4. Store `result` in `dp[day, canBuy]` before returning.

```csharp
public class Solution
{
    public int MaxProfitI(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];
        for (int i = 0; i <= n; i++)
            for (int j = 0; j < 2; j++)
                dp[i, j] = -1;

        return Solve(0, 1, prices, dp);
    }

    private int Solve(int day, int canBuy, int[] prices, int[,] dp)
    {
        if (day == prices.Length) return 0;
        if (dp[day, canBuy] != -1) return dp[day, canBuy];

        int result;
        if (canBuy == 1)
        {
            int buy = -prices[day] + Solve(day + 1, 0, prices, dp);
            int skip = Solve(day + 1, 1, prices, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day];
            int skip = Solve(day + 1, 0, prices, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, canBuy] = result;
    }
}
```
Time Complexity: O(n * 2) states, O(1) work each -> O(n). Space Complexity: O(n) dp table + O(n) recursion stack.

**Walkthrough:** With `prices = [7,1,5,3,6,4]`, `dp[day, canBuy]` is filled the first time each state is reached; e.g., `dp[4, 0]` (holding, at day 4) is computed once as `max(6, dp[5,0]) = 6` and reused by every earlier day's recursion path that reaches day 4 while holding, instead of recomputing it. The top call `Solve(0, 1)` still returns `5`, matching the Output.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[day, canBuy]` = best profit from `day` onward given `canBuy` (1 = allowed to buy, 0 = holding).
2. Base case: `dp[n, *] = 0` (array default), matching the recursion's exhausted-days base case.
3. Transition (iterating `day` from `n-1` down to `0`): `dp[day,1] = max(-prices[day] + dp[day+1,0], dp[day+1,1])`; `dp[day,0] = max(prices[day], dp[day+1,0])` (selling ends the single transaction immediately, no further recursion added).
4. The answer is `dp[0, 1]` (start of the array, allowed to buy).

```csharp
public class Solution
{
    public int MaxProfitI(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];
        // dp[n, *] = 0 already by default

        for (int day = n - 1; day >= 0; day--)
        {
            // canBuy = 1
            int buy = -prices[day] + dp[day + 1, 0];
            int skipBuy = dp[day + 1, 1];
            dp[day, 1] = Math.Max(buy, skipBuy);

            // canBuy = 0
            int sell = prices[day];
            int skipSell = dp[day + 1, 0];
            dp[day, 0] = Math.Max(sell, skipSell);
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n). Space Complexity: O(n) for the dp table.

**Walkthrough:** With `prices = [7,1,5,3,6,4]` (n=6), filling backward: `dp[6,*]=0`; `dp[5,0]=max(4,0)=4`; `dp[4,0]=max(6,4)=6`, `dp[4,1]=max(-6+0,0)=0`; `dp[3,0]=max(3,6)=6`; `dp[2,0]=max(5,6)=6`; `dp[1,0]=max(1,6)=6`, `dp[1,1]=max(-1+6,dp[2,1])=5`; `dp[0,1]=max(-7+dp[1,0], dp[1,1])=max(-1,5)=5`. Final `dp[0,1] = 5`, matching the Output.

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: only the "day+1" row is kept, `aheadBuy`/`aheadNotBuy`, since `dp[day]` depends only on `dp[day+1]`.
2. Transition per day (iterating backward): `curBuy = max(-prices[day]+aheadNotBuy, aheadBuy)`; `curNotBuy = max(prices[day], aheadNotBuy)`.
3. After each day, roll `curBuy`/`curNotBuy` into `aheadBuy`/`aheadNotBuy`; after the loop `aheadBuy` is the answer.
4. Bonus greedy: track `minPrice` seen so far and `maxProfit = max(maxProfit, price - minPrice)` in a single left-to-right pass — mathematically equivalent since buying at the running minimum and selling at the current price maximizes profit for one transaction.

```csharp
public class Solution
{
    public int MaxProfitI(int[] prices)
    {
        int n = prices.Length;
        int aheadBuy = 0, aheadNotBuy = 0; // dp[day+1, 1] and dp[day+1, 0]
        int curBuy = 0, curNotBuy = 0;

        for (int day = n - 1; day >= 0; day--)
        {
            curBuy = Math.Max(-prices[day] + aheadNotBuy, aheadBuy);
            curNotBuy = Math.Max(prices[day], aheadNotBuy);

            aheadBuy = curBuy;
            aheadNotBuy = curNotBuy;
        }

        return aheadBuy;
    }

    // Bonus: the classic O(1)-space greedy that this whole DP collapses to
    // when only one transaction is allowed:
    public int MaxProfitGreedy(int[] prices)
    {
        int minPrice = int.MaxValue;
        int maxProfit = 0;
        foreach (int price in prices)
        {
            minPrice = Math.Min(minPrice, price);
            maxProfit = Math.Max(maxProfit, price - minPrice);
        }
        return maxProfit;
    }
}
```
Time Complexity: O(n) — single pass. Space Complexity: O(1) — only a fixed number of rolling variables are kept instead of the full `(n+1) x 2` table.

**Walkthrough:** Running the greedy version on `prices = [7,1,5,3,6,4]`: `minPrice` starts at `+inf`. Day 0 (price 7): `minPrice = 7`, profit `0`. Day 1 (price 1): `minPrice = 1`, profit stays `0`. Day 2 (price 5): profit `= 5 - 1 = 4`. Day 3 (price 3): profit stays `4`. Day 4 (price 6): profit `= 6 - 1 = 5`. Day 5 (price 4): profit stays `5`. Final answer `5`, matching the Output of buying on day 1 and selling on day 4.

---

## 2. Best Time to Buy and Sell Stock II (Unlimited Transactions)

**Problem Statement:**
Given `prices`, you may complete as many transactions as you like (buy one and sell one share of the stock multiple times), but you must sell the stock before you buy again (you cannot hold more than one share at a time). Find the maximum total profit.

**Example:**
- Input: `prices = [7,1,5,3,6,4]`
- Output: `7`
- Explanation: Buy on day 1 (price 1), sell on day 2 (price 5), profit `= 4`. Then buy on day 3 (price 3), sell on day 4 (price 6), profit `= 3`. Total profit `= 4 + 3 = 7`.

**Approach 1 — Recursion (state: day, holding):**

**Logic (Steps):**
1. State: `Solve(day, canBuy)` = best profit from `day` onward.
2. Base case: `day == prices.Length` → 0.
3. If `canBuy`, pick max of skip (`Solve(day+1, true)`) or buy (`-prices[day] + Solve(day+1, false)`).
4. If holding, since transactions are unlimited, selling immediately re-opens the buy option: `prices[day] + Solve(day+1, true)`, compared against skipping (`Solve(day+1, false)`).

```csharp
public class Solution
{
    public int MaxProfitII(int[] prices)
    {
        return Solve(0, true, prices);
    }

    private int Solve(int day, bool canBuy, int[] prices)
    {
        if (day == prices.Length) return 0;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, false, prices);
            int skip = Solve(day + 1, true, prices);
            return Math.Max(buy, skip);
        }
        else
        {
            // Unlimited transactions: after selling we can buy again immediately.
            int sell = prices[day] + Solve(day + 1, true, prices);
            int skip = Solve(day + 1, false, prices);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n). Space Complexity: O(n) recursion stack.

**Walkthrough:** With `prices = [7,1,5,3,6,4]`, the branch buying at day 1 (price 1) then selling at day 2 (price 5, `5 + Solve(3,true)`) followed by buying at day 3 (price 3) and selling at day 4 (price 6, `6 + Solve(5,true)`) accumulates `-1+5-3+6 = 7`. This branch beats all single-transaction alternatives, so the top call returns `7`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion as Approach 1, cached on `(day, canBuy)`.
2. Check `dp[day, canBuy]` (initialized to `-1`) first; return it if cached.
3. Otherwise compute `result` the same way (buy/skip, or sell-and-reopen/skip).
4. Store `result` in `dp[day, canBuy]` before returning.

```csharp
public class Solution
{
    public int MaxProfitII(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];
        for (int i = 0; i <= n; i++)
            for (int j = 0; j < 2; j++)
                dp[i, j] = -1;

        return Solve(0, 1, prices, dp);
    }

    private int Solve(int day, int canBuy, int[] prices, int[,] dp)
    {
        if (day == prices.Length) return 0;
        if (dp[day, canBuy] != -1) return dp[day, canBuy];

        int result;
        if (canBuy == 1)
        {
            int buy = -prices[day] + Solve(day + 1, 0, prices, dp);
            int skip = Solve(day + 1, 1, prices, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 1, 1, prices, dp);
            int skip = Solve(day + 1, 0, prices, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, canBuy] = result;
    }
}
```
Time Complexity: O(n). Space Complexity: O(n) dp table + O(n) recursion stack.

**Walkthrough:** With `prices = [7,1,5,3,6,4]`, every `(day, canBuy)` pair is computed once and cached; e.g. `dp[3,1]` (allowed to buy at day 3) is reused by any path reaching it, instead of recomputing. The top call `Solve(0,1)` still returns `7`, matching the Output, bounded to O(n) distinct states.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[day, canBuy]` as before, filled bottom-up.
2. Base case: `dp[n, *] = 0` (array default).
3. Transition (iterating `day` from `n-1` to `0`): `dp[day,1] = max(-prices[day]+dp[day+1,0], dp[day+1,1])`; `dp[day,0] = max(prices[day]+dp[day+1,1], dp[day+1,0])`.
4. Answer is `dp[0, 1]`.

```csharp
public class Solution
{
    public int MaxProfitII(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];

        for (int day = n - 1; day >= 0; day--)
        {
            dp[day, 1] = Math.Max(-prices[day] + dp[day + 1, 0], dp[day + 1, 1]);
            dp[day, 0] = Math.Max(prices[day] + dp[day + 1, 1], dp[day + 1, 0]);
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n). Space Complexity: O(n).

**Walkthrough:** With `prices = [7,1,5,3,6,4]` (n=6), filling backward: `dp[6,*]=0`; `dp[5,1]=max(-4,0)=0`, `dp[5,0]=max(4,0)=4`; `dp[4,1]=max(-6+4,0)=0`, `dp[4,0]=max(6+0,4)=6`; `dp[3,1]=max(-3+6,0)=3`, `dp[3,0]=max(3+0,6)=6`. Continuing this fill down through `dp[2,*]`, `dp[1,*]`, the final `dp[0,1] = 7`, matching the Output.

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: only `aheadBuy`/`aheadNotBuy` (row `day+1`) are kept.
2. Transition per day: `curBuy = max(-prices[day]+aheadNotBuy, aheadBuy)`; `curNotBuy = max(prices[day]+aheadBuy, aheadNotBuy)`.
3. Roll `cur*` into `ahead*` after each day; the answer is the final `aheadBuy`.
4. Bonus greedy: sum every positive day-to-day price increase — equivalent because unlimited transactions let the DP "capture" every local upward run of the price curve independently.

```csharp
public class Solution
{
    public int MaxProfitII(int[] prices)
    {
        int n = prices.Length;
        int aheadBuy = 0, aheadNotBuy = 0;

        for (int day = n - 1; day >= 0; day--)
        {
            int curBuy = Math.Max(-prices[day] + aheadNotBuy, aheadBuy);
            int curNotBuy = Math.Max(prices[day] + aheadBuy, aheadNotBuy);

            aheadBuy = curBuy;
            aheadNotBuy = curNotBuy;
        }

        return aheadBuy;
    }

    // Bonus: equivalent greedy — sum every positive day-to-day price increase.
    public int MaxProfitGreedy(int[] prices)
    {
        int profit = 0;
        for (int i = 1; i < prices.Length; i++)
            if (prices[i] > prices[i - 1])
                profit += prices[i] - prices[i - 1];
        return profit;
    }
}
```
Time Complexity: O(n). Space Complexity: O(1).

**Walkthrough:** On `prices = [7,1,5,3,6,4]`, the greedy sum-of-positive-differences view: `1->5` gain `4`, `5->3` no gain, `3->6` gain `3`, `6->4` no gain. Total `4 + 3 = 7`, matching the Output and the rolling-DP result (buy@1/sell@5, buy@3/sell@6).

---

## 3. Best Time to Buy and Sell Stock III (At Most Two Transactions)

**Problem Statement:**
Given `prices`, find the maximum profit you can achieve with **at most two transactions**. You must sell the stock before you buy again.

**Example:**
- Input: `prices = [3,3,5,0,0,3,1,4]`
- Output: `6`
- Explanation: Buy on day 3 (price 0), sell on day 5 (price 3), profit `= 3`. Then buy on day 6 (price 1), sell on day 7 (price 4), profit `= 3`. Total `= 6`.

**Approach 1 — Recursion (state: day, transactionNumber 1-4, holding via parity):**

**Logic (Steps):**
1. State: `Solve(day, transactionCount)`, where `transactionCount` runs `1(buy1) -> 2(sell1) -> 3(buy2) -> 4(sell2) -> 5(done)`; its parity tells us whether we're at a buy step (odd) or sell step (even).
2. Base case: `day == prices.Length` or `transactionCount == 5` (all allowed actions used) → 0.
3. At a buy step, choose max of buying (`-prices[day] + Solve(day+1, transactionCount+1)`) or skipping (`Solve(day+1, transactionCount)`).
4. At a sell step, choose max of selling (`prices[day] + Solve(day+1, transactionCount+1)`) or skipping.

```csharp
public class Solution
{
    public int MaxProfitIII(int[] prices)
    {
        // transactionCount goes 1(buy1) -> 2(sell1) -> 3(buy2) -> 4(sell2) -> 5(done)
        return Solve(0, 1, prices);
    }

    private int Solve(int day, int transactionCount, int[] prices)
    {
        if (day == prices.Length || transactionCount == 5) return 0;

        bool canBuy = transactionCount % 2 == 1; // odd -> buy step, even -> sell step

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, transactionCount + 1, prices);
            int skip = Solve(day + 1, transactionCount, prices);
            return Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 1, transactionCount + 1, prices);
            int skip = Solve(day + 1, transactionCount, prices);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n) worst case. Space Complexity: O(n) recursion stack.

**Walkthrough:** With `prices = [3,3,5,0,0,3,1,4]`, the branch that skips until day 3 (buy1, price 0, `transactionCount 1->2`), sells at day 5 (price 3, `2->3`), buys at day 6 (price 1, `3->4`), and sells at day 7 (price 4, `4->5`) accumulates `-0+3-1+4 = 6`. No branch beats this, so `Solve(0,1)` returns `6`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion, cached on `(day, transactionCount)`.
2. Check `dp[day, transactionCount]` (initialized `-1`); return it if cached.
3. Otherwise compute `result` the same way based on the parity of `transactionCount`.
4. Store `result` before returning.

```csharp
public class Solution
{
    public int MaxProfitIII(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 5];
        for (int i = 0; i <= n; i++)
            for (int j = 0; j < 5; j++)
                dp[i, j] = -1;

        return Solve(0, 1, prices, dp);
    }

    private int Solve(int day, int transactionCount, int[] prices, int[,] dp)
    {
        if (day == prices.Length || transactionCount == 5) return 0;
        if (dp[day, transactionCount] != -1) return dp[day, transactionCount];

        bool canBuy = transactionCount % 2 == 1;
        int result;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, transactionCount + 1, prices, dp);
            int skip = Solve(day + 1, transactionCount, prices, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 1, transactionCount + 1, prices, dp);
            int skip = Solve(day + 1, transactionCount, prices, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, transactionCount] = result;
    }
}
```
Time Complexity: O(n * 5) -> O(n). Space Complexity: O(n) dp + O(n) recursion stack.

**Walkthrough:** With `prices = [3,3,5,0,0,3,1,4]`, each `(day, transactionCount)` pair (40 total states) is computed once and cached; a state like `(6, 3)` reached via several skip sequences is resolved once. `Solve(0,1)` still returns `6`, matching the Output, using O(5n) states.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[day, t]` for `t` in `1..4`, same meaning as the recursion.
2. Base case: `dp[n, *] = 0` (array default) and `dp[*, 5]` unused/implicitly 0.
3. Transition (iterating `day` from `n-1` to `0`, and for each day iterating `t` from `4` down to `1`): buy step (`t` odd) — `dp[day,t] = max(-prices[day]+dp[day+1,t+1], dp[day+1,t])`; sell step (`t` even) — `dp[day,t] = max(prices[day]+dp[day+1,t+1], dp[day+1,t])`.
4. Answer is `dp[0, 1]`.

```csharp
public class Solution
{
    public int MaxProfitIII(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 5];

        for (int day = n - 1; day >= 0; day--)
        {
            for (int t = 4; t >= 1; t--)
            {
                bool canBuy = t % 2 == 1;
                if (canBuy)
                {
                    int buy = -prices[day] + dp[day + 1, t + 1];
                    int skip = dp[day + 1, t];
                    dp[day, t] = Math.Max(buy, skip);
                }
                else
                {
                    int sell = prices[day] + dp[day + 1, t + 1];
                    int skip = dp[day + 1, t];
                    dp[day, t] = Math.Max(sell, skip);
                }
            }
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n * 4) -> O(n). Space Complexity: O(n).

**Walkthrough:** With `prices = [3,3,5,0,0,3,1,4]` (n=8), filling backward through all `(day, t)` cells with the transitions above eventually converges at `dp[0,1] = 6`, matching the Output — the same value the recursion found via buy@3/sell@5/buy@6/sell@7.

---

**Approach 4 — Space Optimization (four-variable running technique):**
```csharp
public class Solution
{
    public int MaxProfitIII(int[] prices)
    {
        int buy1 = int.MinValue, sell1 = 0;
        int buy2 = int.MinValue, sell2 = 0;

        foreach (int price in prices)
        {
            buy1 = Math.Max(buy1, -price);            // best profit after 1st buy
            sell1 = Math.Max(sell1, buy1 + price);     // best profit after 1st sell
            buy2 = Math.Max(buy2, sell1 - price);      // best profit after 2nd buy (funded by 1st sell)
            sell2 = Math.Max(sell2, buy2 + price);      // best profit after 2nd sell
        }

        return sell2;
    }
}
```
Time Complexity: O(n) — single left-to-right pass. Space Complexity: O(1) — only four running scalars replace the `(n+1) x 5` table.

**Walkthrough:** With `buy1 = -inf, sell1 = 0, buy2 = -inf, sell2 = 0` initially, on `prices = [3,3,5,0,0,3,1,4]`:

| price | buy1 = max(buy1, -price) | sell1 = max(sell1, buy1+price) | buy2 = max(buy2, sell1-price) | sell2 = max(sell2, buy2+price) |
|---|---|---|---|---|
| 3 | -3 | 0 | -3 | 0 |
| 3 | -3 | 0 | -3 | 0 |
| 5 | -3 | 2 | -3 | 2 |
| 0 | 0 | 2 | 2 | 2 |
| 0 | 0 | 2 | 2 | 2 |
| 3 | 0 | 3 | 2 | 5 |
| 1 | 0 | 3 | 2 | 5 |
| 4 | 0 | 3 | 2 | 6 |

Final `sell2 = 6`, matching the Output. This works because `buy1`/`sell1` track the best profit after the first buy/sell (restricted to `transactionCount ∈ {1,2}`) and `buy2`/`sell2` track the second (restricted to `{3,4}`), each update folding "skip vs act" into a single running max, e.g. `max(buy1_prev, -price)` already encodes "keep the old buy or act today."

---

## 4. Best Time to Buy and Sell Stock IV (At Most K Transactions)

**Problem Statement:**
Given `prices` and an integer `k`, find the maximum profit you can achieve with **at most `k` transactions**.

**Example:**
- Input: `k = 2, prices = [3,3,5,0,0,3,1,4]`
- Output: `6`
- Explanation: Same as Problem 3 with `k = 2` — buy day 3 (0) / sell day 5 (3) = 3, buy day 6 (1) / sell day 7 (4) = 3, total `6`.

**Approach 1 — Recursion (state: day, transactionNumber 1..2k, holding via parity):**

**Logic (Steps):**
1. State: `Solve(day, transactionCount)`, generalizing Problem 3 so `transactionCount` runs from `1` to `2k` (instead of a fixed `4`), with parity again distinguishing buy/sell steps.
2. Base case: `day == prices.Length` or `transactionCount == 2k+1` (all `k` buy/sell pairs used) → 0.
3. At a buy step, max of buying (`-prices[day] + Solve(day+1, transactionCount+1)`) or skipping.
4. At a sell step, max of selling (`prices[day] + Solve(day+1, transactionCount+1)`) or skipping.

```csharp
public class Solution
{
    public int MaxProfitIV(int k, int[] prices)
    {
        return Solve(0, 1, k, prices);
    }

    private int Solve(int day, int transactionCount, int k, int[] prices)
    {
        if (day == prices.Length || transactionCount == 2 * k + 1) return 0;

        bool canBuy = transactionCount % 2 == 1;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, transactionCount + 1, k, prices);
            int skip = Solve(day + 1, transactionCount, k, prices);
            return Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 1, transactionCount + 1, k, prices);
            int skip = Solve(day + 1, transactionCount, k, prices);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n) worst case. Space Complexity: O(n) recursion stack.

**Walkthrough:** With `k=2`, `prices = [3,3,5,0,0,3,1,4]`, this is structurally identical to Problem 3's recursion with `2k+1 = 5` as the stopping bound. The branch buy@3(price 0)/sell@5(price 3)/buy@6(price 1)/sell@7(price 4) again yields `-0+3-1+4=6`, so `Solve(0,1)` returns `6`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion, cached on `(day, transactionCount)` in a `(n+1) x (2k+2)` table.
2. Check `dp[day, transactionCount]` (initialized `-1`); return cached value if present.
3. Otherwise compute `result` as in Approach 1.
4. Store `result` before returning.

```csharp
public class Solution
{
    public int MaxProfitIV(int k, int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2 * k + 2];
        for (int i = 0; i <= n; i++)
            for (int j = 0; j < 2 * k + 2; j++)
                dp[i, j] = -1;

        return Solve(0, 1, k, prices, dp);
    }

    private int Solve(int day, int transactionCount, int k, int[] prices, int[,] dp)
    {
        if (day == prices.Length || transactionCount == 2 * k + 1) return 0;
        if (dp[day, transactionCount] != -1) return dp[day, transactionCount];

        bool canBuy = transactionCount % 2 == 1;
        int result;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, transactionCount + 1, k, prices, dp);
            int skip = Solve(day + 1, transactionCount, k, prices, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 1, transactionCount + 1, k, prices, dp);
            int skip = Solve(day + 1, transactionCount, k, prices, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, transactionCount] = result;
    }
}
```
Time Complexity: O(n * 2k) -> O(n*k). Space Complexity: O(n*k) dp + O(n) recursion stack.

**Walkthrough:** With `k=2`, `prices = [3,3,5,0,0,3,1,4]`, each of the O(n·2k) = O(8·5) states is computed once and cached, e.g. `dp[6,3]` (day 6, on the second buy step) is reused across any path that reaches it. `Solve(0,1)` returns `6`, matching the Output.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[day, t]` for `t` in `1..2k`.
2. Base case: `dp[n, *] = 0`.
3. Transition (iterating `day` from `n-1` to `0`, and `t` from `2k` down to `1`): buy step (`t` odd) — `dp[day,t] = max(-prices[day]+dp[day+1,t+1], dp[day+1,t])`; sell step (`t` even) — `dp[day,t] = max(prices[day]+dp[day+1,t+1], dp[day+1,t])`.
4. Answer is `dp[0, 1]`.

```csharp
public class Solution
{
    public int MaxProfitIV(int k, int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2 * k + 2];

        for (int day = n - 1; day >= 0; day--)
        {
            for (int t = 2 * k; t >= 1; t--)
            {
                bool canBuy = t % 2 == 1;
                if (canBuy)
                {
                    int buy = -prices[day] + dp[day + 1, t + 1];
                    int skip = dp[day + 1, t];
                    dp[day, t] = Math.Max(buy, skip);
                }
                else
                {
                    int sell = prices[day] + dp[day + 1, t + 1];
                    int skip = dp[day + 1, t];
                    dp[day, t] = Math.Max(sell, skip);
                }
            }
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n * k). Space Complexity: O(n * k).

**Walkthrough:** With `k=2`, `prices = [3,3,5,0,0,3,1,4]`, filling the `(n+1) x 5` table backward reproduces the exact same values as Problem 3's `dp` table (since `2k=4` matches Problem 3's fixed `4`), converging to `dp[0,1] = 6`, matching the Output.

---

**Approach 4 — Space Optimization (2*K-sized running array):**
**Logic (Steps):**
1. State: a `2k`-sized array `transactions`, where `transactions[2i]` = best profit after the `(i+1)`-th buy, `transactions[2i+1]` = best profit after the `(i+1)`-th sell — generalizing Problem 3's four scalars (`buy1, sell1, buy2, sell2`) to `k` pairs.
2. Base case: buy slots (`even` indices) start at `int.MinValue`; sell slots start at 0 (array default).
3. Transition per price (left-to-right single pass): slot `0` updates as `max(transactions[0], -price)`; each later slot `i` updates from slot `i-1` — odd slots (sell) add `price`, even slots (buy) subtract `price`.
4. After processing all prices, `transactions[2k-1]` (the last sell slot) holds the best profit using at most `k` completed buy/sell pairs.

```csharp
public class Solution
{
    public int MaxProfitIV(int k, int[] prices)
    {
        if (k == 0 || prices.Length == 0) return 0;

        // transactions[2*i]   = best profit after (i+1)-th buy
        // transactions[2*i+1] = best profit after (i+1)-th sell
        int[] transactions = new int[2 * k];
        for (int i = 0; i < 2 * k; i += 2)
            transactions[i] = int.MinValue; // buy slots start at -infinity

        foreach (int price in prices)
        {
            transactions[0] = Math.Max(transactions[0], -price);
            for (int i = 1; i < 2 * k; i++)
            {
                if (i % 2 == 1)
                    // sell slot i: funded by buy slot i-1
                    transactions[i] = Math.Max(transactions[i], transactions[i - 1] + price);
                else
                    // buy slot i: funded by previous sell slot i-1
                    transactions[i] = Math.Max(transactions[i], transactions[i - 1] - price);
            }
        }

        return transactions[2 * k - 1];
    }
}
```
Time Complexity: O(n * k) — for every price we update `2k` running values. Space Complexity: O(k) — only the `2k`-sized array is kept, no day dimension.

**Walkthrough:** With `k=2`, `prices = [3,3,5,0,0,3,1,4]`, `transactions[0..3]` play exactly the role of `buy1, sell1, buy2, sell2` from Problem 3's dry run table, so running the same left-to-right pass produces the same final values, giving `transactions[3] = 6`, matching the Output.

---

## 5. Best Time to Buy and Sell Stock with Cooldown

**Problem Statement:**
Given `prices`, find the maximum profit with unlimited transactions, subject to the constraint that **after you sell a stock you cannot buy again on the very next day** — you must wait (cooldown) for one day before your next buy.

**Example:**
- Input: `prices = [1,2,3,0,2]`
- Output: `3`
- Explanation: Buy on day 0 (price 1), sell on day 1 (price 2), profit `1`. Cooldown day 2. Buy on day 3 (price 0), sell on day 4 (price 2), profit `2`. Total `= 3`.
  (Note: without cooldown you could buy day 3 immediately after selling day... but here the important detail is the transaction sequence buy@0/sell@1 forces a cooldown at day 2 before the next buy is allowed at day 3.)

**Approach 1 — Recursion (state: day, holding — with an explicit "just sold" transition):**

**Logic (Steps):**
1. State: `Solve(day, canBuy)`, same shape as the unlimited-transaction problem.
2. Base case: `day >= prices.Length` → 0.
3. If `canBuy`, max of buy (`-prices[day] + Solve(day+1, false)`) or skip (`Solve(day+1, true)`) — unchanged from Problem 2.
4. If holding, selling encodes the cooldown directly in the transition: jump to `Solve(day+2, true)` (skipping the very next day) instead of `day+1`, compared against skipping (`Solve(day+1, false)`).

```csharp
public class Solution
{
    public int MaxProfitCooldown(int[] prices)
    {
        return Solve(0, true, prices);
    }

    private int Solve(int day, bool canBuy, int[] prices)
    {
        if (day >= prices.Length) return 0;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, false, prices);
            int skip = Solve(day + 1, true, prices);
            return Math.Max(buy, skip);
        }
        else
        {
            // Selling jumps to day + 2 (skips the cooldown day) instead of day + 1.
            int sell = prices[day] + Solve(day + 2, true, prices);
            int skip = Solve(day + 1, false, prices);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n). Space Complexity: O(n) recursion stack.

**Walkthrough:** With `prices = [1,2,3,0,2]`, the branch buying day 0 (price 1) and selling day 1 (price 2, jumping to `Solve(3,true)` due to cooldown) then buying day 3 (price 0) and selling day 4 (price 2) accumulates `-1+2-0+2 = 3`. This beats other branches (e.g. waiting for the day-2 peak of 3, which leaves no time for a profitable second leg), so `Solve(0,true)` returns `3`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion, cached on `(day, canBuy)`, but the table is sized `n+2` to accommodate the `day+2` lookahead on sell without an out-of-bounds index.
2. Check `dp[day, canBuy]` (initialized `-1`); return cached value if present.
3. Otherwise compute `result` the same way (buy/skip, or sell-with-cooldown/skip).
4. Store `result` before returning.

```csharp
public class Solution
{
    public int MaxProfitCooldown(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 2, 2];
        for (int i = 0; i < n + 2; i++)
            for (int j = 0; j < 2; j++)
                dp[i, j] = -1;

        return Solve(0, 1, prices, dp);
    }

    private int Solve(int day, int canBuy, int[] prices, int[,] dp)
    {
        if (day >= prices.Length) return 0;
        if (dp[day, canBuy] != -1) return dp[day, canBuy];

        int result;
        if (canBuy == 1)
        {
            int buy = -prices[day] + Solve(day + 1, 0, prices, dp);
            int skip = Solve(day + 1, 1, prices, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] + Solve(day + 2, 1, prices, dp);
            int skip = Solve(day + 1, 0, prices, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, canBuy] = result;
    }
}
```
Time Complexity: O(n). Space Complexity: O(n) dp (sized `n+2` to absorb the `day+2` lookahead) + O(n) recursion stack.

**Walkthrough:** With `prices = [1,2,3,0,2]`, `dp[1,0]` (holding at day 1) is computed once as `max(2+dp[3,1], dp[2,0])` and reused by any earlier-day path that reaches it. `Solve(0,1)` still returns `3`, matching the Output.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[day, canBuy]`, table sized `n+2` rows.
2. Base case: `dp[n,*] = dp[n+1,*] = 0` (array default).
3. Transition (iterating `day` from `n-1` to `0`): `dp[day,1] = max(-prices[day]+dp[day+1,0], dp[day+1,1])`; `dp[day,0] = max(prices[day]+dp[day+2,1], dp[day+1,0])` — the sell case jumps two rows ahead to enforce the cooldown.
4. Answer is `dp[0, 1]`.

```csharp
public class Solution
{
    public int MaxProfitCooldown(int[] prices)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 2, 2]; // extra rows for the day+2 jump on sell

        for (int day = n - 1; day >= 0; day--)
        {
            dp[day, 1] = Math.Max(-prices[day] + dp[day + 1, 0], dp[day + 1, 1]);
            dp[day, 0] = Math.Max(prices[day] + dp[day + 2, 1], dp[day + 1, 0]);
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n). Space Complexity: O(n).

**Walkthrough:** With `prices = [1,2,3,0,2]` (n=5), filling backward from `dp[5,*]=dp[6,*]=0`: `dp[4,1]=max(-2,0)=0`, `dp[4,0]=max(2+0,0)=2`; `dp[3,1]=max(-0+2,0)=2`, `dp[3,0]=max(0+0,2)=2`; `dp[2,1]=max(-3+2,2)=2`, `dp[2,0]=max(3+0,2)=3`; `dp[1,1]=max(-2+3,2)=2`, `dp[1,0]=max(2+2,3)=4`; `dp[0,1]=max(-1+4,2)=3`. Final `dp[0,1]=3`, matching the Output.

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: since selling needs `dp[day+2]`, two trailing rows are kept — `front1` (day+1) and `front2` (day+2) — instead of just one.
2. Transition per day: `curBuy = max(-prices[day]+front1NotBuy, front1Buy)`; `curNotBuy = max(prices[day]+front2Buy, front1NotBuy)` (the cooldown jump uses `front2Buy`, i.e. `dp[day+2,1]`).
3. Shift the window after each day: `front2* <- front1*`, `front1* <- cur*`.
4. After the loop, `front1Buy` (which has rolled down to represent `dp[0,1]`) is the answer.

```csharp
public class Solution
{
    public int MaxProfitCooldown(int[] prices)
    {
        int n = prices.Length;
        // "front1" = day+1 row, "front2" = day+2 row
        int front1Buy = 0, front1NotBuy = 0;
        int front2Buy = 0, front2NotBuy = 0; // unused until we go two steps back, init 0 for base case day==n and n+1

        for (int day = n - 1; day >= 0; day--)
        {
            int curBuy = Math.Max(-prices[day] + front1NotBuy, front1Buy);
            int curNotBuy = Math.Max(prices[day] + front2Buy, front1NotBuy);

            // shift window: day+2 <- day+1, day+1 <- day
            front2Buy = front1Buy;
            front2NotBuy = front1NotBuy;
            front1Buy = curBuy;
            front1NotBuy = curNotBuy;
        }

        return front1Buy;
    }
}
```
Time Complexity: O(n). Space Complexity: O(1) — four rolling scalars replace the `(n+2) x 2` table.

**Walkthrough:** With `prices = [1,2,3,0,2]`, the rolling window reproduces the exact same values as Approach 3's table (`dp[4,*]=(0,2)`, `dp[3,*]=(2,2)`, `dp[2,*]=(2,3)`, `dp[1,*]=(2,4)`, `dp[0,1]=3`) since `front1`/`front2` track `dp[day+1]`/`dp[day+2]` at every step. Final `front1Buy = 3`, matching the Output — buy@0/sell@1 (profit 1), cooldown@2, buy@3/sell@4 (profit 2), total `3`.

---

## 6. Best Time to Buy and Sell Stock with Transaction Fee

**Problem Statement:**
Given `prices` and an integer `fee`, find the maximum profit with unlimited transactions, where each completed transaction (one buy + one sell) incurs a fixed transaction `fee`, charged once per transaction (commonly modeled as being subtracted at the sell step).

**Example:**
- Input: `prices = [1,3,2,8,4,9], fee = 2`
- Output: `8`
- Explanation: Buy on day 0 (price 1), sell on day 3 (price 8), profit `= 8 - 1 - 2 = 5`. Then buy on day 4 (price 4), sell on day 5 (price 9), profit `= 9 - 4 - 2 = 3`. Total `= 5 + 3 = 8`.

**Approach 1 — Recursion (state: day, holding):**

**Logic (Steps):**
1. State: `Solve(day, canBuy)`, same shape as the unlimited-transaction problem.
2. Base case: `day == prices.Length` → 0.
3. If `canBuy`, max of buy (`-prices[day] + Solve(day+1, false)`) or skip.
4. If holding, selling now subtracts the `fee` once (deducted at sell time): `prices[day] - fee + Solve(day+1, true)`, compared against skipping.

```csharp
public class Solution
{
    public int MaxProfitFee(int[] prices, int fee)
    {
        return Solve(0, true, prices, fee);
    }

    private int Solve(int day, bool canBuy, int[] prices, int fee)
    {
        if (day == prices.Length) return 0;

        if (canBuy)
        {
            int buy = -prices[day] + Solve(day + 1, false, prices, fee);
            int skip = Solve(day + 1, true, prices, fee);
            return Math.Max(buy, skip);
        }
        else
        {
            // Fee is charged once per completed transaction, deducted at sell time.
            int sell = prices[day] - fee + Solve(day + 1, true, prices, fee);
            int skip = Solve(day + 1, false, prices, fee);
            return Math.Max(sell, skip);
        }
    }
}
```
Time Complexity: O(2^n). Space Complexity: O(n) recursion stack.

**Walkthrough:** With `prices = [1,3,2,8,4,9]`, `fee = 2`, the branch buying day 0 (price 1) and selling day 3 (price 8, `8-2 + Solve(4,true)`) then buying day 4 (price 4) and selling day 5 (price 9, `9-2`) accumulates `-1+6-4+7 = 8`. This beats splitting into more, fee-eroded transactions, so `Solve(0,true)` returns `8`, matching the Output.

---

**Approach 2 — Memoization:**
```csharp
public class Solution
{
    public int MaxProfitFee(int[] prices, int fee)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];
        for (int i = 0; i <= n; i++)
            for (int j = 0; j < 2; j++)
                dp[i, j] = -1;

        return Solve(0, 1, prices, fee, dp);
    }

    private int Solve(int day, int canBuy, int[] prices, int fee, int[,] dp)
    {
        if (day == prices.Length) return 0;
        if (dp[day, canBuy] != -1) return dp[day, canBuy];

        int result;
        if (canBuy == 1)
        {
            int buy = -prices[day] + Solve(day + 1, 0, prices, fee, dp);
            int skip = Solve(day + 1, 1, prices, fee, dp);
            result = Math.Max(buy, skip);
        }
        else
        {
            int sell = prices[day] - fee + Solve(day + 1, 1, prices, fee, dp);
            int skip = Solve(day + 1, 0, prices, fee, dp);
            result = Math.Max(sell, skip);
        }

        return dp[day, canBuy] = result;
    }
}
```
Time Complexity: O(n). Space Complexity: O(n) dp + O(n) recursion stack.

**Approach 3 — Tabulation:**
```csharp
public class Solution
{
    public int MaxProfitFee(int[] prices, int fee)
    {
        int n = prices.Length;
        int[,] dp = new int[n + 1, 2];

        for (int day = n - 1; day >= 0; day--)
        {
            dp[day, 1] = Math.Max(-prices[day] + dp[day + 1, 0], dp[day + 1, 1]);
            dp[day, 0] = Math.Max(prices[day] - fee + dp[day + 1, 1], dp[day + 1, 0]);
        }

        return dp[0, 1];
    }
}
```
Time Complexity: O(n). Space Complexity: O(n).

**Approach 4 — Space Optimization:**
```csharp
public class Solution
{
    public int MaxProfitFee(int[] prices, int fee)
    {
        int n = prices.Length;
        int aheadBuy = 0, aheadNotBuy = 0;

        for (int day = n - 1; day >= 0; day--)
        {
            int curBuy = Math.Max(-prices[day] + aheadNotBuy, aheadBuy);
            int curNotBuy = Math.Max(prices[day] - fee + aheadBuy, aheadNotBuy);

            aheadBuy = curBuy;
            aheadNotBuy = curNotBuy;
        }

        return aheadBuy;
    }

    // Bonus: equivalent classic left-to-right O(1)-space greedy formulation.
    public int MaxProfitFeeGreedy(int[] prices, int fee)
    {
        int cash = 0;                 // best profit currently holding no stock
        int hold = -prices[0];        // best profit currently holding a stock
        for (int i = 1; i < prices.Length; i++)
        {
            cash = Math.Max(cash, hold + prices[i] - fee);
            hold = Math.Max(hold, cash - prices[i]);
        }
        return cash;
    }
}
```
Time Complexity: O(n). Space Complexity: O(1).

**Explanation:**
Dry run of the left-to-right `cash`/`hold` formulation on `prices = [1,3,2,8,4,9], fee = 2`:

Initialize `hold = -1, cash = 0` (from `prices[0] = 1`).

| day | price | cash = max(cash, hold+price-fee) | hold = max(hold, cash-price) |
|---|---|---|---|
| 1 | 3 | max(0, -1+3-2)=0 | max(-1, 0-3)=-1 |
| 2 | 2 | max(0, -1+2-2)=0 | max(-1, 0-2)=-1 |
| 3 | 8 | max(0, -1+8-2)=5 | max(-1, 5-8)=-1 |
| 4 | 4 | max(5, -1+4-2)=5 | max(-1, 5-4)=1 |
| 5 | 9 | max(5, 1+9-2)=8 | max(1, 8-9)=1 |

Final `cash = 8`, matching buy@0/sell@3 (profit `8-1-2=5`) plus buy@4/sell@5 (profit `9-4-2=3`), total `8`. The fee only ever gets subtracted once per sell, so it never double-charges a single buy/sell pair, and the DP naturally re-buys the moment it's profitable net of the fee (here, at day 4 with `hold` becoming `1`, better than staying at `-1`).
