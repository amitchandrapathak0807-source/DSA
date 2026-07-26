# Strings (Advanced) — Pattern Matching Algorithms

## 1. Z-Function (Z-Array) and Pattern Searching Using It

### 1. Z-Function (Z-Array) and Pattern Searching Using It

**Problem Statement:**
Given a string `s` of length `n`, the Z-array (Z-function) is an array `Z` of length `n` where `Z[i]` denotes the length of the longest common prefix between `s` and the suffix of `s` starting at index `i` (`Z[0]` is conventionally undefined/set to `0` or `n`). The Z-array can be used to search for all occurrences of a `pattern` inside a `text` in linear time by building the string `pattern + "$" + text` (the `$` is a separator character that does not appear in either string) and scanning its Z-array: wherever `Z[i]` equals the length of the pattern, an occurrence starts at that position in `text`.

**Example:**
- Input: `text = "aabxaabxcaabxaabxay"`, `pattern = "aabxaabxy"` — searched inside a longer text such as `"aabxaabxcaabxaabxay"`. For a self-contained Z-array demonstration: `s = "aabxaabxcaabxaabxay"`.
- Output: For pattern searching with `text = "aabaabaabab"`, `pattern = "aab"` → matches found at indices `[0, 3, 6]`.
- Explanation: We build `combined = pattern + "$" + text = "aab$aabaabaabab"`. Computing the Z-array of `combined` and checking every index `i > pattern.Length` where `Z[i] == pattern.Length` gives the starting positions of matches in `text`. Subtracting `pattern.Length + 1` (length of `pattern + "$"`) from those indices yields `[0, 3, 6]`.

**Brute Force Approach:**
Naive sliding-window comparison: for every starting index in `text`, compare characters of `pattern` one by one.

```csharp
using System;
using System.Collections.Generic;

public class PatternSearchBruteForce
{
    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        int n = text.Length;
        int m = pattern.Length;

        for (int i = 0; i <= n - m; i++)
        {
            int j = 0;
            while (j < m && text[i + j] == pattern[j])
            {
                j++;
            }
            if (j == m)
            {
                result.Add(i);
            }
        }
        return result;
    }
}
```

Time Complexity: O(n*m) — in the worst case (e.g. text = "aaaaa...a", pattern = "aaab") every starting position requires up to `m` comparisons.
Space Complexity: O(1) extra (excluding the output list).

**Optimized Approach:**
Build the Z-array in O(n) using the `[l, r]` window (the "Z-box") technique, then use it for pattern matching:

1. Maintain two pointers `l` and `r` representing the rightmost segment `[l, r]` found so far that matches a prefix of `s` (i.e., `s[l..r]` is also a prefix of `s`).
2. For each index `i` from `1` to `n-1`:
   - If `i > r`: there is no overlap with any known Z-box, so start comparing `s[0]` with `s[i]` from scratch, incrementing until a mismatch. Set `Z[i]` accordingly, and if `Z[i] > 0`, update `l = i`, `r = i + Z[i] - 1`.
   - If `i <= r`: `i` lies inside the current Z-box, so `s[i]` corresponds to `s[i-l]` inside the prefix. Let `k = i - l`.
     - If `Z[k] < r - i + 1`, the value is fully known and safe to copy: `Z[i] = Z[k]` (no further comparison needed — this is what saves time).
     - Otherwise (`Z[k] >= r - i + 1`), we only know a lower bound, so start comparing from position `r + 1` onward, extending as far as possible, then update `l = i`, `r` to the new match end.
3. For pattern searching, run this construction on `pattern + '$' + text` and report every index `i` (beyond the separator) where `Z[i] == pattern.Length`.

The key insight is that once we know a match exists inside a previously discovered window `[l, r]`, we can reuse previously computed Z-values (via the mirror index `k = i - l`) instead of recomputing character comparisons from scratch, which is what makes the overall algorithm run in linear time — each character is compared at most a constant number of times.

