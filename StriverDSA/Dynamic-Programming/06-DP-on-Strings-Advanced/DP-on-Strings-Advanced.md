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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
Reconstructing the actual supersequence needs the full 2-D table for backtracking, so space optimization only applies to computing its *length* (using rolling rows for the underlying LCS length).
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

**Explanation:**
The Shortest Common Supersequence problem reduces to the Longest Common Subsequence (LCS) problem. Intuitively, the shortest supersequence should reuse every character the two strings have in common exactly once, and additionally include every character that is *not* part of that common subsequence from both strings. If `L` is the length of the LCS of `s1` and `s2`, then:

```
length(SCS) = n + m - L
```

because the `L` matched characters would otherwise be counted twice (once from each string).

**Dry run** with `s1 = "abac"` (n=4), `s2 = "cab"` (m=3):

First build the LCS length table `dp[i,j]` for `s1[0..i)` vs `s2[0..j)`:

```
        ""  c   a   b
    ""   0  0   0   0
    a    0  0   1   1
    b    0  0   1   2
    a    0  0   1   2
    c    0  1   1   2
```

`dp[4,3] = 2`, confirming LCS length 2 (LCS = "ab"). So SCS length = 4 + 3 - 2 = 5.

Now reconstruct by walking from `(x=4, y=3)` back to `(0,0)`:
- `s1[3]='c'`, `s2[2]='b'` → mismatch. Compare `dp[3,3]=2` vs `dp[4,2]=1`. Since `dp[3,3] > dp[4,2]`, move up: take `s1[3]='c'`, `x=3`.
- `s1[2]='a'`, `s2[2]='b'` → mismatch. `dp[2,3]=2` vs `dp[3,2]=1`. Take `s1[2]='a'`, `x=2`.
- `s1[1]='b'`, `s2[2]='b'` → match! Take `'b'`, `x=1, y=2`.
- `s1[0]='a'`, `s2[1]='a'` → match! Take `'a'`, `x=0, y=1`.
- `x=0`, remaining `y=1`: append `s2[0]='c'`.

Characters collected in reverse order: `c, a, b, a, c` → reversing gives `"cabac"`, exactly length 5, and it contains both `"abac"` and `"cab"` as subsequences.

## 2. Distinct Subsequences

### 2. Distinct Subsequences

**Problem Statement:**
Given two strings `s` and `t`, count the number of distinct ways `t` appears as a subsequence of `s` (i.e., count the distinct index-selections in `s` that spell out `t` in order).

**Example:**
- Input: `s = "rabbbit"`, `t = "rabbit"`
- Output: `3`
- Explanation: There are 3 ways to choose indices from `s` that spell `"rabbit"`, corresponding to picking one of the three `'b'`s in `"rabbbit"` to skip: `ra**b**b**b**it`, `ra**b****b**bit`, `ra**b**b**b**it` (choosing 2 of the 3 middle `b`s each time).

**Approach 1 — Recursion:**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
At each pair of indices `(i, j)`, we decide how character `s[i]` participates while trying to match `t[j]`. If `s[i] == t[j]`, we have two independent choices: use `s[i]` to match `t[j]` and advance both pointers (`i+1, j+1`), or skip `s[i]` entirely and keep looking for `t[j]` later in `s` (`i+1, j`) — both contribute to the count because they represent distinct index-selections in `s`. If the characters differ, `s[i]` cannot help match `t[j]`, so we must skip it (`i+1, j`). The base cases are: if all of `t` has been matched (`j == t.Length`), that's 1 valid way; if `s` runs out before `t` is fully matched, that's 0 ways. The tabulation table initializes `dp[i,0] = 1` because an empty `t` is trivially a subsequence of any prefix of `s` in exactly one way (choosing nothing).

## 3. Edit Distance

### 3. Edit Distance

**Problem Statement:**
Given two strings `s1` and `s2`, find the minimum number of operations (insert a character, delete a character, or replace a character) required to convert `s1` into `s2`.

**Example:**
- Input: `s1 = "horse"`, `s2 = "ros"`
- Output: `3`
- Explanation: `horse` → `rorse` (replace `'h'` with `'r'`) → `rose` (delete `'r'`) → `ros` (delete `'e'`). Three operations suffice, and no fewer are possible.

