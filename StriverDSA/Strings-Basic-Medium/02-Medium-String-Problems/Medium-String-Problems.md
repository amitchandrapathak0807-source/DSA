# Strings — Medium String Problems

## 1. Sort Characters by Frequency

### 1. Sort Characters by Frequency
**Problem Statement:** Given a string `s`, sort it in decreasing order based on the frequency of characters. If multiple characters have the same frequency, their relative order does not matter (any valid arrangement is accepted).

**Example:**
- Input: `s = "tree"`
- Output: `"eert"`
- Explanation: `'e'` appears twice and `'r'` and `'t'` appear once. So `'e'` must appear before both `'r'` and `'t'`. `"eetr"` is also a valid answer.

**Brute Force Approach:** Count frequency of every character using a dictionary, then repeatedly scan the dictionary to find the character with the maximum remaining frequency, append it that many times, and mark it as used. Repeat until all characters are consumed.

```csharp
public class Solution
{
    public string FrequencySortBrute(string s)
    {
        Dictionary<char, int> freq = new Dictionary<char, int>();
        foreach (char c in s)
        {
            if (!freq.ContainsKey(c)) freq[c] = 0;
            freq[c]++;
        }

        StringBuilder result = new StringBuilder();
        HashSet<char> used = new HashSet<char>();

        // Repeat until every character has been placed into the result
        for (int iter = 0; iter < freq.Count; iter++)
        {
            char maxChar = '\0';
            int maxCount = -1;

            // Linear scan to find the current maximum frequency character
            foreach (var kvp in freq)
            {
                if (!used.Contains(kvp.Key) && kvp.Value > maxCount)
                {
                    maxCount = kvp.Value;
                    maxChar = kvp.Key;
                }
            }

            used.Add(maxChar);
            result.Append(new string(maxChar, maxCount));
        }

        return result.ToString();
    }
}
```
Time Complexity: O(n + k^2) where n is the length of the string and k is the number of distinct characters (each of the k iterations scans up to k entries).
Space Complexity: O(k) for the frequency map and used set, plus O(n) for the output.

**Optimized Approach:** Use bucket sort by frequency. Count frequencies with a dictionary (O(n)), then create an array of buckets indexed by frequency (0 to n). Place each character into the bucket corresponding to its frequency. Finally, iterate buckets from highest frequency to lowest and append each character that many times. This avoids repeatedly scanning the whole frequency map.

```csharp
public class Solution
{
    public string FrequencySortOptimal(string s)
    {
        Dictionary<char, int> freq = new Dictionary<char, int>();
        foreach (char c in s)
        {
            if (!freq.ContainsKey(c)) freq[c] = 0;
            freq[c]++;
        }

        int n = s.Length;
        // buckets[i] holds all characters that occur exactly i times
        List<char>[] buckets = new List<char>[n + 1];
        for (int i = 0; i <= n; i++) buckets[i] = new List<char>();

        foreach (var kvp in freq)
        {
            buckets[kvp.Value].Add(kvp.Key);
        }

        StringBuilder result = new StringBuilder();
        for (int count = n; count >= 1; count--)
        {
            foreach (char c in buckets[count])
            {
                result.Append(new string(c, count));
            }
        }

        return result.ToString();
    }
}
```
Time Complexity: O(n) — counting frequencies is O(n), bucket placement is O(k), and the final pass visits at most n buckets and outputs n characters total.
Space Complexity: O(n) for the buckets array and output string.

**Explanation:** Bucket sort works here because character frequency is bounded by `n` (the string length), so instead of comparison-based sorting (O(k log k)), we can directly index buckets by frequency value, achieving linear time overall. For `s = "tree"`, frequencies are `{'t':1, 'r':1, 'e':2}`. Bucket 2 contains `['e']`, bucket 1 contains `['t', 'r']`. Scanning from count = 4 down to 1, we hit bucket 2 first, appending `"ee"`, then bucket 1, appending `"t"` and `"r"`, giving `"eetr"` (equivalent to `"eert"`).

---

## 2. Maximum Nesting Depth of Parentheses

### 2. Maximum Nesting Depth of Parentheses
**Problem Statement:** Given a valid parentheses string `s` (which may also contain other characters that should be ignored, or a pure parentheses string depending on constraints), return the maximum nesting depth of the parentheses. The depth of `'('` is 1 more than the depth of the enclosing scope; the depth outside all parentheses is 0.