```csharp
using System;
using System.Collections.Generic;

public class PatternSearchZFunction
{
    public static int[] BuildZArray(string s)
    {
        int n = s.Length;
        int[] z = new int[n];
        int l = 0, r = 0;

        for (int i = 1; i < n; i++)
        {
            if (i < r)
            {
                int k = i - l;
                // Take the minimum of the known value and remaining window size
                z[i] = Math.Min(r - i, z[k]);
            }

            // Try to extend the match beyond what we already know
            while (i + z[i] < n && s[z[i]] == s[i + z[i]])
            {
                z[i]++;
            }

            // Update the [l, r] window if we extended past r
            if (i + z[i] > r)
            {
                l = i;
                r = i + z[i];
            }
        }
        return z;
    }

    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        string combined = pattern + "$" + text;
        int[] z = BuildZArray(combined);
        int patternLength = pattern.Length;

        for (int i = patternLength + 1; i < combined.Length; i++)
        {
            if (z[i] == patternLength)
            {
                result.Add(i - patternLength - 1);
            }
        }
        return result;
    }
}
```

Time Complexity: O(n + m), where `n = text.Length` and `m = pattern.Length` — the Z-array of the combined string of length `n + m + 1` is built in linear time.
Space Complexity: O(n + m) for the combined string and the Z-array.

**Explanation:**
Dry run of Z-array construction on `s = "aaaaa"` (n = 5, indices 0..4):

- `l = 0, r = 0`, `z[0]` unused (0 by default).
- `i = 1`: `i >= r` (1 >= 0), so no window reuse. Compare `s[0]` vs `s[1]`: `'a'=='a'` → extend; `s[1]` vs `s[2]`... continues until index runs out. `z[1] = 4` (matches `s[1..4] = "aaaa"` against prefix `"aaaa"`). Since `i + z[i] = 5 > r = 0`, update `l = 1, r = 5`.
- `i = 2`: `i < r` (2 < 5), `k = i - l = 1`. `z[k] = z[1] = 4`. Candidate `= min(r - i, z[k]) = min(5-2, 4) = min(3,4) = 3`. Try extending beyond: `z[2] + i = 2+3=5` which is `== n`, loop condition `i+z[i] < n` fails immediately, so `z[2] = 3`. Since `i + z[2] = 5 > r`? `5 > 5` false, so window stays `l=1, r=5`.
- `i = 3`: `i < r` (3<5), `k = 2`, `z[k] = 3`. Candidate `= min(r-i, z[k]) = min(5-3,3) = min(2,3) = 2`. Extend attempt: `i+z[i]=3+2=5`, not `<5`, stop. `z[3] = 2`. `i+z[3]=5`, not `>r(5)`, window unchanged.
- `i = 4`: `i < r`(4<5), `k=3`, `z[k]=2`. Candidate `= min(r-i, z[k]) = min(5-4,2)=min(1,2)=1`. Extend attempt: `i+z[i]=4+1=5`, not `<5`, stop. `z[4]=1`.

Final Z-array: `[_, 4, 3, 2, 1]` — as expected, each suffix of an all-`'a'` string matches the prefix up to the remaining length.

For pattern searching with `text="aabaabaabab"`, `pattern="aab"`: `combined = "aab$aabaabaabab"`. Scanning the Z-array of `combined` at positions after the `$`, we find `Z[i] == 3` at the three positions corresponding to `text` offsets `0`, `3`, and `6`, giving matches `[0, 3, 6]`.

## 2. KMP Algorithm — Longest Prefix Suffix (LPS) Array and Pattern Searching

### 2. KMP Algorithm — Longest Prefix Suffix (LPS) Array and Pattern Searching

**Problem Statement:**
Given a `text` and a `pattern`, find all starting indices in `text` where `pattern` occurs, using the Knuth-Morris-Pratt (KMP) algorithm. KMP precomputes an array `lps` (Longest Proper Prefix which is also a Suffix) for the `pattern`, where `lps[i]` is the length of the longest proper prefix of `pattern[0..i]` that is also a suffix of `pattern[0..i]`. This array lets the matching phase skip re-examining characters of `text` that are already known to match, avoiding backtracking in `text`.

**Example:**
- Input: `text = "abxabcabcaby"`, `pattern = "abcaby"`
- Output: Match found at index `6`.
- Explanation: `pattern` occurs in `text` starting at index 6 (`"abcaby"` starting at position 6 of `text`). The `lps` array for `pattern = "abcaby"` is `[0, 0, 0, 1, 2, 0]`, which lets the matcher, upon a mismatch, jump forward using previously matched information instead of restarting the comparison from the text's next character.