**Approach 1 — Recursion:**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
At indices `(i, j)`, if `s1[i] == s2[j]` no operation is needed for these characters, so we move to `(i+1, j+1)` for free. Otherwise we must pay for one operation and try all three possibilities: **insert** a character into `s1` matching `s2[j]` (advance only `j`, since the inserted character now matches, but `s1` hasn't been consumed: `i, j+1`), **delete** `s1[i]` (advance only `i`: `i+1, j`), or **replace** `s1[i]` with `s2[j]` (advance both: `i+1, j+1`). We take the minimum of the three plus 1.

**Dry run** with `s1 = "horse"`, `s2 = "ros"` (using 0-indexed recursion `EditDistanceRec(s1, s2, 0, 0)`):
- `(i=0,'h'), (j=0,'r')` → mismatch. Try insert/delete/replace, each costs `1 +` the sub-problem.
  - `replaceOp`: `1 + f(1,1)` where `f(1,1)` compares `"orse"` vs `"os"`.
    - `(i=1,'o'), (j=1,'o')` → match, move to `f(2,2)`: `"rse"` vs `"s"`.
      - `(i=2,'r'), (j=2,'s')` → mismatch. Best turns out to be `deleteOp = 1 + f(3,2)`: `"se"` vs `"s"`.
        - `(i=3,'s'), (j=2,'s')` → match, move to `f(4,3)`: `"e"` vs `""`.
          - `j == s2.Length` → return `s1.Length - i = 5 - 4 = 1` (delete the trailing `'e'`).
        - So `f(3,2) = 1`, and `f(2,2) = 1 + 1 = 2`.
      - So `f(1,1) = 2`.
  - So `replaceOp` path gives `1 + 2 = 3`.
  - The other branches (pure insert-heavy or delete-heavy paths) cost 4 or more, so the minimum overall is **3**, matching the sequence: replace `'h'`→`'r'`, delete the extra `'r'`, delete the trailing `'e'`.

## 4. Wildcard Matching

### 4. Wildcard Matching

**Problem Statement:**
Given an input string `s` and a pattern `p` containing the wildcard characters `'?'` (matches any single character) and `'*'` (matches any sequence of characters, including the empty sequence), determine whether `p` matches the entirety of `s`.

**Example:**
- Input: `s = "adceb"`, `p = "*a*b"`
- Output: `true`
- Explanation: The leading `'*'` matches the empty prefix, `'a'` matches `'a'`, the middle `'*'` matches `"dce"`, and the trailing `'b'` matches `'b'`. So `p` matches `s` entirely.

**Approach 1 — Recursion:**
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

**Approach 2 — Memoization:**
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

**Approach 3 — Tabulation:**
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

**Approach 4 — Space Optimization:**
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

**Explanation:**
The base cases handle exhaustion of either string: if `s` is fully consumed (`i == s.Length`) while pattern characters remain, the match can still succeed only if every remaining pattern character is `'*'`, since `'*'` alone can absorb an empty remainder — this is why a leading run of `'*'` characters in `p` can validly match an empty `s`. If `p` is exhausted first while `s` still has characters left, the match fails immediately since there is no way to consume the leftover string.

At a `'*'` character, the recursion tries two choices and returns true if either succeeds: (1) **match zero characters** — treat the `'*'` as consuming nothing and move only the pattern pointer forward (`i, j+1`), letting the rest of the pattern try to match the remaining string; (2) **match one more character** — let the `'*'` absorb `s[i]` and stay on the same `'*'` for the next string character (`i+1, j`), since a single `'*'` can greedily consume an arbitrarily long run of characters one at a time across recursive calls.

**Dry run 1**: `s = "aa"`, `p = "*"`.
- `i=0, j=0`: not the double-exhausted base case (`j != p.Length` is false here since `m=1`, so continue). `p[0] == '*'`.
  - Option A (match zero): `IsMatchRec(s, p, 0, 1)` → now `j == p.Length` but `i == 0 != s.Length`, so this branch returns `false`.
  - Option B (match one more): `IsMatchRec(s, p, 1, 0)` → `i=1, j=0`, `p[0]` is still `'*'`.
    - Option A: `IsMatchRec(s, p, 1, 1)` → `j == p.Length`, `i=1 != s.Length(2)` → `false`.
    - Option B: `IsMatchRec(s, p, 2, 0)` → `i == s.Length`, remaining pattern from `j=0` is `"*"`, which is all `'*'` → returns `true`.
  - So Option B eventually returns `true`, and the overall result is `true` — the single `'*'` absorbs both `'a'` characters.

**Dry run 2**: `s = "cb"`, `p = "?a"`.
- `i=0, j=0`: `p[0] == '?'`, which matches any character including `s[0]='c'`. Move to `IsMatchRec(s, p, 1, 1)`.
- `i=1, j=1`: `p[1] == 'a'`, `s[1] == 'b'`. They are not equal and `p[1]` is not `'?'` or `'*'` → returns `false`.
- Overall result: `false`. Although `'?'` successfully matched `'c'`, the literal `'a'` in the pattern cannot match `'b'` in the string, so `p` does not match `s`.
