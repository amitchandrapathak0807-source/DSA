# Strings — Basic String Problems

## 1. Remove Outermost Parentheses

**Problem Statement:** Given a valid parentheses string `s` consisting only of `(` and `)`, and made up of one or more "primitive" valid parentheses strings concatenated together, remove the outermost parentheses of every primitive substring and return the resulting string. A primitive string is a non-empty valid parentheses string that cannot be split into two non-empty valid parentheses strings.

**Example:**
- Input: `s = "(()())(())"`
- Output: `"()()()"`
- Explanation: The primitive strings are `"(()())"` and `"(())"`. Removing the outermost parentheses of each gives `"()()"` and `"()"`, which concatenate to `"()()()"`.

**Brute Force Approach:** Split the string into primitive substrings by tracking balance with a counter — whenever the running open/close count returns to zero, that marks the end of one primitive substring. Collect each primitive substring, strip its first and last character, and append the remainder to the result using a `List<char>` or `StringBuilder`.

```csharp
public static string RemoveOuterParentheses(string s)
{
    List<string> primitives = new List<string>();
    int start = 0;
    int balance = 0;

    for (int i = 0; i < s.Length; i++)
    {
        if (s[i] == '(')
            balance++;
        else
            balance--;

        if (balance == 0)
        {
            primitives.Add(s.Substring(start, i - start + 1));
            start = i + 1;
        }
    }

    StringBuilder result = new StringBuilder();
    foreach (string primitive in primitives)
    {
        // strip outermost '(' and ')'
        result.Append(primitive.Substring(1, primitive.Length - 2));
    }

    return result.ToString();
}
```
Time Complexity: O(n) for the split pass plus O(n) for building the result — overall O(n), but with extra overhead from creating substrings.
Space Complexity: O(n) for the `List<string>` of primitives plus O(n) for the result `StringBuilder`.

**Optimized Approach:** Use a single depth counter while scanning the string once. For each `(`, only append it to the result if the current depth is greater than 0 (i.e., it is not an outermost opening bracket), then increment depth. For each `)`, decrement depth first, then only append it if depth is still greater than 0 (i.e., it is not an outermost closing bracket). This avoids any substring extraction or extra list.

```csharp
public static string RemoveOuterParenthesesOptimized(string s)
{
    StringBuilder result = new StringBuilder();
    int depth = 0;

    foreach (char c in s)
    {
        if (c == '(')
        {
            if (depth > 0)
                result.Append(c);
            depth++;
        }
        else // c == ')'
        {
            depth--;
            if (depth > 0)
                result.Append(c);
        }
    }

    return result.ToString();
}
```
Time Complexity: O(n) — single pass over the string with O(1) work per character.
Space Complexity: O(n) for the output `StringBuilder` (excluding the output, extra space is O(1) since only a depth counter is used).

**Explanation:** Dry run of the depth-counter trick on `s = "(()())(())"`:

| char | action | depth before | depth after | appended? |
|------|--------|--------------|--------------|-----------|
| `(`  | outer open | 0 | 1 | no (depth was 0) |
| `(`  | inner open | 1 | 2 | yes → `(` |
| `)`  | inner close | 2 | 1 | yes → `)` |
| `(`  | inner open | 1 | 2 | yes → `(` |
| `)`  | inner close | 2 | 1 | yes → `)` |
| `)`  | outer close | 1 | 0 | no (depth becomes 0) |
| `(`  | outer open | 0 | 1 | no |
| `(`  | inner open | 1 | 2 | yes → `(` |
| `)`  | inner close | 2 | 1 | yes → `)` |
| `)`  | outer close | 1 | 0 | no |

Appended characters in order: `( ) ( ) ( )` → result = `"()()()"`, matching the expected output.

---

## 2. Reverse Words in a String

**Problem Statement:** Given an input string `s`, reverse the order of the words. A word is a maximal sequence of non-space characters. Words in the result should be separated by a single space, and leading/trailing/extra spaces between words should be removed.

**Example:**
- Input: `s = "  the sky   is blue "`
- Output: `"blue is sky the"`
- Explanation: The words `["the", "sky", "is", "blue"]` are reversed to `["blue", "is", "sky", "the"]`, joined with single spaces, and all extra whitespace is discarded.