**Brute Force Approach:**
Sliding-window naive comparison, identical in spirit to the brute-force approach used for the Z-function problem: try every alignment and compare character by character.

```csharp
using System;
using System.Collections.Generic;

public class KmpBruteForce
{
    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        int n = text.Length;
        int m = pattern.Length;

        for (int i = 0; i <= n - m; i++)
        {
            int j = 0;
            while (j < m && text[i + j] == pattern[j])
            {
                j++;
            }
            if (j == m)
            {
                result.Add(i);
            }
        }
        return result;
    }
}
```

Time Complexity: O(n*m) worst case.
Space Complexity: O(1) extra.

**Optimized Approach:**
1. **Build the LPS array** using a two-pointer technique:
   - `len` tracks the length of the current longest matching prefix-suffix; `i` scans the pattern starting at index 1.
   - If `pattern[i] == pattern[len]`, extend the match: `len++`, `lps[i] = len`, `i++`.
   - If they mismatch and `len != 0`, fall back to `len = lps[len - 1]` (do not advance `i`) — reuse previously computed information instead of restarting from zero.
   - If they mismatch and `len == 0`, set `lps[i] = 0` and advance `i`.
2. **Search phase**: walk pointers `i` (text) and `j` (pattern) together.
   - If characters match, advance both.
   - If `j == pattern.Length`, a match is found at `i - j`; then set `j = lps[j - 1]` to continue searching for overlapping matches.
   - On mismatch with `j != 0`, set `j = lps[j - 1]` (do not decrement `i` — text pointer never backtracks, which is the core efficiency gain of KMP).
   - On mismatch with `j == 0`, simply advance `i`.

```csharp
using System;
using System.Collections.Generic;

public class KmpOptimized
{
    public static int[] BuildLpsArray(string pattern)
    {
        int m = pattern.Length;
        int[] lps = new int[m];
        int len = 0; // length of the previous longest prefix-suffix
        int i = 1;

        while (i < m)
        {
            if (pattern[i] == pattern[len])
            {
                len++;
                lps[i] = len;
                i++;
            }
            else if (len != 0)
            {
                len = lps[len - 1]; // fall back, do not advance i
            }
            else
            {
                lps[i] = 0;
                i++;
            }
        }
        return lps;
    }

    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        int n = text.Length;
        int m = pattern.Length;
        if (m == 0) return result;

        int[] lps = BuildLpsArray(pattern);
        int i = 0; // pointer for text
        int j = 0; // pointer for pattern

        while (i < n)
        {
            if (text[i] == pattern[j])
            {
                i++;
                j++;
                if (j == m)
                {
                    result.Add(i - j);
                    j = lps[j - 1]; // look for next overlapping match
                }
            }
            else if (j != 0)
            {
                j = lps[j - 1]; // text pointer i never moves backward
            }
            else
            {
                i++;
            }
        }
        return result;
    }
}
```

Time Complexity: O(n + m) — LPS construction is O(m); the search phase is O(n) because the text pointer `i` never decreases and `j` is bounded by `i`.
Space Complexity: O(m) for the LPS array.

**Explanation:**
Dry run of LPS construction on `pattern = "aabaaab"` (indices 0..6, m = 7):

- `lps[0] = 0` always. `len = 0, i = 1`.
- `i=1`: `pattern[1]='a'`, `pattern[len=0]='a'` → match. `len=1`, `lps[1]=1`, `i=2`.
- `i=2`: `pattern[2]='b'`, `pattern[len=1]='a'` → mismatch, `len!=0` → `len = lps[len-1] = lps[0] = 0`.
- `i=2` (retry): `pattern[2]='b'`, `pattern[len=0]='a'` → mismatch, `len==0` → `lps[2]=0`, `i=3`.
- `i=3`: `pattern[3]='a'`, `pattern[len=0]='a'` → match. `len=1`, `lps[3]=1`, `i=4`.
- `i=4`: `pattern[4]='a'`, `pattern[len=1]='a'` → match. `len=2`, `lps[4]=2`, `i=5`.
- `i=5`: `pattern[5]='a'`, `pattern[len=2]='b'` → mismatch, `len!=0` → `len = lps[1] = 1`.
- `i=5` (retry): `pattern[5]='a'`, `pattern[len=1]='a'` → match. `len=2`, `lps[5]=2`, `i=6`.
- `i=6`: `pattern[6]='b'`, `pattern[len=2]='b'` → match. `len=3`, `lps[6]=3`, `i=7`, loop ends.

