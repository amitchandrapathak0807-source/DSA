# Dynamic Programming — DP on Strings (Advanced)

## 1. Shortest Common Supersequence

### 1. Shortest Common Supersequence

**Problem Statement:**
Given two strings `s1` and `s2`, find the shortest string that has both `s1` and `s2` as subsequences. If multiple shortest supersequences exist, any one of them is a valid answer.

**Example:**
- Input: `s1 = "abac"`, `s2 = "cab"`
- Output: `"cabac"` (length 5)
- Explanation: `"cabac"` contains `"abac"` as a subsequence (positions 1,2,3,4 → a,b,a,c) and `"cab"` as a subsequence (positions 0,1,2 → c,a,b). The LCS of `"abac"` and `"cab"` is `"ab"` (length 2), so the shortest supersequence has length `4 + 3 - 2 = 5`.

**Approach 1 — Recursion:**

**Logic (Steps):**
1. Use two pointers `i` into `s1` and `j` into `s2`.
2. Base case: if `i` reaches the end of `s1`, the answer is whatever remains of `s2` (`s2.Substring(j)`); symmetrically if `j` reaches the end of `s2`.
3. If `s1[i] == s2[j]`, the character is shared — take it once and recurse on `(i+1, j+1)`.
4. Otherwise branch: take `s1[i]` and recurse on `(i+1, j)`, or take `s2[j]` and recurse on `(i, j+1)`.
5. Return whichever branch produces the shorter resulting string.

```csharp
public string ShortestCommonSupersequenceRec(string s1, string s2, int i, int j)
{
    if (i == s1.Length) return s2.Substring(j);
    if (j == s2.Length) return s1.Substring(i);

    if (s1[i] == s2[j])
        return s1[i] + ShortestCommonSupersequenceRec(s1, s2, i + 1, j + 1);

    string include1 = s1[i] + ShortestCommonSupersequenceRec(s1, s2, i + 1, j);
    string include2 = s2[j] + ShortestCommonSupersequenceRec(s1, s2, i, j + 1);

    return include1.Length <= include2.Length ? include1 : include2;
}
```
Time Complexity: O(2^(n+m)) — every mismatched position branches into two recursive calls.
Space Complexity: O(n+m) recursion stack (plus string allocations along the way).

**Walkthrough:** With `s1 = "abac"`, `s2 = "cab"`, call `(i=0, j=0)`: `s1[0]='a'`, `s2[0]='c'` → mismatch, branch into `include1 = 'a' + f(1,0)` and `include2 = 'c' + f(0,1)`. Following the `include2` path, `f(0,1)` compares `s1[0]='a'` with `s2[1]='a'` → match, take `'a'` and recurse `f(1,2)`, which keeps matching down to a length-5 result `"cabac"`. Both branches are explored fully and the shorter one is kept at each level; the final call returns `"cabac"` (length 5), matching the expected Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursive structure as Approach 1, but key the cache on `(i, j)` since the result of a sub-problem depends only on the current pointer positions.
2. Before recursing, check the dictionary `memo` for an already-computed answer at `(i, j)`; return it immediately if present.
3. Otherwise compute `result` exactly as before (match, or the shorter of the two mismatch branches).
4. Store `result` in `memo[(i, j)]` before returning it, so repeated calls with the same `(i, j)` are answered in O(1).

