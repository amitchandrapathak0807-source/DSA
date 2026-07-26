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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
Running the greedy/rolling approach on `prices = [7,1,5,3,6,4]`:
`minPrice` starts at `+inf`. Day 0 (price 7): `minPrice = 7`, profit `0`. Day 1 (price 1): `minPrice = 1`, profit stays `0`. Day 2 (price 5): profit `= 5 - 1 = 4`. Day 3 (price 3): profit stays `4`. Day 4 (price 6): profit `= 6 - 1 = 5`. Day 5 (price 4): profit stays `5`. Final answer `5`, matching buying on day 1 and selling on day 4.

---

## 2. Best Time to Buy and Sell Stock II (Unlimited Transactions)

**Problem Statement:**
Given `prices`, you may complete as many transactions as you like (buy one and sell one share of the stock multiple times), but you must sell the stock before you buy again (you cannot hold more than one share at a time). Find the maximum total profit.

**Example:**
- Input: `prices = [7,1,5,3,6,4]`
- Output: `7`
- Explanation: Buy on day 1 (price 1), sell on day 2 (price 5), profit `= 4`. Then buy on day 3 (price 3), sell on day 4 (price 6), profit `= 3`. Total profit `= 4 + 3 = 7`.

**Approach 1 — Recursion (state: day, holding):**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
On `prices = [7,1,5,3,6,4]`, the greedy sum-of-positive-differences view: `1->5` gain `4`, `5->3` no gain, `3->6` gain `3`, `6->4` no gain. Total `4 + 3 = 7`, exactly matching the two-transaction DP result (buy@1/sell@5, buy@3/sell@6). This equivalence holds because unlimited transactions let us "capture" every local upward run of the price curve independently.

---

## 3. Best Time to Buy and Sell Stock III (At Most Two Transactions)

**Problem Statement:**
Given `prices`, find the maximum profit you can achieve with **at most two transactions**. You must sell the stock before you buy again.

**Example:**
- Input: `prices = [3,3,5,0,0,3,1,4]`
- Output: `6`
- Explanation: Buy on day 3 (price 0), sell on day 5 (price 3), profit `= 3`. Then buy on day 6 (price 1), sell on day 7 (price 4), profit `= 3`. Total `= 6`.

**Approach 1 — Recursion (state: day, transactionNumber 1-4, holding via parity):**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Explanation (dry run — four-variable technique on `prices = [3,3,5,0,0,3,1,4]`):**

Initialize `buy1 = -inf, sell1 = 0, buy2 = -inf, sell2 = 0`.

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

Final `sell2 = 6`. This matches the equivalent `(day, transactionCount, holding)` DP: `buy1`/`sell1` are exactly `dp[day, canBuy=1/0]` restricted to `transactionCount ∈ {1,2}` (the first transaction), and `buy2`/`sell2` are the same restricted to `transactionCount ∈ {3,4}` (the second transaction), except tracked as running maxima instead of a per-day table — `buy1` at price `p` equals `max` over all days up to and including the current one of `-prices[day]`, i.e., it folds the "skip vs act" choice into a single running max update, since `max(buy1_prev, -price)` already encodes "keep the old buy or act today."

---

## 4. Best Time to Buy and Sell Stock IV (At Most K Transactions)

**Problem Statement:**
Given `prices` and an integer `k`, find the maximum profit you can achieve with **at most `k` transactions**.

**Example:**
- Input: `k = 2, prices = [3,3,5,0,0,3,1,4]`
- Output: `6`
- Explanation: Same as Problem 3 with `k = 2` — buy day 3 (0) / sell day 5 (3) = 3, buy day 6 (1) / sell day 7 (4) = 3, total `6`.

**Approach 1 — Recursion (state: day, transactionNumber 1..2k, holding via parity):**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization (2*K-sized running array):**
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

**Explanation:**
This generalizes Problem 3's four-variable trick (`buy1, sell1, buy2, sell2`) directly: those four scalars are exactly `transactions[0..3]` when `k = 2`. Each pair `(transactions[2i], transactions[2i+1])` plays the role of one `(buyJ, sellJ)` pair — `buy_{i+1}` is funded from `sell_i` (or from `0`/nothing for `i = 0`) exactly as `buy2` was funded from `sell1` in Problem 3, and `sell_{i+1}` is funded from `buy_{i+1}`. Running the loop left-to-right over prices, `transactions[2*k-1]` (the last sell slot) accumulates the best profit using at most `k` completed buy/sell pairs, generalizing the `sell2` answer of Problem 3 to `sellK`. Verifying on the example with `k = 2` reproduces the same table as the dry run in Problem 3, giving `transactions[3] = 6`.

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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
The recursion needs two distinct "no stock in hand" flavors baked into its transitions because a plain `canBuy` boolean alone can't express "sold yesterday, must skip today." Instead of adding a third explicit state, the trick used above encodes the cooldown directly in the *transition*: when we sell (`canBuy == false` branch, sell action), we jump straight to `Solve(day + 2, true, ...)` rather than `Solve(day + 1, true, ...)`, which skips the very next day for the "can buy again" transition while other transitions (`skip`, `buy`) still advance normally by one day. This is why the tabulation needs `n + 2` rows and why the space-optimized version keeps **two** trailing rows (`front1` for `day+1`, `front2` for `day+2`) instead of just one.

Dry run on `prices = [1,2,3,0,2]` (n = 5), tabulating from `day = 4` down to `day = 0`, with `dp[5,*] = dp[6,*] = 0`:

- day 4 (price 2): `dp[4,1] = max(-2+dp[5,0], dp[5,1]) = max(-2,0) = 0`. `dp[4,0] = max(2+dp[6,1], dp[5,0]) = max(2,0) = 2`.
- day 3 (price 0): `dp[3,1] = max(-0+dp[4,0], dp[4,1]) = max(2,0) = 2`. `dp[3,0] = max(0+dp[5,1], dp[4,0]) = max(0,2) = 2`.
- day 2 (price 3): `dp[2,1] = max(-3+dp[3,0], dp[3,1]) = max(-1,2) = 2`. `dp[2,0] = max(3+dp[4,1], dp[3,0]) = max(3,2) = 3`.
- day 1 (price 2): `dp[1,1] = max(-2+dp[2,0], dp[2,1]) = max(1,2) = 2`. `dp[1,0] = max(2+dp[3,1], dp[2,0]) = max(4,3) = 4`.
- day 0 (price 1): `dp[0,1] = max(-1+dp[1,0], dp[1,1]) = max(3,2) = 3`.

Final answer `dp[0,1] = 3`, matching buy@0/sell@1 (profit 1) then cooldown@2, buy@3/sell@4 (profit 2), total `3`.

---

## 6. Best Time to Buy and Sell Stock with Transaction Fee

**Problem Statement:**
Given `prices` and an integer `fee`, find the maximum profit with unlimited transactions, where each completed transaction (one buy + one sell) incurs a fixed transaction `fee`, charged once per transaction (commonly modeled as being subtracted at the sell step).

**Example:**
- Input: `prices = [1,3,2,8,4,9], fee = 2`
- Output: `8`
- Explanation: Buy on day 0 (price 1), sell on day 3 (price 8), profit `= 8 - 1 - 2 = 5`. Then buy on day 4 (price 4), sell on day 5 (price 9), profit `= 9 - 4 - 2 = 3`. Total `= 5 + 3 = 8`.

**Approach 1 — Recursion (state: day, holding):**
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