Final `lps = [0, 1, 0, 1, 2, 2, 3]`.

During the search phase, whenever a mismatch occurs after partially matching `j` characters, `j` jumps to `lps[j-1]` instead of resetting to `0` and restarting comparison from `text[i-j+1]`, which is exactly what avoids re-scanning already-matched text characters.

## 3. Minimum Characters Needed at the Front to Make a String a Palindrome (using the KMP failure function)

### 3. Minimum Characters Needed at the Front to Make a String a Palindrome

**Problem Statement:**
Given a string `s`, find the minimum number of characters that need to be added in front of `s` to make the resulting string a palindrome. Equivalently, find the length of the longest palindromic prefix of `s`; the answer is `s.Length` minus that length (the remaining suffix, reversed, is prepended).

**Example:**
- Input: `s = "abcd"`
- Output: `3`
- Explanation: The longest palindromic prefix of `"abcd"` is just `"a"` (length 1). We need to add the reverse of the remaining suffix `"bcd"` (i.e. `"dcb"`) to the front, giving `"dcbabcd"`, which is a palindrome. That requires adding `4 - 1 = 3` characters.

**Brute Force Approach:**
For each possible prefix length from longest to shortest, check whether that prefix is itself a palindrome; the first (longest) one found gives the answer.

```csharp
using System;

public class MinCharsPalindromeBruteForce
{
    private static bool IsPalindrome(string s, int start, int end)
    {
        while (start < end)
        {
            if (s[start] != s[end]) return false;
            start++;
            end--;
        }
        return true;
    }

    public static int MinCharsToAdd(string s)
    {
        int n = s.Length;
        for (int len = n; len >= 1; len--)
        {
            if (IsPalindrome(s, 0, len - 1))
            {
                return n - len;
            }
        }
        return n - 0; // empty string edge case (loop covers len=1..n; if s is empty n=0)
    }
}
```

Time Complexity: O(n^2) — checking each of the `n` candidate prefixes for a palindrome costs up to O(n).
Space Complexity: O(1) extra.

