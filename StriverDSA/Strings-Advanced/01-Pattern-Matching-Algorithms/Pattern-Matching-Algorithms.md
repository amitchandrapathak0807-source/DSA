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

**Logic (Steps):**
1. Loop `i` from `0` to `n - m` (every possible starting position in `text`).
2. For each `i`, reset `j = 0` and compare `text[i+j]` with `pattern[j]`, incrementing `j` while they match.
3. If `j` reaches `m` (the full pattern matched), record `i` as a match.
4. Move to the next `i` and repeat.

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

**Walkthrough:** With `text = "aabaabaabab"`, `pattern = "aab"`: at `i=0`, `j` matches `'a','a','b'` all three, `j` reaches `m=3`, record `0`. At `i=1`, `text[1]='a'` vs `pattern[0]='a'` matches, `text[2]='b'` vs `pattern[1]='a'` mismatches, `j` stays at 1, no match. Continuing this way, matches are found at `i=0`, `3`, and `6`, giving the result `[0, 3, 6]` as stated in the Example.

---

**Optimized Approach:**
Build the Z-array in O(n) using the `[l, r]` window (the "Z-box") technique, then use it for pattern matching:

**Logic (Steps):**
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

**Walkthrough:** For `text="aabaabaabab"`, `pattern="aab"`: build `combined = pattern + "$" + text = "aab$aabaabaabab"`. Computing the Z-array of `combined`, e.g. on `s="aaaaa"` as a simpler illustration of the same mechanics: `i=1` has no window yet (`i>=r`), so compare directly from `s[0]`, extending to `z[1]=4` and setting `l=1, r=5`. `i=2` falls inside the window (`k=1`), candidate `=min(r-i, z[k])=min(3,4)=3`, extension attempt fails immediately, `z[2]=3`. Similarly `z[3]=2`, `z[4]=1`, giving Z-array `[_,4,3,2,1]`. Applying the same construction to `combined`, every position after the `$` where `Z[i]==pattern.Length(3)` marks a match; these occur at `text` offsets `0`, `3`, and `6`, giving the result `[0, 3, 6]`, matching the Example's Output.

---

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

**Logic (Steps):**
1. Loop `i` from `0` to `n - m`, trying every alignment of `pattern` against `text`.
2. For each `i`, compare characters `text[i+j]` and `pattern[j]` for increasing `j` until a mismatch or `j == m`.
3. If `j == m`, the full pattern matched at `i`, so record `i`.
4. Advance to the next `i` and repeat, with no reuse of prior comparisons.

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

**Walkthrough:** With `text = "abxabcabcaby"`, `pattern = "abcaby"`: at `i=0`, `text[0]='a'` vs `pattern[0]='a'` matches, `text[1]='b'` vs `pattern[1]='b'` matches, `text[2]='x'` vs `pattern[2]='c'` mismatches, `j` stops at 2, no match. Trying each subsequent `i` similarly, the full pattern `"abcaby"` finally lines up starting at `i=6` (`text[6..11] = "abcaby"`), so `j` reaches `m=6` and `6` is recorded — matching the expected Output of index `6`.

---

**Optimized Approach:**

**Logic (Steps):**
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

**Walkthrough:** For `pattern = "abcaby"`, `lps` is built by comparing `pattern[i]` against `pattern[len]`: `i=1` `'b'` vs `'a'`(len=0) mismatches → `lps[1]=0`; `i=2` `'c'` vs `'a'` mismatches → `lps[2]=0`; `i=3` `'a'` vs `'a'` matches → `len=1, lps[3]=1`; `i=4` `'b'` vs `pattern[1]='b'` matches → `len=2, lps[4]=2`; `i=5` `'y'` vs `pattern[2]='c'` mismatches, `len` falls back via `lps[1]=0`, retry `'y'` vs `'a'` mismatches → `lps[5]=0`. Final `lps = [0,0,0,1,2,0]`, matching the value stated in the Example. During the search phase over `text = "abxabcabcaby"`, pointers `i` and `j` advance together; mismatches make `j` jump via `lps[j-1]` without moving `i` backward, and the match is ultimately found when `j` reaches `m=6` at `i=12`, reporting `i-j=6` — the expected Output.

---

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

**Logic (Steps):**
1. Loop `len` from `n` down to `1`.
2. For each `len`, call `IsPalindrome(s, 0, len - 1)`, which checks characters from both ends inward.
3. The first (longest) `len` for which the prefix is a palindrome gives the answer `n - len`.
4. Return that value immediately once found.

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

**Walkthrough:** For `s = "abcd"` (n=4): try `len=4`, prefix `"abcd"`, `IsPalindrome` compares `'a'` vs `'d'` → false. Try `len=3`, prefix `"abc"`, compares `'a'` vs `'c'` → false. Try `len=2`, prefix `"ab"`, compares `'a'` vs `'b'` → false. Try `len=1`, prefix `"a"`, trivially a palindrome (single char) → true, return `n - len = 4 - 1 = 3`, matching the expected Output.

---