**Brute Force Approach:** Split the string on whitespace (using an overload that discards empty entries), then push each word onto a stack (or simply iterate the resulting array from the end), and join them with single spaces using a `List<string>` / `StringBuilder`.

```csharp
public static string ReverseWords(string s)
{
    string[] words = s.Split(' ', StringSplitOptions.RemoveEmptyEntries);
    List<string> collected = new List<string>();

    for (int i = words.Length - 1; i >= 0; i--)
    {
        collected.Add(words[i]);
    }

    return string.Join(" ", collected);
}
```
Time Complexity: O(n), where n is the length of the string — splitting and rejoining each take linear time.
Space Complexity: O(n) for the array of words and the intermediate list.

**Optimized Approach:** Do it in a single pass without relying on `Split`: scan the string from right to left, collecting characters of the current word into a `StringBuilder` buffer; whenever a space boundary is hit and the buffer is non-empty, append the buffer (plus a separating space) to the result, then clear the buffer. This mimics an in-place "split and reverse" without building an intermediate array of words.

```csharp
public static string ReverseWordsOptimized(string s)
{
    StringBuilder result = new StringBuilder();
    StringBuilder word = new StringBuilder();
    int i = s.Length - 1;

    while (i >= 0)
    {
        // skip trailing/extra spaces
        while (i >= 0 && s[i] == ' ')
            i--;

        if (i < 0)
            break;

        // collect one word (scanning backwards, then reverse it)
        word.Clear();
        while (i >= 0 && s[i] != ' ')
        {
            word.Insert(0, s[i]);
            i--;
        }

        if (result.Length > 0)
            result.Append(' ');
        result.Append(word);
    }

    return result.ToString();
}
```
Time Complexity: O(n) — every character is visited a constant number of times (once to skip/scan, once to insert into the word buffer), no library split overhead.
Space Complexity: O(n) for the result and temporary word buffer; no auxiliary array of words is created.

**Explanation:** For `s = "  the sky   is blue "`, scanning from the right: skip the trailing space, read `"blue"`, append to result → `"blue"`. Skip spaces, read `"is"` → result `"blue is"`. Skip spaces, read `"sky"` → result `"blue is sky"`. Skip spaces, read `"the"` → result `"blue is sky the"`. Skip leading spaces, `i` becomes negative, loop ends. Final output: `"blue is sky the"`.

---

## 3. Largest Odd Number in a String

**Problem Statement:** Given a string `num` representing a large non-negative integer, return the largest-valued odd substring that is a prefix of `num` (i.e., remove trailing digits from the end until the remaining prefix ends in an odd digit). If no odd digit exists, return an empty string.

**Example:**
- Input: `num = "35427"`
- Output: `"35427"`
- Explanation: The last digit `7` is already odd, so the entire string is the largest odd prefix.

Second example:
- Input: `num = "1234"`
- Output: `"123"`
- Explanation: `4` is even, so it is dropped; `3` is odd, so `"123"` is the answer.

**Brute Force Approach:** Try every prefix length from the full string down to length 1, convert the last character of each candidate prefix to check parity, and return the first (longest) prefix whose last digit is odd. This can be done by building substrings explicitly.

```csharp
public static string LargestOddNumber(string num)
{
    for (int length = num.Length; length >= 1; length--)
    {
        string candidate = num.Substring(0, length);
        int lastDigit = candidate[candidate.Length - 1] - '0';
        if (lastDigit % 2 != 0)
        {
            return candidate;
        }
    }
    return "";
}
```
Time Complexity: O(n^2) in the worst case, since `Substring` creates a new string of up to length n for each of up to n iterations.
Space Complexity: O(n) for the candidate substring created in each iteration (not counting the returned result).

**Optimized Approach:** Scan the string once from the last character toward the front, checking parity directly on the character without building any substring. Stop at the first (rightmost) odd digit found and return `num.Substring(0, i + 1)` — only one substring is ever created, for the final answer.

```csharp
public static string LargestOddNumberOptimized(string num)
{
    for (int i = num.Length - 1; i >= 0; i--)
    {
        int digit = num[i] - '0';
        if (digit % 2 != 0)
        {
            return num.Substring(0, i + 1);
        }
    }
    return "";
}
```
Time Complexity: O(n) — a single backward scan; only one `Substring` call is made (for the final result), not one per iteration.
Space Complexity: O(1) auxiliary space (excluding the single output substring), since no intermediate candidates are built.