```csharp
public string ShortestCommonSupersequenceMemo(
    string s1, string s2, int i, int j, Dictionary<(int, int), string> memo)
{
    if (i == s1.Length) return s2.Substring(j);
    if (j == s2.Length) return s1.Substring(i);
    if (memo.TryGetValue((i, j), out string cached)) return cached;

    string result;
    if (s1[i] == s2[j])
    {
        result = s1[i] + ShortestCommonSupersequenceMemo(s1, s2, i + 1, j + 1, memo);
    }
    else
    {
        string include1 = s1[i] + ShortestCommonSupersequenceMemo(s1, s2, i + 1, j, memo);
        string include2 = s2[j] + ShortestCommonSupersequenceMemo(s1, s2, i, j + 1, memo);
        result = include1.Length <= include2.Length ? include1 : include2;
    }

    memo[(i, j)] = result;
    return result;
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s1 = "abac"`, `s2 = "cab"`, call `(0,0)` again mismatches `'a'` vs `'c'`, so it needs `(1,0)` and `(0,1)`. Because those sub-calls overlap with ones reached from other branches (e.g. `(1,0)` may be revisited via a different path), the `memo` dictionary stores each `(i,j)` result the first time it's computed and returns it directly on subsequent hits, avoiding recomputation. The top-level call `(0,0)` still resolves to `"cabac"` (length 5), matching the Output, but with only O(n·m) distinct sub-problems evaluated instead of an exponential number.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[i, j]` = length of the LCS of `s1[0..i)` and `s2[0..j)`. Base case: row/column 0 stay 0 (LCS with an empty string is 0).
2. Transition: if `s1[i-1] == s2[j-1]`, `dp[i, j] = 1 + dp[i-1, j-1]`; otherwise `dp[i, j] = max(dp[i-1, j], dp[i, j-1])`. Fill row by row, `i` then `j` increasing.
3. To reconstruct the supersequence, walk backward from `(x=n, y=m)`: if `s1[x-1] == s2[y-1]`, take that shared character once and move both pointers diagonally.
4. Otherwise move toward whichever neighbor (`dp[x-1,y]` or `dp[x,y-1]`) has the larger LCS value, appending the corresponding character from `s1` or `s2`.
5. Once one pointer hits 0, append all remaining characters from the other string, then reverse the collected characters to get the final answer.

```csharp
public string ShortestCommonSupersequenceTab(string s1, string s2)
{
    int n = s1.Length, m = s2.Length;
    int[,] dp = new int[n + 1, m + 1];

    // dp[i, j] = length of LCS of s1[0..i) and s2[0..j)
    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (s1[i - 1] == s2[j - 1])
                dp[i, j] = 1 + dp[i - 1, j - 1];
            else
                dp[i, j] = Math.Max(dp[i - 1, j], dp[i, j - 1]);
        }
    }

    // Reconstruct the supersequence by walking the LCS table from bottom-right to top-left
    StringBuilder sb = new StringBuilder();
    int x = n, y = m;
    while (x > 0 && y > 0)
    {
        if (s1[x - 1] == s2[y - 1])
        {
            sb.Append(s1[x - 1]);
            x--; y--;
        }
        else if (dp[x - 1, y] > dp[x, y - 1])
        {
            sb.Append(s1[x - 1]);
            x--;
        }
        else
        {
            sb.Append(s2[y - 1]);
            y--;
        }
    }
    while (x > 0) { sb.Append(s1[x - 1]); x--; }
    while (y > 0) { sb.Append(s2[y - 1]); y--; }

    char[] arr = sb.ToString().ToCharArray();
    Array.Reverse(arr);
    return new string(arr);
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s1 = "abac"` (n=4), `s2 = "cab"` (m=3), the LCS length table fills as:
```
        ""  c   a   b
    ""   0  0   0   0
    a    0  0   1   1
    b    0  0   1   2
    a    0  0   1   2
    c    0  1   1   2
```
`dp[4,3] = 2` (LCS = "ab"), so `length(SCS) = n + m - L = 4 + 3 - 2 = 5`, since the 2 matched characters would otherwise be double-counted from both strings. Reconstructing from `(x=4, y=3)`: `s1[3]='c'` vs `s2[2]='b'` mismatch, `dp[3,3]=2 > dp[4,2]=1` → take `'c'`, `x=3`; `s1[2]='a'` vs `s2[2]='b'` mismatch, `dp[2,3]=2 > dp[3,2]=1` → take `'a'`, `x=2`; `s1[1]='b'` vs `s2[2]='b'` match → take `'b'`, `x=1,y=2`; `s1[0]='a'` vs `s2[1]='a'` match → take `'a'`, `x=0,y=1`; remaining `y=1` → append `s2[0]='c'`. Collected in reverse: `c,a,b,a,c` → reversed gives `"cabac"`, length 5, matching the Output.

---

**Approach 4 — Space Optimization:**
Reconstructing the actual supersequence needs the full 2-D table for backtracking, so space optimization only applies to computing its *length* (using rolling rows for the underlying LCS length).

