# Greedy Algorithms — Basics

## Concept: Greedy Algorithms

A **greedy algorithm** builds a solution step by step, and at every step it makes the choice that looks **locally optimal** — the best option available *right now* — without reconsidering it later, hoping that this sequence of locally optimal choices adds up to a **globally optimal** solution.

This strategy is fast and simple (usually a single pass plus a sort), but it does not work for every problem. It only produces a correct answer when the problem exhibits two properties:

1. **Greedy choice property** — A globally optimal solution can be reached by making a locally optimal (greedy) choice at each step, without ever needing to revisit or change earlier choices. In other words, there always exists an optimal solution that agrees with the greedy pick made at the first step.
2. **Optimal substructure** — An optimal solution to the problem contains within it optimal solutions to subproblems. Once the greedy choice is made, what remains is a smaller version of the same problem.

To trust a greedy strategy, you typically justify it with an **exchange argument**: assume some optimal solution does *not* make the greedy choice, then show you can swap/exchange elements in that solution to match the greedy choice without making the solution worse. If that swap never hurts (and can only help or stay equal), the greedy choice is safe.

### Greedy vs. Dynamic Programming

Dynamic Programming (DP) is needed exactly when the greedy choice property **fails** — i.e., when the locally best choice at one step can block you from reaching the true global optimum, so you must explore multiple choices and combine results from overlapping subproblems (often with memoization/tabulation).

The classic contrast:

- **Fractional Knapsack** — you can take *fractions* of items, so always picking the item with the highest value/weight ratio first is provably optimal. Greedy works.
- **0/1 Knapsack** — you can only take an item whole or not at all. Greedily taking the highest ratio item first can leave you with wasted capacity that a different combination would have used better. There is no exchange argument that holds in general, so greedy fails here — you need DP to try combinations and pick the best.

So: reach for greedy when you can prove the exchange argument holds; reach for DP when greedy choices can be "wrong" in a way that only shows up later.

---

## 1. Assign Cookies

**Problem Statement:** You are given two arrays: `greed[]`, where `greed[i]` is the minimum cookie size that makes child `i` content, and `cookies[]`, where `cookies[j]` is the size of cookie `j`. Each child can receive at most one cookie, and a cookie can satisfy a child only if its size is greater than or equal to the child's greed factor. Maximize the number of content children.

**Example:**
- Input: `greed = [1, 2, 3]`, `cookies = [1, 1]`
- Output: `1`
- Explanation: You have 2 cookies of size 1. Only the child with greed factor 1 can be satisfied (child with greed 2 or 3 needs a bigger cookie). So at most 1 child is content.

**Approach:** Sort both `greed` and `cookies` in ascending order. Use two pointers: try to satisfy the least greedy child first with the smallest cookie that can satisfy them. If the current smallest available cookie can satisfy the current least greedy child, assign it and move both pointers forward; otherwise the cookie is too small for anyone remaining (since children are sorted ascending), so discard it and move only the cookie pointer.

This is safe by an exchange argument: giving the smallest sufficient cookie to the least greedy child never hurts, because that small cookie could only have satisfied children with even smaller (or equal) greed anyway — using it on the least greedy child "saves" all larger cookies for greedier children, which can only help or stay equal compared to any other assignment.

```csharp
public class Solution {
    public int FindContentChildren(int[] greed, int[] cookies) {
        Array.Sort(greed);
        Array.Sort(cookies);

        int i = 0; // pointer for greed
        int j = 0; // pointer for cookies
        int content = 0;

        while (i < greed.Length && j < cookies.Length) {
            if (cookies[j] >= greed[i]) {
                content++;
                i++;
                j++;
            } else {
                j++;
            }
        }

        return content;
    }
}
```