**Example:**
- Input: `s = "(1+(2*3)+((8)/4))+1"`
- Output: `3`
- Explanation: The digit `8` is enclosed by 3 pairs of parentheses: `(1+(2*3)+((8)/4))+1`, so the maximum depth is 3.

**Brute Force Approach:** For every `'('` encountered, use an explicit stack to push a marker, and compute the current depth as the stack size. Track the maximum depth seen. This is the "brute force" in the sense of literally simulating a stack even though a simple counter would suffice.

```csharp
public class Solution
{
    public int MaxDepthBrute(string s)
    {
        Stack<char> stack = new Stack<char>();
        int maxDepth = 0;

        foreach (char c in s)
        {
            if (c == '(')
            {
                stack.Push(c);
                maxDepth = Math.Max(maxDepth, stack.Count);
            }
            else if (c == ')')
            {
                stack.Pop();
            }
        }

        return maxDepth;
    }
}
```
Time Complexity: O(n) — one pass through the string, each stack operation is O(1).
Space Complexity: O(n) in the worst case (all characters are `'('`), for the stack.

**Optimized Approach:** Since we only ever need the current count of open, unmatched parentheses (not their actual values), we can replace the stack with a simple integer counter: increment on `'('`, decrement on `')'`, and track the maximum value the counter reaches. This removes the extra space used by the stack entirely.

```csharp
public class Solution
{
    public int MaxDepthOptimal(string s)
    {
        int currentDepth = 0;
        int maxDepth = 0;

        foreach (char c in s)
        {
            if (c == '(')
            {
                currentDepth++;
                maxDepth = Math.Max(maxDepth, currentDepth);
            }
            else if (c == ')')
            {
                currentDepth--;
            }
        }

        return maxDepth;
    }
}
```
Time Complexity: O(n) — single linear pass over the string.
Space Complexity: O(1) — only a couple of integer counters are used, no auxiliary data structure.

**Explanation:** The key insight is that a stack of parentheses only ever needs its *size* to answer this question, not the actual matched pairs, so a counter suffices. Dry run on `"(1+(2*3)+((8)/4))+1"`: depth goes 0→1 at first `(`, stays 1 through `1+`, →2 at next `(`, back to 1 after its `)`, stays 1 through `+`, →2 at next `(`, →3 at the next `(` (giving max = 3), back to 2 after `)` around `8`, back to 1 after `)` around `/4`, back to 0 after final `)`. Maximum recorded depth is 3.

---

## 3. Roman Number to Integer and Integer to Roman

### 3. Roman Number to Integer and Integer to Roman
**Problem Statement:** Convert a Roman numeral string to its integer value, and conversely convert an integer to its Roman numeral representation. Roman numerals use symbols `I=1, V=5, X=10, L=50, C=100, D=500, M=1000`, with subtractive notation (e.g., `IV=4`, `IX=9`, `XL=40`, `XC=90`, `CD=400`, `CM=900`).

**Example:**
- Input: `s = "LVIII"` (Roman to Integer)
- Output: `58`
- Explanation: `L = 50, V = 5, III = 3` → `50 + 5 + 3 = 58`.

- Input: `num = 1994` (Integer to Roman)
- Output: `"MCMXCIV"`
- Explanation: `1000 = M`, `900 = CM`, `90 = XC`, `4 = IV` → concatenated: `MCMXCIV`.

**Brute Force Approach:** For Roman to Integer, use a dictionary mapping each symbol to its value, then walk the string left to right. For each character, if it's less than the character to its right, subtract it; otherwise add it. For Integer to Roman, repeatedly find the largest Roman symbol value less than or equal to the remaining number and append its symbol, subtracting the value, looping until the number is 0 (checking every value one by one rather than using a precomputed lookup table).