**Explanation:** Brief dry run for `num = "1234"`: start at index 3 (`'4'`), even, skip. Index 2 (`'3'`), odd — stop here and return `num.Substring(0, 3)` = `"123"`, matching the expected output.

---

## 4. Longest Common Prefix

**Problem Statement:** Given an array of strings `strs`, find the longest common prefix string amongst all the strings in the array. If there is no common prefix, return an empty string `""`.

**Example:**
- Input: `strs = ["flower", "flow", "flight"]`
- Output: `"fl"`
- Explanation: `"fl"` is the longest string that is a prefix of all three input strings; `"flo"` fails because `"flight"` doesn't start with it.

**Brute Force Approach:** Take the first string as a candidate prefix. For each candidate length from the full length of the first string down to 0, check (using nested loops / an extra `List<char>`) whether every other string in the array starts with that candidate prefix; return the first one that works.

```csharp
public static string LongestCommonPrefix(string[] strs)
{
    if (strs == null || strs.Length == 0)
        return "";

    string first = strs[0];

    for (int length = first.Length; length >= 1; length--)
    {
        string candidate = first.Substring(0, length);
        bool isCommon = true;

        for (int i = 1; i < strs.Length; i++)
        {
            if (!strs[i].StartsWith(candidate, StringComparison.Ordinal))
            {
                isCommon = false;
                break;
            }
        }

        if (isCommon)
            return candidate;
    }

    return "";
}
```
Time Complexity: O(m^2 * n) in the worst case, where m is the length of the first string and n is the number of strings — for each of the m candidate lengths, up to n strings are checked with an O(m) `StartsWith`/substring comparison.
Space Complexity: O(m) for each candidate substring created.

**Optimized Approach:** Compare characters column by column across all strings simultaneously. For each character position `i` starting at 0, take `strs[0][i]` as the reference character and check that every other string has the same character at position `i` (and is long enough). Stop at the first mismatch or when the shortest string is exhausted, and return the prefix built up to that point.

```csharp
public static string LongestCommonPrefixOptimized(string[] strs)
{
    if (strs == null || strs.Length == 0)
        return "";

    for (int i = 0; i < strs[0].Length; i++)
    {
        char c = strs[0][i];

        for (int j = 1; j < strs.Length; j++)
        {
            if (i == strs[j].Length || strs[j][i] != c)
            {
                return strs[0].Substring(0, i);
            }
        }
    }

    return strs[0];
}
```
Time Complexity: O(m * n), where m is the length of the shortest common prefix (bounded by the shortest string) and n is the number of strings — each character position is checked across all strings exactly once, with early exit on mismatch. This avoids the repeated re-scanning of the brute force approach.
Space Complexity: O(1) auxiliary space (excluding the single output substring), since comparisons are done character-by-character in place.

**Explanation:** Brief walk-through for `strs = ["flower", "flow", "flight"]`: at `i = 0`, `'f'` matches in all three. At `i = 1`, `'l'` matches. At `i = 2`, reference is `'o'` from `"flower"`, but `"flight"` has `'i'` at index 2 — mismatch, so return `strs[0].Substring(0, 2)` = `"fl"`.

---

## 5. Isomorphic Strings

**Problem Statement:** Given two strings `s` and `t`, determine if they are isomorphic. Two strings are isomorphic if the characters in `s` can be replaced to get `t`, such that there is a consistent one-to-one mapping from every character in `s` to a character in `t` (no two characters map to the same character, and the mapping must be consistent in both directions).

**Example:**
- Input: `s = "egg"`, `t = "add"`
- Output: `true`
- Explanation: `'e' -> 'a'` and `'g' -> 'd'` is a consistent, one-to-one mapping.

Second example:
- Input: `s = "foo"`, `t = "bar"`
- Output: `false`
- Explanation: `'o'` would need to map to both `'o'` and `'a'`, which is not consistent.

**Brute Force Approach:** Use two `Dictionary<char, char>` objects (one for `s -> t` and one for `t -> s`) to track mappings as extra data structures, checking both directions at every index to ensure the mapping is consistent and one-to-one.