Time Complexity: O(n log n + m log m) for sorting both arrays (n = children, m = cookies), plus O(n + m) for the two-pointer scan.
Space Complexity: O(1) extra (ignoring sort's internal space).

---

## 2. Fractional Knapsack Problem

**Problem Statement:** Given `n` items, each with a `value` and a `weight`, and a knapsack of capacity `W`, determine the maximum total value that can be obtained by putting items into the knapsack. Unlike 0/1 knapsack, you are allowed to take a **fraction** of an item.

**Example:**
- Input: `capacity = 50`, items = `[(value=60, weight=10), (value=100, weight=20), (value=120, weight=30)]`
- Output: `240.0`
- Explanation: Take the full item with ratio 6 (value 60, weight 10) and the full item with ratio 5 (value 100, weight 20) — that uses 30 capacity for 160 value. Remaining capacity is 20, and the last item (ratio 4, weight 30) contributes 20/30 of itself = 80 value. Total = 160 + 80 = 240.

**Approach:** Compute each item's value/weight ratio, then sort items in descending order of this ratio. Greedily fill the knapsack: take as much as possible of the current highest-ratio item (all of it if it fits, otherwise only the fraction that fits the remaining capacity), then move to the next.

Why this is safe (exchange argument): suppose an optimal solution does not fully take the item with the highest ratio while capacity remains and a lower-ratio item is (partially) taken instead. Swapping an infinitesimal amount of weight from the lower-ratio item to the higher-ratio item strictly increases (or at worst keeps equal) the total value, since value gained per unit weight is higher. Repeating this swap argument shows any optimal solution can be transformed into the greedy (highest-ratio-first) solution without decreasing value — so greedy is optimal.

```csharp
public class Item {
    public int Value;
    public int Weight;
    public Item(int value, int weight) {
        Value = value;
        Weight = weight;
    }
}

public class Solution {
    public double FractionalKnapsack(int capacity, Item[] items) {
        // Sort items by value/weight ratio in descending order
        Array.Sort(items, (a, b) =>
            ((double)b.Value / b.Weight).CompareTo((double)a.Value / a.Weight));

        double totalValue = 0.0;
        int remainingCapacity = capacity;

        foreach (var item in items) {
            if (remainingCapacity <= 0) break;

            if (item.Weight <= remainingCapacity) {
                // Take the whole item
                totalValue += item.Value;
                remainingCapacity -= item.Weight;
            } else {
                // Take a fraction of the item
                double fraction = (double)remainingCapacity / item.Weight;
                totalValue += item.Value * fraction;
                remainingCapacity = 0;
            }
        }

        return totalValue;
    }
}
```

Time Complexity: O(n log n) for sorting by ratio, plus O(n) for the greedy fill.
Space Complexity: O(1) extra (or O(log n)/O(n) depending on the sort implementation's internal footprint).

---

## 3. Minimum Number of Coins to Make a Given Value

**Problem Statement:** Given a target value `V` and an unlimited supply of coins of certain denominations, find the minimum number of coins needed to make up that value using a greedy approach.

**Important caveat:** The greedy approach (always pick the largest denomination ≤ remaining value) only works for **canonical** coin systems, such as standard currency denominations like `[1, 2, 5, 10, 20, 50, 100, ...]` (e.g., Indian or US-style currency). It does **not** work for arbitrary denominations in general. For example, with denominations `[1, 3, 4]` and target `6`, greedy picks `4 + 1 + 1 = 3` coins, but the optimal answer is `3 + 3 = 2` coins. In the general case (arbitrary denominations), you must use **Dynamic Programming** (the classic "Coin Change — Minimum Coins" DP) to guarantee correctness.

**Example:**
- Input: `V = 49`, canonical denominations `[1, 2, 5, 10, 20, 50, 100, 500, 1000]`
- Output: `[20, 20, 5, 2, 2]` (5 coins)
- Explanation: Greedily take the largest coin ≤ remaining value each time: 49 → take 20 (rem 29) → take 20 (rem 9) → take 5 (rem 4) → take 2 (rem 2) → take 2 (rem 0). Total 5 coins.

**Approach:** Sort denominations in descending order (or assume they are given sorted, as with standard currency). Repeatedly pick the largest denomination that is less than or equal to the remaining value, subtract it, and count it; repeat until the remaining value is 0.

Why this is safe for canonical systems: in a canonical coin system, each denomination is constructed so that taking one larger coin is never "wasteful" compared to any combination of smaller coins that could substitute for it — the denominations are spaced such that the greedy pick never blocks a better solution (this can be verified per-system, e.g., standard currency denominations satisfy this). This property does **not** hold for arbitrary denominations (like `[1, 3, 4]`), which is why greedy can fail there and DP (trying all denominations at each step and taking the minimum) is required in general.

```csharp
public class Solution {
    public List<int> MinCoins(int value, int[] denominations) {
        // Assumes denominations form a canonical system (e.g., standard currency)
        var sorted = denominations.OrderByDescending(d => d).ToArray();
        var result = new List<int>();

        int remaining = value;
        foreach (int coin in sorted) {
            while (remaining >= coin) {
                remaining -= coin;
                result.Add(coin);
            }
        }

        return result; // list of coins used; result.Count is the minimum number of coins
    }
}
```

Time Complexity: O(n log n) for sorting denominations (O(n) if already sorted) plus O(V/smallestCoin) in the worst case for the greedy subtraction loop.
Space Complexity: O(k) where k is the number of coins used in the result.

---

## 4. Lemonade Change

**Problem Statement:** At a lemonade stand, each lemonade costs $5. Customers stand in a queue and pay with a $5, $10, or $20 bill, one customer at a time. You must give each customer the correct change ($5 back for a $10 payment; $15 back for a $20 payment), and you start with no change on hand. Determine if you can provide correct change to every customer given the order `bills[]` in which they pay.

**Example:**
- Input: `bills = [5, 5, 5, 10, 20]`
- Output: `true`
- Explanation: First three customers pay with $5 (no change needed, you now hold three $5 bills). Fourth customer pays $10 — give back one $5 (you now hold two $5 and one $10). Fifth customer pays $20 — give back one $10 and one $5 (or three $5s); either works since you have both, so it succeeds.

**Approach:** Greedily track the count of $5 and $10 bills you are holding. For each customer:
- If they pay $5, just keep it (no change needed).
- If they pay $10, you must give back one $5; if you don't have one, fail immediately.
- If they pay $20, prefer giving change as one $10 + one $5 (if available) over three $5s, saving your $5 bills for future flexibility since $5 bills are more broadly useful (a $10 can only be used to make change for a $20, while a $5 can be used for both $10 and $20 change). If neither combination is available, fail.

This greedy choice (preferring to break a $10 before spending three $5s) is safe because $5 bills are strictly more valuable as "change currency" than $10 bills — a $5 bill can satisfy change for both a $10 and a $20 payment, whereas a $10 bill can only help satisfy change for a $20. Hoarding $5s over $10s never reduces your future flexibility, so it can only help or match any other valid strategy.

```csharp
public class Solution {
    public bool LemonadeChange(int[] bills) {
        int fives = 0;
        int tens = 0;

        foreach (int bill in bills) {
            if (bill == 5) {
                fives++;
            } else if (bill == 10) {
                if (fives == 0) return false;
                fives--;
                tens++;
            } else { // bill == 20
                if (tens > 0 && fives > 0) {
                    tens--;
                    fives--;
                } else if (fives >= 3) {
                    fives -= 3;
                } else {
                    return false;
                }
            }
        }

        return true;
    }
}
```

Time Complexity: O(n), a single pass over the bills array.
Space Complexity: O(1), only two counters are used.

---

## 5. Valid Parenthesis String

**Problem Statement:** Given a string `s` containing only the characters `'('`, `')'`, and `'*'`, where `'*'` can be treated as `'('`, as `')'`, or as an empty string, determine whether `s` can be interpreted as a valid parenthesis sequence (every `'('` has a matching later `')'`, and at no prefix do closes outnumber opens).

**Example:**
- Input: `s = "(*))"`
- Output: `true`
- Explanation: Treat `*` as `'('` — the string becomes `"(())"`, which is a valid parenthesis sequence. (Alternatively, treat `*` as empty — `"())"` is invalid, but at least one valid interpretation exists, so the answer is `true`.)

**Approach:** Instead of tracking a single count of open parentheses (which can't handle the ambiguity of `*`), track a **range** of possible counts of unmatched open parentheses: `low` (the minimum possible open count, treating every `*` as `')'` or empty when beneficial to keep `low` small) and `high` (the maximum possible open count, treating every `*` as `'('` when beneficial). For each character:
- `'('`: both `low` and `high` increase by 1.
- `')'`: both `low` and `high` decrease by 1.
- `'*'`: `low` decreases by 1 (optimistic: treat as `')'`) and `high` increases by 1 (optimistic: treat as `'('`).

If `high` ever drops below 0, too many `)`/`*`-as-`)` are forced and no interpretation can recover — return `false` immediately. Clamp `low` to 0 whenever it goes negative, because a negative "minimum open count" isn't meaningful — you simply choose not to treat that many `*` as `)`. At the end, the string is valid if `low == 0` is achievable, i.e., `low` reaching 0 (through clamping) by the end.

Why this range-tracking is safe: since `*` is genuinely ambiguous, a single running count can't represent all reachable states. By tracking the full range `[low, high]` of achievable open-counts at each position, we implicitly explore all interpretations simultaneously without exponential blowup — as long as 0 remains within `[low, high]` at the end (which clamping `low` at 0 ensures we detect), some valid assignment of `*` exists.

```csharp
public class Solution {
    public bool CheckValidString(string s) {
        int low = 0;  // minimum possible number of unmatched '('
        int high = 0; // maximum possible number of unmatched '('

        foreach (char c in s) {
            if (c == '(') {
                low++;
                high++;
            } else if (c == ')') {
                low--;
                high--;
            } else { // c == '*'
                low--;
                high++;
            }

            if (high < 0) return false; // too many ')' forced, no recovery possible
            if (low < 0) low = 0;       // can't have negative unmatched opens; clamp
        }

        return low == 0;
    }
}
```

Time Complexity: O(n), a single pass over the string.
Space Complexity: O(1), only two counters are used.

---

## Explanation: Dry Runs

### Dry Run — Fractional Knapsack

Consider `capacity = 50` with items:

| Item | Value | Weight | Ratio (Value/Weight) |
|------|-------|--------|-----------------------|
| A    | 60    | 10     | 6.0                   |
| B    | 100   | 20     | 5.0                   |
| C    | 120   | 30     | 4.0                   |

**Step 1 — Sort by ratio descending:** Order becomes `A (6.0), B (5.0), C (4.0)`.

**Step 2 — Fill greedily:**
- Item A: weight 10 ≤ remaining capacity 50 → take all of it. `totalValue = 60`, `remainingCapacity = 50 - 10 = 40`.
- Item B: weight 20 ≤ remaining capacity 40 → take all of it. `totalValue = 60 + 100 = 160`, `remainingCapacity = 40 - 20 = 20`.
- Item C: weight 30 > remaining capacity 20 → take a fraction: `fraction = 20 / 30 = 0.667`. Value added = `120 * 0.667 = 80`. `totalValue = 160 + 80 = 240`, `remainingCapacity = 0`.

**Result:** `totalValue = 240.0`, matching the expected output. The greedy pass never needed to "undo" a choice — each item was fully consumed in ratio order until capacity ran out, confirming the exchange argument in practice: no rearrangement of these picks could produce more value given the capacity constraint.

### Dry Run — Valid Parenthesis String on `"(*))"`

Track `low` (min possible open count) and `high` (max possible open count), starting at `low = 0, high = 0`.

| Step | Char | Action | low | high | Check |
|------|------|--------|-----|------|-------|
| 1 | `(` | low++, high++ | 1 | 1 | high ≥ 0, ok |
| 2 | `*` | low--, high++ | 0 | 2 | high ≥ 0; low was 0, no clamp needed |
| 3 | `)` | low--, high-- | -1 → clamp to 0 | 1 | high ≥ 0, ok; low clamped to 0 |
| 4 | `)` | low--, high-- | -1 → clamp to 0 | 0 | high ≥ 0, ok; low clamped to 0 |

End of string: `low = 0`, so the function returns `true`.

**Why the range was necessary:** at step 2, the `*` is genuinely ambiguous — it could be `(` (pushing possible open count up to 2), `)` (pushing it down to 0), or empty (leaving it at 1). A single running count would have to arbitrarily commit to one interpretation right away and could easily pick wrong (e.g., committing to `*` = `)` would give a count of 0 after step 2, and then step 3's `)` would immediately push the count to -1, incorrectly reporting failure even though the string is actually valid via `*` = `(`). By keeping the whole reachable range `[low, high]`, the algorithm defers the decision: it only declares failure when even the most optimistic interpretation (`high`) goes negative, and it only declares success if 0 is reachable (i.e., `low` can be brought down to exactly 0) by the end — both of which correctly capture the true satisfiability of the wildcard string without needing to enumerate every combination of `*` assignments explicitly.