```csharp
public class Solution
{
    public int RomanToIntBrute(string s)
    {
        Dictionary<char, int> values = new Dictionary<char, int>
        {
            {'I', 1}, {'V', 5}, {'X', 10}, {'L', 50},
            {'C', 100}, {'D', 500}, {'M', 1000}
        };

        int result = 0;
        for (int i = 0; i < s.Length; i++)
        {
            int current = values[s[i]];
            int next = (i + 1 < s.Length) ? values[s[i + 1]] : 0;

            if (current < next)
                result -= current;
            else
                result += current;
        }

        return result;
    }

    public string IntToRomanBrute(int num)
    {
        // Brute force: check each symbol individually, largest first
        int[] values = { 1000, 500, 100, 50, 10, 5, 1 };
        char[] symbols = { 'M', 'D', 'C', 'L', 'X', 'V', 'I' };

        StringBuilder result = new StringBuilder();

        while (num > 0)
        {
            // Find the largest single symbol <= num (does not directly handle
            // subtractive cases like 900 or 40, so we brute-force check them too)
            bool handled = false;

            int[] subtractiveValues = { 900, 400, 90, 40, 9, 4 };
            string[] subtractiveSymbols = { "CM", "CD", "XC", "XL", "IX", "IV" };

            for (int i = 0; i < subtractiveValues.Length; i++)
            {
                if (num >= subtractiveValues[i])
                {
                    result.Append(subtractiveSymbols[i]);
                    num -= subtractiveValues[i];
                    handled = true;
                    break;
                }
            }

            if (handled) continue;

            for (int i = 0; i < values.Length; i++)
            {
                if (num >= values[i])
                {
                    result.Append(symbols[i]);
                    num -= values[i];
                    handled = true;
                    break;
                }
            }
        }

        return result.ToString();
    }
}
```
Time Complexity: O(n) for Roman to Integer (n = length of string); O(1) for Integer to Roman since num is bounded (at most ~13 symbols appended for typical constraints like num <= 3999).
Space Complexity: O(1) auxiliary space (fixed-size dictionaries/arrays).

**Optimized Approach:** Combine values and subtractive combinations into a single sorted lookup table (array of value-symbol pairs from largest to smallest, including the six subtractive cases). Both conversions become a single clean loop using this table — no branching between "handled/not handled" cases. For Roman to Integer, the same left-to-right compare-with-next trick is already optimal at O(n); we simply keep it, using an array instead of a dictionary for slightly faster lookups.

```csharp
public class Solution
{
    // Single source of truth: value-symbol pairs, descending, including subtractive forms
    private static readonly int[] Values = { 1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1 };
    private static readonly string[] Symbols = { "M", "CM", "D", "CD", "C", "XC", "L", "XL", "X", "IX", "V", "IV", "I" };

    public int RomanToIntOptimal(string s)
    {
        int[] map = new int[128];
        map['I'] = 1; map['V'] = 5; map['X'] = 10; map['L'] = 50;
        map['C'] = 100; map['D'] = 500; map['M'] = 1000;

        int result = 0;
        int n = s.Length;

        for (int i = 0; i < n; i++)
        {
            int current = map[s[i]];
            if (i + 1 < n && current < map[s[i + 1]])
                result -= current;
            else
                result += current;
        }

        return result;
    }

    public string IntToRomanOptimal(int num)
    {
        StringBuilder result = new StringBuilder();

        for (int i = 0; i < Values.Length && num > 0; i++)
        {
            while (num >= Values[i])
            {
                num -= Values[i];
                result.Append(Symbols[i]);
            }
        }

        return result.ToString();
    }
}
```
Time Complexity: Roman to Integer is O(n) — one pass over the string with O(1) array lookups. Integer to Roman is O(1) amortized since the lookup table has a fixed size (13 entries) and each entry's `while` loop runs a small bounded number of times for constrained input ranges.
Space Complexity: O(1) — fixed-size arrays regardless of input size.

**Explanation:** The optimized Integer-to-Roman approach merges the "check subtractive pairs" and "check plain symbols" logic into one uniform table ordered from 1000 down to 1, so the algorithm never needs an if/else branch to decide which table to consult — it just walks one array. For `num = 1994`: at `Values[0]=1000` we append `"M"`, num becomes 994; at `Values[1]=900` we append `"CM"`, num becomes 94; skip 500, 400, 100; at `Values[5]=90` append `"XC"`, num becomes 4; skip down to `Values[11]=4` append `"IV"`, num becomes 0. Result: `"MCMXCIV"`.

---

## 4. Implement atoi (String to Integer)

### 4. Implement atoi (String to Integer)
**Problem Statement:** Implement the `myAtoi(string s)` function, which converts a string to a 32-bit signed integer, mimicking the behavior of the C/C++ `atoi` function. The algorithm should: skip leading whitespace; read an optional `'+'` or `'-'` sign; read digits until a non-digit character is found; convert those digits to an integer; and clamp the result to the 32-bit signed integer range `[-2147483648, 2147483647]` if it overflows. If no valid digits were read, return 0.