```csharp
public static bool IsIsomorphic(string s, string t)
{
    if (s.Length != t.Length)
        return false;

    Dictionary<char, char> mapST = new Dictionary<char, char>();
    Dictionary<char, char> mapTS = new Dictionary<char, char>();

    for (int i = 0; i < s.Length; i++)
    {
        char cs = s[i];
        char ct = t[i];

        if (mapST.ContainsKey(cs))
        {
            if (mapST[cs] != ct)
                return false;
        }
        else
        {
            mapST[cs] = ct;
        }

        if (mapTS.ContainsKey(ct))
        {
            if (mapTS[ct] != cs)
                return false;
        }
        else
        {
            mapTS[ct] = cs;
        }
    }

    return true;
}
```
Time Complexity: O(n), where n is the length of the strings — one pass with O(1) dictionary lookups.
Space Complexity: O(k), where k is the number of distinct characters (bounded by the character set size, e.g. 256 for extended ASCII), for the two dictionaries.

**Optimized Approach:** Replace the two dictionaries with two fixed-size integer arrays of size 256 (for ASCII characters), storing the "last seen position + 1" of each character in `s` and `t`. At each index, compare whether the last-seen position of `s[i]` in `s` matches the last-seen position of `t[i]` in `t`; if they differ, the mapping is inconsistent. This avoids hashing overhead and uses simple array indexing.

```csharp
public static bool IsIsomorphicOptimized(string s, string t)
{
    if (s.Length != t.Length)
        return false;

    int[] lastSeenS = new int[256];
    int[] lastSeenT = new int[256];

    for (int i = 0; i < s.Length; i++)
    {
        char cs = s[i];
        char ct = t[i];

        if (lastSeenS[cs] != lastSeenT[ct])
            return false;

        lastSeenS[cs] = i + 1;
        lastSeenT[ct] = i + 1;
    }

    return true;
}
```
Time Complexity: O(n) — single pass, O(1) array access per character (no hashing).
Space Complexity: O(1) — the two arrays have a fixed size of 256 regardless of input length.

**Explanation:** Brief reasoning: storing "last seen index + 1" (0 means "never seen") lets one comparison check both that the current mapping matches an existing one and that no other character already maps to the target — if `s[i]` and `t[i]` were last seen at different points, the mapping is broken. For `s = "egg"`, `t = "add"`: at `i=0`, both unseen (0 == 0), record position 1 for `'e'` and `'a'`. At `i=1`, `'g'` and `'d'` both unseen (0 == 0), record position 2. At `i=2`, `'g'` was last seen at position 2, `'d'` was last seen at position 2 — match, update to position 3. All checks pass → `true`.

---

## 6. Check Whether One String is a Rotation of Another

**Problem Statement:** Given two strings `s1` and `s2`, determine whether `s2` is a rotation of `s1` — i.e., whether `s2` can be obtained by taking some prefix of `s1` and moving it to the end (equivalently, cyclically shifting `s1`).

**Example:**
- Input: `s1 = "waterbottle"`, `s2 = "erbottlewat"`
- Output: `true`
- Explanation: Rotating `s1` by moving the prefix `"wat"` to the end gives `"erbottlewat"`, which equals `s2`.

**Brute Force Approach:** Try every possible rotation point explicitly: for each index `i` from 0 to `s1.Length - 1`, build the rotated string by concatenating `s1.Substring(i)` and `s1.Substring(0, i)` using a `StringBuilder`, and compare it against `s2`.

```csharp
public static bool IsRotation(string s1, string s2)
{
    if (s1.Length != s2.Length || s1.Length == 0)
        return s1 == s2;

    for (int i = 0; i < s1.Length; i++)
    {
        StringBuilder rotated = new StringBuilder();
        rotated.Append(s1.Substring(i));
        rotated.Append(s1.Substring(0, i));

        if (rotated.ToString() == s2)
        {
            return true;
        }
    }

    return false;
}
```
Time Complexity: O(n^2) — for each of the n possible rotation points, building the rotated string and comparing it takes O(n).
Space Complexity: O(n) for the `StringBuilder` rotated string built on each iteration.