**Logic (Steps):**
1. State: `prev[j]` and `curr[j]` hold only the previous and current row of the LCS-length table (only lengths are needed here, not the reconstructed string).
2. Transition: same as tabulation — match adds 1 to the diagonal (`prev[j-1]`), mismatch takes `max(prev[j], curr[j-1])`.
3. After finishing row `i`, clone `curr` into `prev` to roll the window forward; iterate `i` from 1 to n.
4. Once the loop ends, `prev[m]` holds the LCS length; the SCS length is simply `n + m - lcsLength`.

```csharp
public int ShortestCommonSupersequenceLength(string s1, string s2)
{
    int n = s1.Length, m = s2.Length;
    int[] prev = new int[m + 1];
    int[] curr = new int[m + 1];

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (s1[i - 1] == s2[j - 1])
                curr[j] = 1 + prev[j - 1];
            else
                curr[j] = Math.Max(prev[j], curr[j - 1]);
        }
        prev = (int[])curr.Clone();
    }

    int lcsLength = prev[m];
    return n + m - lcsLength;
}
```
Time Complexity: O(n*m), Space Complexity: O(m).

**Walkthrough:** Using the same table from Approach 3 with `s1 = "abac"`, `s2 = "cab"`, the rolling rows converge to `prev[m] = prev[3] = 2` (the LCS length, since the underlying recurrence is identical to the LCS length recurrence). The SCS length is then `n + m - lcsLength = 4 + 3 - 2 = 5`, which matches the length of the reconstructed `"cabac"` from Approach 3's Output, confirming both approaches agree.

---

## 2. Distinct Subsequences

### 2. Distinct Subsequences

**Problem Statement:**
Given two strings `s` and `t`, count the number of distinct ways `t` appears as a subsequence of `s` (i.e., count the distinct index-selections in `s` that spell out `t` in order).

**Example:**
- Input: `s = "rabbbit"`, `t = "rabbit"`
- Output: `3`
- Explanation: There are 3 ways to choose indices from `s` that spell `"rabbit"`, corresponding to picking one of the three `'b'`s in `"rabbbit"` to skip: `ra**b**b**b**it`, `ra**b****b**bit`, `ra**b**b**b**it` (choosing 2 of the 3 middle `b`s each time).

**Approach 1 — Recursion:**

**Logic (Steps):**
1. Use pointer `i` into `s` and `j` into `t`.
2. Base case: if `j == t.Length`, all of `t` has been matched — that's 1 valid way. If `i == s.Length` first, `s` ran out before `t` was fully matched — 0 ways.
3. If `s[i] == t[j]`, two independent choices contribute to the count: use `s[i]` to match `t[j]` (recurse `i+1, j+1`), or skip `s[i]` and still look for `t[j]` later (recurse `i+1, j`) — sum both.
4. If the characters differ, `s[i]` cannot help, so simply skip it (recurse `i+1, j`).

```csharp
public int NumDistinctRec(string s, string t, int i, int j)
{
    if (j == t.Length) return 1; // matched all characters of t
    if (i == s.Length) return 0; // ran out of s before matching all of t

    int count;
    if (s[i] == t[j])
        count = NumDistinctRec(s, t, i + 1, j + 1) + NumDistinctRec(s, t, i + 1, j);
    else
        count = NumDistinctRec(s, t, i + 1, j);

    return count;
}
```
Time Complexity: O(2^n) in the worst case.
Space Complexity: O(n) recursion stack.

**Walkthrough:** With `s = "rabbbit"`, `t = "rabbit"`, at the point where `i` reaches the first `'b'` in `s` (index 2) and `j` points at the first `'b'` in `t`, characters match, so the recursion both "uses" this `b` (advance to `i+1,j+1`) and "skips" it (advance to `i+1,j`) hoping a later `b` matches instead — this branching across the three `b`s in `"rabbbit"` is exactly what produces 3 distinct index-selections. Summing all successful branches (where `j` eventually reaches `t.Length`) yields `3`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursion as Approach 1, keyed on `(i, j)` since each sub-problem depends only on the current pointers.
2. Check `memo[i, j]` first (initialized to `-1`); return the cached value immediately if it's not `-1`.
3. Otherwise compute `count` the same way (sum both branches on a match, skip on a mismatch).
4. Store the result in `memo[i, j]` before returning.