**Example:**
- Input: `s = "   -042"`
- Output: `-42`
- Explanation: Leading whitespace is skipped, `'-'` is read as sign, `"042"` is read as digits (leading zero ignored numerically), giving `-42`.

- Input: `s = "91283472332"`
- Output: `2147483647`
- Explanation: The parsed number `91283472332` exceeds `INT_MAX (2147483647)`, so it's clamped to `2147483647`.

**Brute Force Approach:** Extract the numeric substring (sign + digits) using string scanning, then convert it via `long.Parse` (using a 64-bit type to safely hold the value before clamping), and finally clamp the result to the 32-bit range.

```csharp
public class Solution
{
    public int MyAtoiBrute(string s)
    {
        int i = 0;
        int n = s.Length;

        // Skip leading whitespace
        while (i < n && s[i] == ' ') i++;

        if (i == n) return 0;

        int sign = 1;
        if (s[i] == '+' || s[i] == '-')
        {
            if (s[i] == '-') sign = -1;
            i++;
        }

        StringBuilder digits = new StringBuilder();
        while (i < n && char.IsDigit(s[i]))
        {
            digits.Append(s[i]);
            i++;
        }

        if (digits.Length == 0) return 0;

        // Use long to avoid overflow during parsing, then clamp
        long value;
        if (!long.TryParse(digits.ToString(), out value))
        {
            // Digit string too long even for long -> definitely overflow
            return sign == 1 ? int.MaxValue : int.MinValue;
        }

        value *= sign;

        if (value > int.MaxValue) return int.MaxValue;
        if (value < int.MinValue) return int.MinValue;

        return (int)value;
    }
}
```
Time Complexity: O(n) to scan the string, plus O(d) for parsing the digit substring (d = number of digit characters), so overall O(n).
Space Complexity: O(d) for the intermediate digit `StringBuilder`/string.

**Optimized Approach:** Use a single-pass state-machine style scan that builds the integer value digit-by-digit using a `long` accumulator (or careful `int` overflow checks), clamping as soon as the accumulator would exceed the 32-bit bounds — avoiding any intermediate string allocation entirely.

```csharp
public class Solution
{
    public int MyAtoiOptimal(string s)
    {
        int i = 0;
        int n = s.Length;

        // Step 1: skip leading whitespace
        while (i < n && s[i] == ' ') i++;
        if (i == n) return 0;

        // Step 2: optional sign
        int sign = 1;
        if (s[i] == '+' || s[i] == '-')
        {
            if (s[i] == '-') sign = -1;
            i++;
        }

        // Step 3: read digits, accumulate using long, clamp early to avoid overflow
        long result = 0;
        while (i < n && char.IsDigit(s[i]))
        {
            int digit = s[i] - '0';
            result = result * 10 + digit;

            // Early clamp check: if magnitude already exceeds int bounds, stop early
            if (sign == 1 && result > int.MaxValue)
            {
                return int.MaxValue;
            }
            if (sign == -1 && -result < int.MinValue)
            {
                return int.MinValue;
            }

            i++;
        }

        return (int)(sign * result);
    }
}
```
Time Complexity: O(n) — single pass through the string with no backtracking; whitespace skip, sign check, and digit accumulation each touch each character once. This is asymptotically the same as brute force but avoids extra allocations, and the early-exit clamp check means we stop as soon as overflow is detected rather than scanning trailing digits unnecessarily.
Space Complexity: O(1) — no intermediate string/array is built; only a few scalar variables (`i`, `sign`, `result`) are used.

**Explanation:** Dry run on `"   -042"`: skip 3 spaces (i=3), see `'-'` at i=3, set `sign=-1`, i=4. Read digits: `'0'`→result=0, `'4'`→result=4, `'2'`→result=42. Loop ends (end of string). Return `sign * result = -42`.

Dry run on `"91283472332"` (no sign, sign=1): accumulate digit by digit: 9, 91, 912, 9128, 91283, 912834, 9128347, 91283472, 912834723, then next digit `'3'` gives `result = 9128347233`. Check: `9128347233 > int.MaxValue (2147483647)` → true, so we immediately return `int.MaxValue = 2147483647` without even processing the final digit `'2'`. This demonstrates the early-clamp optimization saving unnecessary work on very long digit strings.