**Optimized Approach:** Use the classic concatenation trick: if `s2` is a rotation of `s1`, then `s2` must appear as a contiguous substring of `s1 + s1`. This is because concatenating `s1` with itself lays out every possible rotation as a substring. So simply check length equality first, then check if `(s1 + s1).Contains(s2)`.

```csharp
public static bool IsRotationOptimized(string s1, string s2)
{
    if (s1.Length != s2.Length)
        return false;

    string doubled = s1 + s1;
    return doubled.Contains(s2);
}
```
Time Complexity: O(n) on average, where n is the length of the strings — building `s1 + s1` is O(n), and `Contains` (using an efficient substring search) runs in O(n) on average (O(n^2) worst case with a naive search, but .NET's implementation performs well in practice).
Space Complexity: O(n) for the doubled string `s1 + s1`.

**Explanation:** Dry run of the `(s1 + s1).Contains(s2)` trick on `s1 = "waterbottle"`, `s2 = "erbottlewat"`:
`doubled = "waterbottle" + "waterbottle" = "waterbottlewaterbottle"`.
Scanning for `"erbottlewat"` inside `doubled`: starting at index 2 of `doubled` (`"erbottlewaterbottle"`), the substring `"erbottlewat"` (11 characters) matches exactly — `doubled.Substring(2, 11) == "erbottlewat"`. Since the substring is found, `Contains` returns `true`, confirming `s2` is a rotation of `s1`.

---

## 7. Check if Two Strings are Anagrams of Each Other

**Problem Statement:** Given two strings `s` and `t`, determine if `t` is an anagram of `s` — that is, both strings use exactly the same characters with exactly the same frequencies, possibly in a different order.

**Example:**
- Input: `s = "anagram"`, `t = "nagaram"`
- Output: `true`
- Explanation: Both strings contain the same multiset of characters: `{a:3, n:2, g:1, r:1, m:1}`.

Second example:
- Input: `s = "rat"`, `t = "car"`
- Output: `false`
- Explanation: `"rat"` contains `'t'` but `"car"` does not, so the character frequencies differ.

**Brute Force Approach:** Convert both strings into `char[]` arrays, sort each array, and compare the sorted arrays (or the resulting strings) for equality. If they match exactly, the strings are anagrams.

```csharp
public static bool IsAnagram(string s, string t)
{
    if (s.Length != t.Length)
        return false;

    char[] sChars = s.ToCharArray();
    char[] tChars = t.ToCharArray();

    Array.Sort(sChars);
    Array.Sort(tChars);

    for (int i = 0; i < sChars.Length; i++)
    {
        if (sChars[i] != tChars[i])
        {
            return false;
        }
    }

    return true;
}
```
Time Complexity: O(n log n), dominated by sorting both character arrays.
Space Complexity: O(n) for the two character arrays created by `ToCharArray()`.

**Optimized Approach:** Use a fixed-size frequency array of size 26 (for lowercase English letters) instead of sorting. Increment counts for characters in `s` and decrement counts for characters in `t` in a single combined pass, then verify every count is zero. This avoids sorting entirely.

```csharp
public static bool IsAnagramOptimized(string s, string t)
{
    if (s.Length != t.Length)
        return false;

    int[] frequency = new int[26];

    for (int i = 0; i < s.Length; i++)
    {
        frequency[s[i] - 'a']++;
        frequency[t[i] - 'a']--;
    }

    foreach (int count in frequency)
    {
        if (count != 0)
        {
            return false;
        }
    }

    return true;
}
```
Time Complexity: O(n) — one pass to accumulate frequencies plus a constant-time pass over the fixed 26-element array, versus O(n log n) for sorting.
Space Complexity: O(1) — the frequency array has a fixed size of 26 regardless of input length.

**Explanation:** Brief reasoning: for `s = "anagram"`, `t = "nagaram"`, every character increment from `s` is exactly canceled out by a matching decrement from `t` because both strings contain the same letters with the same counts, so all 26 buckets end at 0 and the function returns `true`. For `s = "rat"`, `t = "car"`, the bucket for `'t'` ends at `+1` (incremented by `s`, never decremented by `t`) and the bucket for `'c'` ends at `-1`, so the final check finds a non-zero count and returns `false`.