**Optimized Approach:**
Use the KMP failure function (LPS array) trick: build the string `combined = s + "#" + reverse(s)`, where `#` is a separator character guaranteed not to appear in `s` (prevents the prefix/suffix match from crossing over between `s` and its reverse in an invalid way). Compute the LPS array of `combined`. The last value, `lps[combined.Length - 1]`, is the length of the longest prefix of `s` that is also a suffix of `reverse(s)` — which is exactly the longest palindromic prefix of `s` (because a suffix of `reverse(s)` equal to some prefix of `s` means that prefix reads the same forwards as it does from the reversed string, i.e., it's a palindrome). The answer is `n - lps[last]`.

```csharp
using System;

public class MinCharsPalindromeOptimized
{
    private static int[] BuildLpsArray(string pattern)
    {
        int m = pattern.Length;
        int[] lps = new int[m];
        int len = 0;
        int i = 1;

        while (i < m)
        {
            if (pattern[i] == pattern[len])
            {
                len++;
                lps[i] = len;
                i++;
            }
            else if (len != 0)
            {
                len = lps[len - 1];
            }
            else
            {
                lps[i] = 0;
                i++;
            }
        }
        return lps;
    }

    public static int MinCharsToAdd(string s)
    {
        int n = s.Length;
        if (n <= 1) return 0;

        char[] reversedChars = s.ToCharArray();
        Array.Reverse(reversedChars);
        string reversed = new string(reversedChars);

        string combined = s + "#" + reversed;
        int[] lps = BuildLpsArray(combined);

        int longestPalindromicPrefix = lps[combined.Length - 1];
        return n - longestPalindromicPrefix;
    }
}
```

Time Complexity: O(n) — building the combined string of length `2n + 1` and its LPS array is linear.
Space Complexity: O(n) for the combined string and the LPS array.

**Explanation:**
Dry run on `s = "aacecaaa"` (n = 8):

- `reverse(s) = "aaacecaa"`.
- `combined = "aacecaaa" + "#" + "aaacecaa" = "aacecaaa#aaacecaa"` (length 17).

Building the LPS array of `combined` step by step (indices 0..16, characters: `a a c e c a a a # a a a c e c a a`):

- `lps[0]=0`. `len=0`.
- `i=1`: `combined[1]='a'` vs `combined[0]='a'` → match, `len=1`, `lps[1]=1`.
- `i=2`: `combined[2]='c'` vs `combined[1]='a'` → mismatch, `len=lps[0]=0`; retry `'c'` vs `combined[0]='a'` → mismatch, `lps[2]=0`.
- `i=3`: `'e'` vs `combined[0]='a'` → mismatch, `lps[3]=0`.
- `i=4`: `'c'` vs `combined[0]='a'` → mismatch, `lps[4]=0`.
- `i=5`: `'a'` vs `combined[0]='a'` → match, `len=1`, `lps[5]=1`.
- `i=6`: `'a'` vs `combined[len=1]='a'` → match, `len=2`, `lps[6]=2`.
- `i=7`: `'a'` vs `combined[len=2]='c'` → mismatch, `len=lps[1]=1`; retry `'a'` vs `combined[1]='a'` → match, `len=2`, `lps[7]=2`.
- `i=8`: `'#'` vs `combined[len=2]='c'` → mismatch, `len=lps[1]=1`; retry `'#'` vs `combined[1]='a'` → mismatch, `len=lps[0]=0`; retry `'#'` vs `combined[0]='a'` → mismatch, `lps[8]=0`.
- `i=9`: `'a'` vs `combined[0]='a'` → match, `len=1`, `lps[9]=1`.
- `i=10`: `'a'` vs `combined[len=1]='a'` → match, `len=2`, `lps[10]=2`.
- `i=11`: `'a'` vs `combined[len=2]='c'` → mismatch, `len=lps[1]=1`; retry `'a'` vs `combined[1]='a'` → match, `len=2`, `lps[11]=2`.
- `i=12`: `'c'` vs `combined[len=2]='c'` → match, `len=3`, `lps[12]=3`.
- `i=13`: `'e'` vs `combined[len=3]='e'` → match, `len=4`, `lps[13]=4`.
- `i=14`: `'c'` vs `combined[len=4]='c'` → match, `len=5`, `lps[14]=5`.
- `i=15`: `'a'` vs `combined[len=5]='a'` → match, `len=6`, `lps[15]=6`.
- `i=16`: `'a'` vs `combined[len=6]='a'` → match, `len=7`, `lps[16]=7`.

Final value: `lps[16] = 7`. This means the longest palindromic prefix of `s` has length `7` (`"aacecaa"`, which indeed reads the same forwards and backwards). The answer is `n - lps[last] = 8 - 7 = 1`: only one character (`'a'`, the reverse of the leftover suffix `"a"`) needs to be added in front, giving `"a" + "aacecaaa" = "aaacecaaa"`, a palindrome.

## 4. Rabin-Karp Algorithm (String Matching Using Rolling Hash)

### 4. Rabin-Karp Algorithm (String Matching Using Rolling Hash)

**Problem Statement:**
Given a `text` and a `pattern`, find all starting indices where `pattern` occurs in `text` using the Rabin-Karp algorithm, which computes a hash of the pattern and hashes of every length-`m` window of `text`, comparing hashes first and falling back to a full character comparison only when hashes collide (to guard against false positives from hash collisions).

**Example:**
- Input: `text = "abcabcabc"`, `pattern = "cab"`
- Output: Matches at indices `[2, 5]`.
- Explanation: `"cab"` appears in `text` starting at index 2 (`"cab"` within `"abCABcabc"`) and again at index 5. Rabin-Karp computes a hash for `pattern = "cab"` once, then slides a window of length 3 across `text`, updating the window's hash in O(1) at each step using the rolling hash formula, comparing against the pattern's hash, and confirming true matches with a direct string comparison.

**Brute Force Approach:**
Same naive sliding-window character comparison as before, without any hashing.

```csharp
using System;
using System.Collections.Generic;

public class RabinKarpBruteForce
{
    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        int n = text.Length;
        int m = pattern.Length;

        for (int i = 0; i <= n - m; i++)
        {
            int j = 0;
            while (j < m && text[i + j] == pattern[j])
            {
                j++;
            }
            if (j == m)
            {
                result.Add(i);
            }
        }
        return result;
    }
}
```

Time Complexity: O(n*m) worst case.
Space Complexity: O(1) extra.

**Optimized Approach:**
Use a polynomial rolling hash:
- Treat the pattern/window as a number in some base `b` (e.g. 26 or 256) modulo a large prime `p` (to keep hash values bounded and reduce collisions): `hash = s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-1]*b^0 (mod p)`.
- Precompute `pattern`'s hash and the hash of `text`'s first window in O(m).
- To move the window from index `i` to `i+1`, remove the contribution of the outgoing character `text[i]` and add the incoming character `text[i+m]`, all in O(1):
  `newHash = ((oldHash - text[i]*b^(m-1)) * b + text[i+m]) mod p`.
- Whenever `windowHash == patternHash`, do a full character-by-character comparison to confirm the match (protects against hash collisions where different substrings hash to the same value).

```csharp
using System;
using System.Collections.Generic;

public class RabinKarpOptimized
{
    private const long Base = 256;
    private const long Mod = 1_000_000_007L;

    public static List<int> SearchPattern(string text, string pattern)
    {
        List<int> result = new List<int>();
        int n = text.Length;
        int m = pattern.Length;
        if (m == 0 || m > n) return result;

        long patternHash = 0;
        long windowHash = 0;
        long highestPower = 1; // Base^(m-1) mod Mod

        for (int i = 0; i < m - 1; i++)
        {
            highestPower = (highestPower * Base) % Mod;
        }

        // Compute initial hashes for pattern and the first window of text
        for (int i = 0; i < m; i++)
        {
            patternHash = (patternHash * Base + pattern[i]) % Mod;
            windowHash = (windowHash * Base + text[i]) % Mod;
        }

        for (int i = 0; i <= n - m; i++)
        {
            if (patternHash == windowHash)
            {
                // Hashes match: verify to rule out a collision
                if (text.Substring(i, m) == pattern)
                {
                    result.Add(i);
                }
            }

            // Roll the hash forward to the next window
            if (i < n - m)
            {
                windowHash = (windowHash - text[i] * highestPower % Mod + Mod) % Mod;
                windowHash = (windowHash * Base + text[i + m]) % Mod;
            }
        }
        return result;
    }
}
```

Time Complexity: O(n + m) on average, since rolling the hash and comparing hash values is O(1) per window; worst case degrades to O(n*m) if many spurious hash collisions force repeated full-string verifications.
Space Complexity: O(1) extra (excluding the output list) — only a constant number of hash variables are kept.

**Explanation:**
Dry run with `text = "abcabcabc"`, `pattern = "cab"` (using `Base = 256`, `Mod = 1,000,000,007` conceptually, but here we illustrate with the underlying idea rather than exact large numbers):

- `m = 3`. `highestPower = Base^2 mod Mod`.
- `patternHash` = hash of `"cab"` computed from characters `'c','a','b'`.
- Initial `windowHash` = hash of `text[0..2] = "abc"`.
- `i=0`: window is `"abc"`, hash differs from `patternHash` (`"cab"` ≠ `"abc"`) → no match. Roll: remove `text[0]='a'` contribution, add `text[3]='a'` → window becomes `"bca"`.
- `i=1`: window is `"bca"`, hash differs from `patternHash` → no match. Roll: remove `text[1]='b'`, add `text[4]='b'` → window becomes `"cab"`.
- `i=2`: window is `"cab"`, `windowHash == patternHash` → verify with `text.Substring(2,3) == "cab"` → true → record match at index `2`. Roll: remove `text[2]='c'`, add `text[5]='c'` → window becomes `"abc"`.
- `i=3`: window `"abc"` → no match. Roll forward similarly.
- `i=4`: window `"bca"` → no match. Roll forward.
- `i=5`: window `"cab"` → hash matches, verify → true → record match at index `5`.
- `i=6`: window `"abc"` → no match (loop ends since `i <= n-m = 6`).

Final result: matches at indices `[2, 5]`, confirming the rolling hash correctly and efficiently locates both occurrences of `"cab"` while only needing full string verification at the two positions where the hash actually matched.