---

## 5. Count Number of Substrings with Exactly K Distinct Characters

### 5. Count Number of Substrings with Exactly K Distinct Characters
**Problem Statement:** Given a string `s` and an integer `k`, count the number of substrings of `s` that contain exactly `k` distinct characters.

**Example:**
- Input: `s = "pqpqs"`, `k = 2`
- Output: `7`
- Explanation: Substrings with exactly 2 distinct characters are: `"pq"`, `"pqp"`, `"pqpq"`, `"qp"`, `"qpq"`, `"pq"` (from index 2-3), `"qs"`. Counting all valid (start, end) occurrences gives 7.

**Brute Force Approach:** Generate every possible substring using two nested loops, and for each substring use a `HashSet<char>` to count distinct characters, checking if it equals `k`.

```csharp
public class Solution
{
    public int CountSubstringsExactlyKBrute(string s, int k)
    {
        int n = s.Length;
        int count = 0;

        for (int i = 0; i < n; i++)
        {
            HashSet<char> distinct = new HashSet<char>();
            for (int j = i; j < n; j++)
            {
                distinct.Add(s[j]);
                if (distinct.Count == k)
                {
                    count++;
                }
                else if (distinct.Count > k)
                {
                    break; // no point extending further from this start
                }
            }
        }

        return count;
    }
}
```
Time Complexity: O(n^2) — nested loop over all substring start/end pairs, with O(1) amortized HashSet insert per step.
Space Complexity: O(k) for the HashSet, at most O(26) for lowercase letters.

**Optimized Approach:** Use the identity `exactly(k) = atMost(k) - atMost(k-1)`, where `atMost(k)` counts substrings with **at most** `k` distinct characters using a sliding window with a frequency map. The window expands the right pointer, and whenever distinct character count exceeds `k`, the left pointer shrinks until it's back within bounds. For each valid right position, `atMost(k)` adds `(right - left + 1)` substrings (all substrings ending at `right` starting from `left` to `right`).

```csharp
public class Solution
{
    private int AtMostKDistinct(string s, int k)
    {
        if (k < 0) return 0;

        int n = s.Length;
        Dictionary<char, int> freq = new Dictionary<char, int>();
        int left = 0;
        int count = 0;

        for (int right = 0; right < n; right++)
        {
            char c = s[right];
            if (!freq.ContainsKey(c)) freq[c] = 0;
            freq[c]++;

            // Shrink window from the left while distinct chars exceed k
            while (freq.Count > k)
            {
                char leftChar = s[left];
                freq[leftChar]--;
                if (freq[leftChar] == 0)
                {
                    freq.Remove(leftChar);
                }
                left++;
            }

            // All substrings ending at 'right' starting anywhere in [left, right]
            // have at most k distinct characters
            count += (right - left + 1);
        }

        return count;
    }

    public int CountSubstringsExactlyKOptimal(string s, int k)
    {
        return AtMostKDistinct(s, k) - AtMostKDistinct(s, k - 1);
    }
}
```
Time Complexity: O(n) per `AtMostKDistinct` call — each character is added to the window once and removed at most once (amortized two-pointer technique), so two calls give O(n) overall (not O(2n) treated as separate O(n) terms collapses to O(n)).
Space Complexity: O(k) for the frequency dictionary inside the sliding window (bounded by alphabet size).

**Explanation:** The sliding window trick works because `atMost(k)` is monotonic and easy to compute incrementally: for each right endpoint, once the window `[left, right]` has at most `k` distinct chars, every substring `s[left..right], s[left+1..right], ..., s[right..right]` also has at most `k` distinct chars, so we add `right - left + 1` in one shot instead of enumerating them individually. Subtracting `atMost(k-1)` from `atMost(k)` removes substrings with fewer than `k` distinct chars, leaving exactly those with precisely `k`. For `s = "pqpqs"`, `k = 2`: `atMost(2) = 11` and `atMost(1) = 4`, giving `11 - 4 = 7`, matching the expected output.

---

## 6. Longest Palindromic Substring