```csharp
public int NumDistinctMemo(string s, string t, int i, int j, int[,] memo)
{
    if (j == t.Length) return 1;
    if (i == s.Length) return 0;
    if (memo[i, j] != -1) return memo[i, j];

    int count;
    if (s[i] == t[j])
        count = NumDistinctMemo(s, t, i + 1, j + 1, memo) + NumDistinctMemo(s, t, i + 1, j, memo);
    else
        count = NumDistinctMemo(s, t, i + 1, j, memo);

    return memo[i, j] = count;
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s = "rabbbit"`, `t = "rabbit"`, the sub-problem `(i, j)` at the second `'b'` position gets reached via multiple paths (skip-then-match vs. match-then-skip), so without memoization it would be recomputed repeatedly. Storing `memo[i, j]` the first time each pair is resolved collapses this to O(n·m) unique calls. The top-level call `(0, 0)` still returns `3`, matching the Output.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[i, j]` = number of distinct ways `t[0..j)` appears as a subsequence of `s[0..i)`.
2. Base case: `dp[i, 0] = 1` for all `i` (an empty `t` is trivially a subsequence of any prefix, in exactly one way — choosing nothing).
3. Transition: if `s[i-1] == t[j-1]`, `dp[i, j] = dp[i-1, j-1] + dp[i-1, j]` (use this match, or skip it); otherwise `dp[i, j] = dp[i-1, j]` (must skip).
4. Fill row by row (`i` outer, `j` inner) from 1 to n and 1 to m; the answer is `dp[n, m]`.

```csharp
public int NumDistinctTab(string s, string t)
{
    int n = s.Length, m = t.Length;
    int[,] dp = new int[n + 1, m + 1];

    // An empty t is a subsequence of any prefix of s exactly once
    for (int i = 0; i <= n; i++) dp[i, 0] = 1;

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (s[i - 1] == t[j - 1])
                dp[i, j] = dp[i - 1, j - 1] + dp[i - 1, j];
            else
                dp[i, j] = dp[i - 1, j];
        }
    }

    return dp[n, m];
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s = "rabbbit"` (n=7), `t = "rabbit"` (m=6), building the table row by row, each of the three `'b'` rows in `s` (indices 3, 4, 5 in `s`, corresponding to `i=3,4,5`) adds `dp[i-1,j-1]` (use this b) to `dp[i-1,j]` (skip it) at the column for `t`'s single `'b'` (j=3). By the time `i=7` (all of `s` consumed), the accumulated count at `dp[7,6]` is `3`, matching the Output.

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: only two rows are kept, `prev` (row `i-1`) and `curr` (row `i`), each indexed by `j`.
2. Base case: `prev[0] = curr[0] = 1` (empty `t` matches trivially at every `i`).
3. Transition: same as tabulation — `curr[j] = prev[j-1] + prev[j]` on a match, `curr[j] = prev[j]` otherwise.
4. After each row `i`, clone `curr` into `prev`; after the loop, `prev[m]` holds the answer.

```csharp
public int NumDistinctSpaceOptimized(string s, string t)
{
    int n = s.Length, m = t.Length;
    int[] prev = new int[m + 1];
    int[] curr = new int[m + 1];

    prev[0] = 1;
    curr[0] = 1;

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (s[i - 1] == t[j - 1])
                curr[j] = prev[j - 1] + prev[j];
            else
                curr[j] = prev[j];
        }
        prev = (int[])curr.Clone();
    }

    return prev[m];
}
```
Time Complexity: O(n*m), Space Complexity: O(m).

**Walkthrough:** Using the same `s = "rabbbit"`, `t = "rabbit"` example, the rolling arrays produce identical values to the 2-D table at each row, so `prev[6]` after processing all 7 rows equals `dp[7,6] = 3` from Approach 3, matching the Output while using only O(m) extra space.

---

## 3. Edit Distance

### 3. Edit Distance

**Problem Statement:**
Given two strings `s1` and `s2`, find the minimum number of operations (insert a character, delete a character, or replace a character) required to convert `s1` into `s2`.

**Example:**
- Input: `s1 = "horse"`, `s2 = "ros"`
- Output: `3`
- Explanation: `horse` → `rorse` (replace `'h'` with `'r'`) → `rose` (delete `'r'`) → `ros` (delete `'e'`). Three operations suffice, and no fewer are possible.

**Approach 1 — Recursion:**

**Logic (Steps):**
1. Use pointers `i` into `s1`, `j` into `s2`.
2. Base case: if `s1` is exhausted, insert all remaining `s2` characters (`s2.Length - j` ops); if `s2` is exhausted, delete all remaining `s1` characters (`s1.Length - i` ops).
3. If `s1[i] == s2[j]`, no operation is needed here — recurse straight to `(i+1, j+1)`.
4. Otherwise try all three operations: **insert** (advance `j` only, `i, j+1`), **delete** (advance `i` only, `i+1, j`), **replace** (advance both, `i+1, j+1`) — each costs `1 +` the sub-problem.
5. Return `1 +` the minimum of the three branches.

```csharp
public int EditDistanceRec(string s1, string s2, int i, int j)
{
    if (i == s1.Length) return s2.Length - j; // insert all remaining characters of s2
    if (j == s2.Length) return s1.Length - i; // delete all remaining characters of s1

    if (s1[i] == s2[j])
        return EditDistanceRec(s1, s2, i + 1, j + 1);

    int insertOp = 1 + EditDistanceRec(s1, s2, i, j + 1);
    int deleteOp = 1 + EditDistanceRec(s1, s2, i + 1, j);
    int replaceOp = 1 + EditDistanceRec(s1, s2, i + 1, j + 1);

    return Math.Min(insertOp, Math.Min(deleteOp, replaceOp));
}
```
Time Complexity: O(3^(n+m)) — three branches at every mismatched pair of indices.
Space Complexity: O(n+m) recursion stack.

**Walkthrough:** With `s1 = "horse"`, `s2 = "ros"`, calling `EditDistanceRec(s1, s2, 0, 0)`: `s1[0]='h'` vs `s2[0]='r'` mismatch, so try all three ops. Following `replaceOp = 1 + f(1,1)`: `f(1,1)` compares `"orse"` vs `"os"` — `s1[1]='o'` matches `s2[1]='o'`, advance to `f(2,2)` (`"rse"` vs `"s"`); there `s1[2]='r'` vs `s2[2]='s'` mismatch, and `deleteOp = 1 + f(3,2)` wins (`"se"` vs `"s"`); `s1[3]='s'` matches `s2[2]='s'`, advance to `f(4,3)` (`"e"` vs `""`) which hits the base case `j == s2.Length`, returning `s1.Length - i = 5 - 4 = 1` (delete trailing `'e'`). Unwinding: `f(3,2)=1`, `f(2,2)=1+1=2`, `f(1,1)=2`, and the top-level `replaceOp = 1+2 = 3`. Other branches cost 4 or more, so the minimum is `3`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursive structure as Approach 1, cached on `(i, j)`.
2. Check `memo[i, j]` (initialized to `-1`) before doing any work; return the cached value if present.
3. Otherwise compute `result` the same way (free move on match, min of insert/delete/replace on mismatch).
4. Store `result` in `memo[i, j]` before returning.

```csharp
public int EditDistanceMemo(string s1, string s2, int i, int j, int[,] memo)
{
    if (i == s1.Length) return s2.Length - j;
    if (j == s2.Length) return s1.Length - i;
    if (memo[i, j] != -1) return memo[i, j];

    int result;
    if (s1[i] == s2[j])
    {
        result = EditDistanceMemo(s1, s2, i + 1, j + 1, memo);
    }
    else
    {
        int insertOp = 1 + EditDistanceMemo(s1, s2, i, j + 1, memo);
        int deleteOp = 1 + EditDistanceMemo(s1, s2, i + 1, j, memo);
        int replaceOp = 1 + EditDistanceMemo(s1, s2, i + 1, j + 1, memo);
        result = Math.Min(insertOp, Math.Min(deleteOp, replaceOp));
    }

    return memo[i, j] = result;
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s1 = "horse"`, `s2 = "ros"`, sub-problem `(3,2)` (`"se"` vs `"s"`) can be reached through several different combinations of earlier insert/delete/replace choices; `memo[3,2]` caches its value (`1`) the first time it's computed so later paths reuse it directly. The top call `(0,0)` still resolves to `3`, matching the Output, now with only O(n·m) distinct sub-problems.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[i, j]` = min operations to convert `s1[0..i)` into `s2[0..j)`.
2. Base case: `dp[i, 0] = i` (delete everything), `dp[0, j] = j` (insert everything).
3. Transition: if `s1[i-1] == s2[j-1]`, `dp[i, j] = dp[i-1, j-1]` (no cost); else `dp[i, j] = 1 + min(dp[i, j-1], dp[i-1, j], dp[i-1, j-1])` (insert, delete, replace).
4. Fill row by row from `i=1..n`, `j=1..m`; the answer is `dp[n, m]`.

```csharp
public int EditDistanceTab(string s1, string s2)
{
    int n = s1.Length, m = s2.Length;
    int[,] dp = new int[n + 1, m + 1];

    for (int i = 0; i <= n; i++) dp[i, 0] = i; // delete all of s1[0..i)
    for (int j = 0; j <= m; j++) dp[0, j] = j; // insert all of s2[0..j)

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (s1[i - 1] == s2[j - 1])
            {
                dp[i, j] = dp[i - 1, j - 1];
            }
            else
            {
                int insertOp = 1 + dp[i, j - 1];
                int deleteOp = 1 + dp[i - 1, j];
                int replaceOp = 1 + dp[i - 1, j - 1];
                dp[i, j] = Math.Min(insertOp, Math.Min(deleteOp, replaceOp));
            }
        }
    }

    return dp[n, m];
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s1 = "horse"` (n=5), `s2 = "ros"` (m=3): row 0 is `[0,1,2,3]` (insert costs) and column 0 is `[0,1,2,3,4,5]` (delete costs). Filling forward, at `i=1` (`'h'`) vs `j=1` (`'r'`) mismatch gives `dp[1,1] = 1 + min(dp[1,0], dp[0,1], dp[0,0]) = 1 + min(1,1,0) = 1`. Continuing this fill through the full table yields `dp[5,3] = 3`, matching the Output (replace `'h'`→`'r'`, delete `'r'`, delete `'e'`).

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: only two rows kept, `prev` (row `i-1`) and `curr` (row `i`).
2. Base case: `prev[j] = j` for all `j` (row 0); `curr[0] = i` is set at the start of each outer iteration (column 0).
3. Transition: same as tabulation — match copies `prev[j-1]`; mismatch takes `1 + min(curr[j-1], prev[j], prev[j-1])`.
4. After each row, clone `curr` into `prev`; after the loop, `prev[m]` is the answer.

```csharp
public int EditDistanceSpaceOptimized(string s1, string s2)
{
    int n = s1.Length, m = s2.Length;
    int[] prev = new int[m + 1];
    int[] curr = new int[m + 1];

    for (int j = 0; j <= m; j++) prev[j] = j;

    for (int i = 1; i <= n; i++)
    {
        curr[0] = i;
        for (int j = 1; j <= m; j++)
        {
            if (s1[i - 1] == s2[j - 1])
            {
                curr[j] = prev[j - 1];
            }
            else
            {
                int insertOp = 1 + curr[j - 1];
                int deleteOp = 1 + prev[j];
                int replaceOp = 1 + prev[j - 1];
                curr[j] = Math.Min(insertOp, Math.Min(deleteOp, replaceOp));
            }
        }
        prev = (int[])curr.Clone();
    }

    return prev[m];
}
```
Time Complexity: O(n*m), Space Complexity: O(m).

**Walkthrough:** Using the same `s1 = "horse"`, `s2 = "ros"` example, the rolling rows compute identical values to Approach 3's table at every position, so after processing all 5 rows, `prev[3] = 3` — matching the Output while only keeping O(m) extra space instead of the full 2-D table.

---

## 4. Wildcard Matching

### 4. Wildcard Matching

**Problem Statement:**
Given an input string `s` and a pattern `p` containing the wildcard characters `'?'` (matches any single character) and `'*'` (matches any sequence of characters, including the empty sequence), determine whether `p` matches the entirety of `s`.

**Example:**
- Input: `s = "adceb"`, `p = "*a*b"`
- Output: `true`
- Explanation: The leading `'*'` matches the empty prefix, `'a'` matches `'a'`, the middle `'*'` matches `"dce"`, and the trailing `'b'` matches `'b'`. So `p` matches `s` entirely.

**Approach 1 — Recursion:**

**Logic (Steps):**
1. Use pointer `i` into `s`, `j` into `p`.
2. Base case: if both are exhausted, it's a match; if only `p` is exhausted, it's a mismatch (leftover `s` characters).
3. If `s` is exhausted but `p` isn't, the match succeeds only if every remaining pattern character is `'*'` (since `'*'` can absorb an empty remainder).
4. If `p[j]` equals `s[i]` or is `'?'`, consume both and recurse `(i+1, j+1)`.
5. If `p[j] == '*'`, try both: match zero characters (`i, j+1`) or match one more character and stay on the same `'*'` (`i+1, j`) — return true if either succeeds.
6. Otherwise (literal mismatch), return false.

```csharp
public bool IsMatchRec(string s, string p, int i, int j)
{
    if (i == s.Length && j == p.Length) return true;
    if (j == p.Length) return false; // pattern exhausted but string has leftover characters

    if (i == s.Length)
    {
        // string exhausted; remaining pattern must consist only of '*'
        for (int k = j; k < p.Length; k++)
            if (p[k] != '*') return false;
        return true;
    }

    if (p[j] == s[i] || p[j] == '?')
        return IsMatchRec(s, p, i + 1, j + 1);

    if (p[j] == '*')
        return IsMatchRec(s, p, i + 1, j) || IsMatchRec(s, p, i, j + 1);

    return false;
}
```
Time Complexity: O(2^(n+m)) worst case — every `'*'` branches into two recursive calls.
Space Complexity: O(n+m) recursion stack.

**Walkthrough:** With `s = "adceb"`, `p = "*a*b"`: at `(0,0)` `p[0]='*'` tries "match zero" (`0,1`) first — `p[1]='a'` matches `s[0]='a'`, advance to `(1,2)`. There `p[2]='*'` tries matching zero, one, two, three characters of `s` until it lands on `(4,3)` where `p[3]='b'` matches `s[4]='b'`, advancing to `(5,4)` — both indices exhausted, so this branch returns `true`. Since at least one branch of each `'*'` succeeds, the overall call returns `true`, matching the Output.

---

**Approach 2 — Memoization:**

**Logic (Steps):**
1. Same recursive structure as Approach 1, cached on `(i, j)` (the two base cases run first since they're cheap and don't need caching).
2. Check `memo[i, j]` (initialized to `-1`); if set, return `memo[i, j] == 1`.
3. Otherwise compute `result` exactly as in Approach 1 (match/`?`, `*` branch, or literal mismatch).
4. Store `result` as `1`/`0` in `memo[i, j]` before returning.

```csharp
public bool IsMatchMemo(string s, string p, int i, int j, int[,] memo)
{
    if (i == s.Length && j == p.Length) return true;
    if (j == p.Length) return false;

    if (i == s.Length)
    {
        for (int k = j; k < p.Length; k++)
            if (p[k] != '*') return false;
        return true;
    }

    if (memo[i, j] != -1) return memo[i, j] == 1;

    bool result;
    if (p[j] == s[i] || p[j] == '?')
        result = IsMatchMemo(s, p, i + 1, j + 1, memo);
    else if (p[j] == '*')
        result = IsMatchMemo(s, p, i + 1, j, memo) || IsMatchMemo(s, p, i, j + 1, memo);
    else
        result = false;

    memo[i, j] = result ? 1 : 0;
    return result;
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s = "adceb"`, `p = "*a*b"`, the second `'*'` at `j=2` is probed from `(1,2)` repeatedly as it tries consuming 0, 1, 2, 3 characters of `s` — each `(i, 2)` sub-problem is cached after its first evaluation, so subsequent probes reuse it instead of recomputing. The call still returns `true`, matching the Output, now bounded to O(n·m) sub-problems.

---

**Approach 3 — Tabulation:**

**Logic (Steps):**
1. State: `dp[i, j]` = true if `s[0..i)` matches pattern `p[0..j)`.
2. Base case: `dp[0,0] = true` (empty matches empty); `dp[0, j] = dp[0, j-1] && p[j-1]=='*'` (empty `s` matches a pattern prefix only if it's all `'*'`).
3. Transition: if `p[j-1]` equals `s[i-1]` or is `'?'`, `dp[i,j] = dp[i-1,j-1]`; if `p[j-1]=='*'`, `dp[i,j] = dp[i-1,j] || dp[i,j-1]` (use `*` on one more char, or skip `*`); otherwise `dp[i,j] = false`.
4. Fill row by row from `i=1..n`, `j=1..m`; the answer is `dp[n, m]`.

```csharp
public bool IsMatchTab(string s, string p)
{
    int n = s.Length, m = p.Length;
    bool[,] dp = new bool[n + 1, m + 1];
    dp[0, 0] = true;

    // Empty string matches a pattern prefix only if that prefix is all '*'
    for (int j = 1; j <= m; j++)
        dp[0, j] = dp[0, j - 1] && p[j - 1] == '*';

    for (int i = 1; i <= n; i++)
    {
        for (int j = 1; j <= m; j++)
        {
            if (p[j - 1] == s[i - 1] || p[j - 1] == '?')
                dp[i, j] = dp[i - 1, j - 1];
            else if (p[j - 1] == '*')
                dp[i, j] = dp[i - 1, j] || dp[i, j - 1];
            else
                dp[i, j] = false;
        }
    }

    return dp[n, m];
}
```
Time Complexity: O(n*m), Space Complexity: O(n*m).

**Walkthrough:** With `s = "adceb"` (n=5), `p = "*a*b"` (m=4): row 0 is `[true, true, false, true, false]` since only the leading `'*'` (j=1) keeps the empty match alive, and `j=2` ('a') breaks it, but `j=3` is `'*'` again after a false — actually `dp[0,3] = dp[0,2] && '*' = false` (once broken, stays false), and `dp[0,4] = dp[0,3] && 'b'=='*' = false`. Filling the rest of the table row by row using the transition, `dp[5,4]` evaluates to `true`, matching the Output — confirming `"*a*b"` matches `"adceb"`.

---

**Approach 4 — Space Optimization:**

**Logic (Steps):**
1. State: only two rows kept, `prev` (row `i-1`) and `curr` (row `i`).
2. Base case: `prev[0] = true`; `prev[j] = prev[j-1] && p[j-1]=='*'` for the rest of row 0.
3. Transition: `curr[0] = false` at the start of each row (a non-empty pattern-consumed-so-far can't match empty `s` unless `j=0`, which is trivially false for `i>0`); then same match/`*`/mismatch rules as tabulation using `prev` and `curr`.
4. After each row, clone `curr` into `prev`; after the loop, `prev[m]` is the answer.

```csharp
public bool IsMatchSpaceOptimized(string s, string p)
{
    int n = s.Length, m = p.Length;
    bool[] prev = new bool[m + 1];
    bool[] curr = new bool[m + 1];

    prev[0] = true;
    for (int j = 1; j <= m; j++)
        prev[j] = prev[j - 1] && p[j - 1] == '*';

    for (int i = 1; i <= n; i++)
    {
        curr[0] = false;
        for (int j = 1; j <= m; j++)
        {
            if (p[j - 1] == s[i - 1] || p[j - 1] == '?')
                curr[j] = prev[j - 1];
            else if (p[j - 1] == '*')
                curr[j] = prev[j] || curr[j - 1];
            else
                curr[j] = false;
        }
        prev = (bool[])curr.Clone();
    }

    return prev[m];
}
```
Time Complexity: O(n*m), Space Complexity: O(m).

**Walkthrough:** Using the same `s = "adceb"`, `p = "*a*b"` example, the rolling rows compute identical values to Approach 3's full table row by row, ending with `prev[4] = true` after processing all 5 rows — matching the Output while using only O(m) extra space instead of the full 2-D table.

---
