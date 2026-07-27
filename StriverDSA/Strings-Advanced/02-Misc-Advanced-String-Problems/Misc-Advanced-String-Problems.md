# Strings (Advanced) — Miscellaneous Problems

## 1. Count and Say Sequence

**Problem Statement:**
The count-and-say sequence is a sequence of digit strings defined by the recursive formula:
- `countAndSay(1) = "1"`
- `countAndSay(n)` is the run-length encoded version of `countAndSay(n - 1)`.

To generate the run-length encoding of a string, scan it left to right, grouping consecutive identical characters together. For each group, output the count of characters in the group followed by the character itself.
Given a positive integer `n`, return the `n`th term of the count-and-say sequence.

**Example:**
- Input: `n = 4`
- Output: `"1211"`
- Explanation: `countAndSay(1) = "1"`, `countAndSay(2) = "11"` (one 1), `countAndSay(3) = "21"` (two 1s), `countAndSay(4) = "1211"` (one 2, one 1).

**Approach:**
Start with the base term `"1"`. Iteratively build each next term from the current term using run-length encoding: walk through the current string with a pointer, and while the next character equals the current one, extend the run and increment a counter. Once the run breaks (or the string ends), append the run's length followed by the repeated character to a `StringBuilder`. Move the pointer to the start of the next run and repeat until the whole string is consumed. Do this `n - 1` times starting from `"1"` to reach the `n`th term.

**Logic (Steps):**
1. Initialize `current = "1"` (term 1).
2. Loop `term` from `2` to `n`, replacing `current` with `RunLengthEncode(current)` each iteration.
3. Inside `RunLengthEncode`: start `i = 0`; while `i < s.Length`, record `currentChar = s[i]` and count how many consecutive characters equal it, advancing `i` for each.
4. Append the count followed by `currentChar` to a `StringBuilder`, then continue the outer while-loop from the new `i` (start of the next run).
5. After `n - 1` encodings, return the final `current` string.

```csharp
using System;
using System.Text;

public class CountAndSay
{
    public string CountAndSaySequence(int n)
    {
        string current = "1";

        for (int term = 2; term <= n; term++)
        {
            current = RunLengthEncode(current);
        }

        return current;
    }

    private string RunLengthEncode(string s)
    {
        StringBuilder result = new StringBuilder();
        int i = 0;

        while (i < s.Length)
        {
            char currentChar = s[i];
            int count = 0;

            while (i < s.Length && s[i] == currentChar)
            {
                count++;
                i++;
            }

            result.Append(count);
            result.Append(currentChar);
        }

        return result.ToString();
    }
}
```

**Time Complexity:** `O(n * L)`, where `L` is the length of the longest generated term (the string length can grow exponentially in the worst case, but for practical/interview-sized `n` this is treated as the dominant per-term encoding cost across `n - 1` iterations).
**Space Complexity:** `O(L)` to hold the current and next term (the `StringBuilder` buffer), where `L` is the length of the term being built.

**Walkthrough:** For `n = 4`: `term 1 = "1"` (base case). `term 2`: encode `"1"` → one run of `'1'` length 1 → `"1"+"1"` = `"11"`. `term 3`: encode `"11"` → one run of `'1'` length 2 → `"2"+"1"` = `"21"`. `term 4`: encode `"21"` → two runs: `'2'` length 1 → `"1"+"2"`, then `'1'` length 1 → `"1"+"1"` → result `"1211"`. Final answer `"1211"` matches the expected Output.

---

## 2. Compare Version Numbers

**Problem Statement:**
Given two version numbers `version1` and `version2` as strings, where each version consists of one or more revisions separated by dots (`'.'`), compare them. Each revision is a numeric string, but it may contain leading zeros (e.g., `"01"` represents the same value as `"1"`). If a version has fewer revisions than the other, treat the missing revisions as `0`. Return `1` if `version1 > version2`, `-1` if `version1 < version2`, and `0` if they are equal.

**Example:**
- Input: `version1 = "1.01", version2 = "1.001"`
- Output: `0`
- Explanation: Ignoring leading zeros, both `"01"` and `"001"` represent the numeric value `1`, and the first segments (`"1"` vs `"1"`) are equal, so the versions are equal.

**Approach:**
Split both version strings on `'.'` using `string.Split('.')` to get their revision segments as string arrays. Iterate over the maximum number of segments between the two arrays. For each index, retrieve the segment from each array if it exists, or use `"0"` as a default when one version has run out of segments (padding the shorter version). Convert each segment to an integer with `int.Parse`, which naturally strips any leading zeros. Compare the two integers: if they differ, immediately return `1` or `-1` accordingly. If every corresponding segment is equal after checking all segments, the versions are equal, so return `0`.

**Logic (Steps):**
1. Split `version1` and `version2` on `'.'` into `segments1` and `segments2`.
2. Compute `maxLength = Max(segments1.Length, segments2.Length)` to cover the longer version.
3. For each index `i` from `0` to `maxLength - 1`, read `value1` from `segments1[i]` (or `0` if out of range) and `value2` similarly from `segments2[i]`, using `int.Parse` to strip leading zeros.
4. If `value1 != value2`, return `1` (if `value1 > value2`) or `-1` immediately.
5. If the loop finishes with no differences found, return `0`.

```csharp
using System;

public class CompareVersionNumbers
{
    public int CompareVersion(string version1, string version2)
    {
        string[] segments1 = version1.Split('.');
        string[] segments2 = version2.Split('.');

        int maxLength = Math.Max(segments1.Length, segments2.Length);

        for (int i = 0; i < maxLength; i++)
        {
            int value1 = i < segments1.Length ? int.Parse(segments1[i]) : 0;
            int value2 = i < segments2.Length ? int.Parse(segments2[i]) : 0;

            if (value1 != value2)
            {
                return value1 > value2 ? 1 : -1;
            }
        }

        return 0;
    }
}
```

**Time Complexity:** `O(n + m)`, where `n` and `m` are the lengths of `version1` and `version2` respectively (splitting and parsing each segment is linear in the total input length).
**Space Complexity:** `O(n + m)` for storing the split segment arrays of both versions.

**Walkthrough:** For `version1 = "1.01"`, `version2 = "1.001"`: split gives `segments1 = ["1","01"]`, `segments2 = ["1","001"]`, `maxLength = 2`. Index 0: `value1 = int.Parse("1") = 1`, `value2 = 1` → equal. Index 1: `value1 = int.Parse("01") = 1` (leading zero stripped), `value2 = int.Parse("001") = 1` → equal. Loop ends with no differences → return `0`, matching the expected Output. (Padding case for reference: `"1.0"` vs `"1.0.0"` — at index 2, `segments1` has no element so `value1` defaults to `0`, matching `segments2[2] = 0`, again returning `0`.)