### 6. Longest Palindromic Substring
**Problem Statement:** Given a string `s`, return the longest substring of `s` that is a palindrome. (Note: this section covers the brute-force and expand-around-center O(n^2) approaches; the optimal O(n) Manacher's Algorithm is covered separately in the advanced strings topic.)

**Example:**
- Input: `s = "babad"`
- Output: `"bab"`
- Explanation: `"bab"` is a palindrome of length 3. `"aba"` is also a valid answer since both are palindromic substrings of maximum length.

**Brute Force Approach:** Generate every substring using two nested loops, then check each one for the palindrome property by comparing it to its reverse (or a two-pointer check), keeping track of the longest palindrome found so far. This is O(n^2) substrings each taking up to O(n) to verify, giving O(n^3) overall.

```csharp
public class Solution
{
    private bool IsPalindrome(string s, int left, int right)
    {
        while (left < right)
        {
            if (s[left] != s[right]) return false;
            left++;
            right--;
        }
        return true;
    }

    public string LongestPalindromeBrute(string s)
    {
        int n = s.Length;
        if (n == 0) return "";

        int bestStart = 0;
        int bestLength = 1;

        for (int i = 0; i < n; i++)
        {
            for (int j = i; j < n; j++)
            {
                if (IsPalindrome(s, i, j))
                {
                    int length = j - i + 1;
                    if (length > bestLength)
                    {
                        bestLength = length;
                        bestStart = i;
                    }
                }
            }
        }

        return s.Substring(bestStart, bestLength);
    }
}
```
Time Complexity: O(n^3) — O(n^2) substrings, each palindrome check costs up to O(n).
Space Complexity: O(1) auxiliary space (excluding the output substring).

**Optimized Approach:** Use the Expand-Around-Center technique. A palindrome mirrors around its center, and there are `2n - 1` possible centers (n single-character centers for odd-length palindromes, and n-1 between-character centers for even-length palindromes). For each center, expand outward while characters match, and track the maximum length palindrome found. This reduces the complexity to O(n^2) since each expansion is O(n) but there are only O(n) centers, and it avoids the redundant O(n) palindrome-verification of the brute force.

```csharp
public class Solution
{
    private int ExpandAroundCenter(string s, int left, int right)
    {
        // Expands outward while s[left] == s[right]; returns length of palindrome
        while (left >= 0 && right < s.Length && s[left] == s[right])
        {
            left--;
            right++;
        }
        // When loop ends, left and right have gone one step too far
        return right - left - 1;
    }

    public string LongestPalindromeOptimal(string s)
    {
        if (string.IsNullOrEmpty(s)) return "";

        int start = 0;
        int end = 0;

        for (int center = 0; center < s.Length; center++)
        {
            // Odd-length palindromes centered at 'center'
            int len1 = ExpandAroundCenter(s, center, center);
            // Even-length palindromes centered between 'center' and 'center + 1'
            int len2 = ExpandAroundCenter(s, center, center + 1);

            int maxLen = Math.Max(len1, len2);

            if (maxLen > end - start + 1)
            {
                // Recompute the substring boundaries from the center and length
                start = center - (maxLen - 1) / 2;
                end = center + maxLen / 2;
            }
        }

        return s.Substring(start, end - start + 1);
    }
}
```
Time Complexity: O(n^2) — there are `2n - 1` centers, and each expansion can take up to O(n) in the worst case (e.g., a string of all identical characters), giving O(n^2) total, a clear improvement over the brute force's O(n^3).
Space Complexity: O(1) auxiliary space (excluding the output substring), since expansion uses only pointers.

**Explanation:** Dry run of Expand-Around-Center on `s = "babad"` (indices: b=0, a=1, b=2, a=3, d=4):
- center=0 ('b'): odd expand from (0,0) → s[0]='b', can't expand left (left=-1). len1=1. Even expand from (0,1): s[0]='b', s[1]='a', mismatch, len2=0. maxLen=1, current best is empty so update start=0, end=0 → "b".
- center=1 ('a'): odd expand from (1,1): left=0,right=2 → s[0]='b'==s[2]='b', expand further: left=-1 stop. len1 = 2-(-1)-1 = 3 (substring "bab"). Even expand from (1,2): s[1]='a', s[2]='b' mismatch, len2=0. maxLen=3 > current best (1), update: start = 1 - (3-1)/2 = 0, end = 1 + 3/2 = 2 → substring s[0..2] = "bab".
- center=2 ('b'): odd expand from (2,2): left=1,right=3 → s[1]='a'==s[3]='a', expand further: left=0,right=4 → s[0]='b', s[4]='d' mismatch, stop. len1 = 4-0-1=3 (substring "aba", indices 1..3). Even expand: s[2]='b', s[3]='a' mismatch, len2=0. maxLen=3, not greater than current best length 3, so no update (ties keep the first found, "bab").
- center=3 ('a'): odd len1=1, even mismatch len2=0. No update.
- center=4 ('d'): odd len1=1. No update.

Final answer: `"bab"` (length 3), matching the expected output. (`"aba"` would also be accepted as a valid alternative answer since it ties in length.)

---

## 7. Sum of Beauty of All Substrings

### 7. Sum of Beauty of All Substrings
**Problem Statement:** The beauty of a string is defined as the difference between the frequency of the most frequent character and the frequency of the least frequent character in that string. Given a string `s`, return the sum of beauty of all of its substrings.

**Example:**
- Input: `s = "aabcb"`
- Output: `5`
- Explanation: The substrings with non-zero beauty are `"aabc"` (beauty = 2-1 = 1), `"aabcb"` (beauty = 2-1 = 1), `"abcb"` (beauty = 2-1 = 1), `"aabcb"` variations etc. Summing beauty over all substrings gives a total of `5`.

**Brute Force Approach:** Generate every substring using two nested loops. For each substring, count the frequency of each of the 26 lowercase letters, then find the max and min non-zero frequency, and add `(max - min)` to the running total.

```csharp
public class Solution
{
    public int BeautySumBrute(string s)
    {
        int n = s.Length;
        int totalBeauty = 0;

        for (int i = 0; i < n; i++)
        {
            int[] freq = new int[26];

            for (int j = i; j < n; j++)
            {
                freq[s[j] - 'a']++;

                int maxFreq = 0;
                int minFreq = int.MaxValue;

                for (int f = 0; f < 26; f++)
                {
                    if (freq[f] > 0)
                    {
                        maxFreq = Math.Max(maxFreq, freq[f]);
                        minFreq = Math.Min(minFreq, freq[f]);
                    }
                }

                totalBeauty += (maxFreq - minFreq);
            }
        }

        return totalBeauty;
    }
}
```
Time Complexity: O(n^2 * 26) — for every substring (O(n^2) of them via the nested loop), we scan the 26-letter frequency array to find max/min, which is O(26) = O(1) but stated explicitly for clarity.
Space Complexity: O(26) = O(1) for the frequency array.

**Optimized Approach:** The core O(n^2 * 26) approach is essentially already near-optimal for this problem (there's no known way to compute this without considering all O(n^2) substrings in the general case), but we can restructure the loops to build the frequency array incrementally as `j` extends (avoiding recomputation of frequency counts from scratch for each substring) and inline the max/min scan efficiently. This keeps the complexity at O(n^2 * 26) but is a cleaner, more cache-friendly version of the same idea — expanding the substring one character at a time and updating max/min only for the changed character where possible.

```csharp
public class Solution
{
    public int BeautySumOptimal(string s)
    {
        int n = s.Length;
        int totalBeauty = 0;

        for (int i = 0; i < n; i++)
        {
            int[] freq = new int[26];

            for (int j = i; j < n; j++)
            {
                // Incrementally update frequency for the newly included character
                freq[s[j] - 'a']++;

                int maxFreq = 0;
                int minFreq = n + 1;

                // Scan only 26 buckets (constant work) to get max/min for current window
                for (int f = 0; f < 26; f++)
                {
                    if (freq[f] == 0) continue;
                    if (freq[f] > maxFreq) maxFreq = freq[f];
                    if (freq[f] < minFreq) minFreq = freq[f];
                }

                totalBeauty += (maxFreq - minFreq);
            }
        }

        return totalBeauty;
    }
}
```
Time Complexity: O(n^2) effectively, since the inner 26-scan is a constant factor (O(1)) independent of `n`; asymptotically this is O(n^2), the best known general approach for this problem because beauty depends on the full multiset of characters in each substring and there are O(n^2) substrings to consider.
Space Complexity: O(26) = O(1) for the reusable frequency array per outer loop iteration.

**Explanation:** Since the frequency array is reset once per outer loop (`i`) and incrementally updated as `j` grows, we avoid recomputing character counts from scratch for every single substring — each substring's frequency array is derived from the previous one by a single increment, turning what looks like redundant work into an O(1) update per step. The 26-bucket max/min scan is constant time regardless of substring length because the alphabet size is fixed, keeping the total work at O(n^2) substring-endpoints times O(26) scan = O(26 n^2), which simplifies to O(n^2). For `s = "aabcb"`: substrings starting at i=0 build up frequencies as `a`(1,0,0,0)->beauty 0, `aa`(2,0,0,0)->beauty 0, `aab`(2,1,0,0)->beauty 1, `aabc`(2,1,1,0)->beauty 1, `aabcb`(2,1,1,1)->beauty 1; continuing this process for all starting indices and summing every non-zero beauty value yields the total of 5.

---

## 8. Check if a String is a Palindrome After Removing At Most One Character

### 8. Check if a String is a Palindrome After Removing At Most One Character
**Problem Statement:** Given a string `s`, determine whether it can become a palindrome by removing **at most one character** from it. This is a natural extension of the standard "check palindrome" problem and is distinct from a plain reverse-words problem — it tests the two-pointer technique combined with a one-time "skip" decision.

**Example:**
- Input: `s = "abca"`
- Output: `true`
- Explanation: Removing the character `'b'` (or `'c'`) results in `"aca"`, which is a palindrome. So the answer is `true`.

**Brute Force Approach:** For each index `i` in the string, form a new string with the character at index `i` removed, and check if that resulting string is a palindrome. If any such removal (or the original string itself with zero removals) yields a palindrome, return true.

```csharp
public class Solution
{
    private bool IsPalindromeSimple(string s)
    {
        int left = 0;
        int right = s.Length - 1;
        while (left < right)
        {
            if (s[left] != s[right]) return false;
            left++;
            right--;
        }
        return true;
    }

    public bool ValidPalindromeBrute(string s)
    {
        // Check with zero removals first
        if (IsPalindromeSimple(s)) return true;

        // Try removing each character one at a time
        for (int i = 0; i < s.Length; i++)
        {
            string candidate = s.Remove(i, 1);
            if (IsPalindromeSimple(candidate))
            {
                return true;
            }
        }

        return false;
    }
}
```
Time Complexity: O(n^2) — for each of the n possible removal positions, building the candidate string and checking it for palindrome-ness costs O(n).
Space Complexity: O(n) for each candidate string created during the loop.

**Optimized Approach:** Use two pointers starting from both ends of the string. Move them toward each other while characters match. The moment a mismatch is found, there are only two possibilities to try: skip the left character (`left+1, right`) or skip the right character (`left, right-1`). Check if either resulting substring is a palindrome using a plain two-pointer palindrome check; because we only ever get one mismatch chance, this avoids generating any new string entirely.

```csharp
public class Solution
{
    private bool IsPalindromeRange(string s, int left, int right)
    {
        while (left < right)
        {
            if (s[left] != s[right]) return false;
            left++;
            right--;
        }
        return true;
    }

    public bool ValidPalindromeOptimal(string s)
    {
        int left = 0;
        int right = s.Length - 1;

        while (left < right)
        {
            if (s[left] != s[right])
            {
                // Exactly one mismatch allowed: try skipping either side
                return IsPalindromeRange(s, left + 1, right) ||
                       IsPalindromeRange(s, left, right - 1);
            }
            left++;
            right--;
        }

        // No mismatch found at all -- already a palindrome
        return true;
    }
}
```
Time Complexity: O(n) — the main two-pointer scan is O(n), and on the first (and only, since we return immediately) mismatch, we perform two O(n) palindrome checks, giving O(n) + O(n) + O(n) = O(n) overall, a significant improvement over the brute force's O(n^2).
Space Complexity: O(1) — only pointer variables are used; no new strings are allocated at any point.

**Explanation:** Because removing at most one character means we can tolerate exactly one mismatched pair of characters during the two-pointer scan, as soon as `s[left] != s[right]` occurs, the answer hinges entirely on whether skipping the left character or skipping the right character produces a palindrome in the remaining range — trying both covers every possibility since only one removal is allowed. Dry run on `s = "abca"`: left=0('a'), right=3('a') match, move to left=1('b'), right=2('c') — mismatch. Try skipping left: check range (2,2) which is a single character `"c"`, trivially a palindrome → returns `true`. So `ValidPalindromeOptimal` returns `true`, matching the expected output (this corresponds to removing `'b'`, leaving `"aca"`).