**Optimized Approach:**
Use the KMP failure function (LPS array) trick: build the string `combined = s + "#" + reverse(s)`, where `#` is a separator character guaranteed not to appear in `s` (prevents the prefix/suffix match from crossing over between `s` and its reverse in an invalid way). Compute the LPS array of `combined`. The last value, `lps[combined.Length - 1]`, is the length of the longest prefix of `s` that is also a suffix of `reverse(s)` — which is exactly the longest palindromic prefix of `s` (because a suffix of `reverse(s)` equal to some prefix of `s` means that prefix reads the same forwards as it does from the reversed string, i.e., it's a palindrome). The answer is `n - lps[last]`.

**Logic (Steps):**
1. Compute `reversed = reverse(s)`.
2. Build `combined = s + "#" + reversed`.
3. Run `BuildLpsArray(combined)` (same LPS construction as the KMP algorithm).
4. Read `longestPalindromicPrefix = lps[combined.Length - 1]`.
5. Return `n - longestPalindromicPrefix` as the number of characters to prepend.

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

**Walkthrough:** For `s = "abcd"` (n=4): `reversed = "dcba"`, `combined = "abcd#dcba"` (length 9). Building its LPS array: `lps[0]=0`; `i=1` `'b'` vs `combined[0]='a'` mismatches → `lps[1]=0`; `i=2` `'c'` vs `'a'` mismatches → `lps[2]=0`; `i=3` `'d'` vs `'a'` mismatches → `lps[3]=0`; `i=4` `'#'` vs `'a'` mismatches → `lps[4]=0`; `i=5` `'d'` vs `'a'` mismatches → `lps[5]=0`; `i=6` `'c'` vs `'a'` mismatches → `lps[6]=0`; `i=7` `'b'` vs `'a'` mismatches → `lps[7]=0`; `i=8` `'a'` vs `combined[0]='a'` matches → `len=1, lps[8]=1`. Final `lps[8]=1`, so the longest palindromic prefix of `s` has length `1` (just `"a"`). Answer = `n - 1 = 4 - 1 = 3`, matching the expected Output.

---

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

**Logic (Steps):**
1. Loop `i` from `0` to `n - m`, trying each alignment.
2. Compare `text[i+j]` with `pattern[j]` for increasing `j` until mismatch or `j == m`.
3. If `j == m`, record `i` as a match.
4. Repeat for the next `i`.

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

**Walkthrough:** With `text = "abcabcabc"`, `pattern = "cab"`: at `i=0` window `"abc"` vs `"cab"` mismatches at `j=0`. At `i=1` window `"bca"` mismatches at `j=0`. At `i=2` window `"cab"` matches all three characters, `j` reaches `m=3`, record `2`. Continuing through `i=3,4`, then `i=5` window `"cab"` again matches, record `5`. Final result `[2, 5]`, matching the expected Output.

---

**Optimized Approach:**
Use a polynomial rolling hash:
- Treat the pattern/window as a number in some base `b` (e.g. 26 or 256) modulo a large prime `p` (to keep hash values bounded and reduce collisions): `hash = s[0]*b^(m-1) + s[1]*b^(m-2) + ... + s[m-1]*b^0 (mod p)`.
- Precompute `pattern`'s hash and the hash of `text`'s first window in O(m).
- To move the window from index `i` to `i+1`, remove the contribution of the outgoing character `text[i]` and add the incoming character `text[i+m]`, all in O(1):
  `newHash = ((oldHash - text[i]*b^(m-1)) * b + text[i+m]) mod p`.
- Whenever `windowHash == patternHash`, do a full character-by-character comparison to confirm the match (protects against hash collisions where different substrings hash to the same value).

**Logic (Steps):**
1. Compute `highestPower = Base^(m-1) mod Mod`, used to remove the leading character's contribution when rolling.
2. Compute `patternHash` and the initial `windowHash` (for `text[0..m-1]`) in O(m).
3. For each window position `i`, if `patternHash == windowHash`, verify with a direct substring comparison and record `i` on a true match.
4. Roll `windowHash` forward: subtract `text[i] * highestPower`, multiply by `Base`, add `text[i+m]`, all modulo `Mod`.
5. Repeat until all windows from `0` to `n-m` are checked.

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

**Walkthrough:** With `text = "abcabcabc"`, `pattern = "cab"`, `m=3`: initial `windowHash` = hash of `"abc"`, `patternHash` = hash of `"cab"` — they differ, no match at `i=0`. Roll forward (remove `text[0]='a'`, add `text[3]='a'`) → window `"bca"`; hash still differs at `i=1`. Roll again (remove `text[1]='b'`, add `text[4]='b'`) → window `"cab"`; at `i=2`, `windowHash == patternHash`, verify with `text.Substring(2,3) == "cab"` → true → record `2`. Continuing to roll, `i=3` gives `"abc"` (no match), `i=4` gives `"bca"` (no match), `i=5` gives `"cab"` again — hash matches, verified true, record `5`. Final result `[2, 5]`, matching the expected Output, with full string verification only performed at the two positions where the hash actually matched.
