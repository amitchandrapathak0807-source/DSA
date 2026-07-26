# Tries — Advanced: XOR and Substrings

## Concept: Binary Trie for XOR Problems

A **bit trie** (also called a binary trie or XOR trie) is the same idea as a character trie, except instead of inserting a word letter-by-letter, we insert a number **bit-by-bit**, using its fixed-width binary representation (commonly 32 bits, since most competitive programming constraints fit within a 32-bit integer). Each number is treated as a "string" over the alphabet `{0, 1}`, and we insert it into the trie **most significant bit (MSB) first**, so numbers that share a common high-order prefix share the same trie path near the root, and diverge lower down where their bits differ.

Because the alphabet size is only 2, every node in a bit trie has **at most 2 children** — one for the `0` branch and one for the `1` branch. This makes the trie compact and every insertion/search take exactly `O(number of bits)` time (e.g., `O(32)`), independent of how many numbers are stored.

The real power of a bit trie is answering **"maximize XOR with this number"** queries greedily:

- Recall that XOR of two bits is `1` only when the bits differ, and `0` when they are the same.
- To maximize the XOR of a query number `x` against a set of numbers stored in the trie, we want the **most significant bits to differ** first (since higher bit positions contribute more to the final value), and only settle for matching bits when no other option exists.
- So, walking the trie from the root, at each bit position of `x`, we look at the current bit `b`. We **try to go to the child representing `1 - b` (the opposite bit)** first. If that child exists in the trie, we take it (this guarantees a `1` in the XOR result at this position) and add this bit's positional value to our running XOR result. If the opposite-bit child does not exist, we are forced to go to the same-bit child (contributing a `0` at this position).
- After walking through all bit positions, the path we traced corresponds to the number in the trie that produces the maximum possible XOR with `x`.

This greedy bit-by-bit strategy is optimal because higher bit positions dominate the numeric value of the result — it is always better to fix a higher bit to `1` even if it forces all lower bits to be suboptimal, exactly analogous to how greedy digit-maximization works in numeral systems.

We will use this concept, plus a regular character trie for substring counting, to solve three classic problems below.

---

## 1. Count the Number of Distinct Substrings in a String (using a Trie)

**Problem Statement:**
Given a string `s`, count the number of **distinct (non-empty) substrings** of `s`, using a Trie-based approach.

**Example:**
- Input: `s = "ababa"`
- Output: `10`
- Explanation: The distinct substrings of `"ababa"` are: `a, b, ab, ba, aba, bab, abab, baba, ababa, aa`... let's verify precisely: all substrings are `a, b, ab, ba, ba(2nd), ab(2nd), aba, bab, aba(2nd), abab, baba, ababa` etc. After removing duplicates, the distinct set is `{a, b, ab, ba, aba, bab, abab, baba, ababa}` plus the empty consideration handled separately — for `"ababa"` the correct distinct-substring count (non-empty) is `10`. We do not count the empty substring.

**Brute Force Approach:**
Generate every substring `s[i..j]` for all `0 <= i <= j < n` and insert each into a `HashSet<string>`. The set automatically deduplicates. The final answer is the size of the set. There are `O(n^2)` substrings, and each substring can take up to `O(n)` to hash/compare/store, giving `O(n^3)` in the worst case (or `O(n^2)` amortized substring generation with `O(n)` extra per insertion for hashing), commonly quoted as `O(n^2 * L)` where `L` is average substring length.

```csharp
using System;
using System.Collections.Generic;

public class DistinctSubstringsBruteForce
{
    public int CountDistinctSubstrings(string s)
    {
        int n = s.Length;
        HashSet<string> distinct = new HashSet<string>();

        for (int i = 0; i < n; i++)
        {
            for (int j = i; j < n; j++)
            {
                string sub = s.Substring(i, j - i + 1);
                distinct.Add(sub);
            }
        }

        return distinct.Count;
    }
}
```

Time Complexity: `O(n^2 * L)` where `L` is the average length of generated substrings (substring creation is `O(length)`, and hashing/comparing strings in the `HashSet` is also `O(length)`); effectively `O(n^3)` worst case.
Space Complexity: `O(n^2 * L)` to store all distinct substrings (or `O(n^3)` worst case), since in the worst case (all characters distinct) nearly all `O(n^2)` substrings are distinct and stored fully.

**Optimized Trie-Based Approach:**
Insert every **suffix** of the string into a Trie, character by character. Each time an insertion creates a **new node** (i.e., the required child did not already exist), that corresponds to a **new distinct substring**, because every substring of `s` is a prefix of some suffix of `s`, and every prefix of every suffix corresponds to exactly one path from the root in the trie. Counting new nodes created across all suffix insertions gives the count of distinct substrings directly, without ever materializing substring strings.

```csharp
using System;
using System.Collections.Generic;

public class TrieNode
{
    public Dictionary<char, TrieNode> Children = new Dictionary<char, TrieNode>();
}

public class DistinctSubstringsTrie
{
    public int CountDistinctSubstrings(string s)
    {
        int n = s.Length;
        TrieNode root = new TrieNode();
        int distinctCount = 0;

        for (int i = 0; i < n; i++)
        {
            TrieNode current = root;
            // Insert the suffix s[i..n-1] into the trie
            for (int j = i; j < n; j++)
            {
                char c = s[j];
                if (!current.Children.ContainsKey(c))
                {
                    current.Children[c] = new TrieNode();
                    distinctCount++; // a brand-new node = a brand-new distinct substring
                }
                current = current.Children[c];
            }
        }

        return distinctCount;
    }
}
```

Time Complexity: `O(n^2)` — there are `n` suffixes, and inserting the `i`-th suffix (of length `n - i`) takes `O(n - i)` time; the sum over all suffixes is `O(n^2)`. This still touches every substring exactly once via shared trie prefixes, but with a much better constant factor than hashing full-length substring strings, since each character comparison/insertion is `O(1)` instead of `O(length)`.
Space Complexity: `O(n^2)` in the worst case (e.g., a string of all distinct characters produces `O(n^2)` trie nodes, one per distinct substring), though in practice with repeated characters the trie shares many prefixes and uses far less space.

**Explanation:**
Dry run on `s = "ababa"` (indices 0..4, characters `a,b,a,b,a`):

- Insert suffix starting at `i=0`: `"ababa"` → path `a → b → a → b → a`. All 5 nodes are new. `distinctCount = 5`. New substrings found: `a, ab, aba, abab, ababa`.
- Insert suffix starting at `i=1`: `"baba"` → path `b → a → b → a`. Root's `b` child does not exist yet, so it's new; then `a`, `b`, `a` under it are all new (this is a different branch than the first suffix, since the first suffix started with `a`). All 4 nodes are new. `distinctCount = 5 + 4 = 9`. New substrings found: `b, ba, bab, baba`.
- Insert suffix starting at `i=2`: `"aba"` → path `a → b → a`. Root's `a` child already exists (from suffix 1), its `b` child already exists (from suffix 1's `a→b`), and that `b`'s `a` child already exists (from suffix 1's `a→b→a`). No new nodes. `distinctCount` stays `9`. (All of `a, ab, aba` were already counted.)
- Insert suffix starting at `i=3`: `"ba"` → path `b → a`. Root's `b` child exists (from suffix 2), and its `a` child exists (from suffix 2's `b→a`). No new nodes. `distinctCount` stays `9`.
- Insert suffix starting at `i=4`: `"a"` → path `a`. Root's `a` child exists. No new nodes. `distinctCount` stays `9`.

Wait — running this carefully gives `9`, but let's recheck against the known answer. Actually for `"ababa"` the well-known distinct substring count is `9` (not `10`) when counting only non-empty substrings — the set is exactly `{a, b, ab, ba, aba, bab, abab, baba, ababa}`, which has 9 elements. The trie method above confirms this: `distinctCount = 9`. This dry run shows precisely why: only the first two suffixes (the two longest, which explore genuinely new branches) contribute new nodes; every shorter suffix is already fully absorbed as a prefix of a longer suffix's path in the trie, so it contributes zero new distinct substrings.

---

## 2. Maximum XOR of Two Numbers in an Array

**Problem Statement:**
Given an array of integers `nums`, find the maximum possible value of `nums[i] XOR nums[j]` over all pairs `i, j` (including `i == j`, though the max is typically achieved with distinct indices).

**Example:**
- Input: `nums = [3, 10, 5, 25, 2, 8]`
- Output: `28`
- Explanation: The maximum XOR pair is `5 XOR 25 = 28`, since `5 = 00101` and `25 = 11001`, and `00101 XOR 11001 = 11100 = 28`.

**Brute Force Approach:**
Check every pair `(i, j)` with `i < j`, compute `nums[i] XOR nums[j]`, and track the maximum.

```csharp
using System;

public class MaxXorBruteForce
{
    public int FindMaximumXOR(int[] nums)
    {
        int maxXor = 0;
        int n = nums.Length;

        for (int i = 0; i < n; i++)
        {
            for (int j = i + 1; j < n; j++)
            {
                int currentXor = nums[i] ^ nums[j];
                maxXor = Math.Max(maxXor, currentXor);
            }
        }

        return maxXor;
    }
}
```

Time Complexity: `O(n^2)` — every pair is checked, and XOR itself is `O(1)` for fixed-width integers.
Space Complexity: `O(1)` extra space.

**Optimized Trie-Based Approach:**
Build a bit trie by inserting the 32-bit binary representation of every number (MSB first). Then, for each number `x` in the array, walk the trie greedily: at each bit position, try to move to the child representing the **opposite** bit of `x`'s current bit; if that child doesn't exist, move to the same-bit child. Accumulate the resulting XOR value bit by bit, and track the maximum across all numbers.

```csharp
using System;

public class BitTrieNode
{
    public BitTrieNode[] Children = new BitTrieNode[2];
}

public class MaxXorTrie
{
    private const int BITS = 32;
    private BitTrieNode root = new BitTrieNode();

    private void Insert(int num)
    {
        BitTrieNode current = root;
        for (int i = BITS - 1; i >= 0; i--)
        {
            int bit = (num >> i) & 1;
            if (current.Children[bit] == null)
            {
                current.Children[bit] = new BitTrieNode();
            }
            current = current.Children[bit];
        }
    }

    private int MaxXorWith(int num)
    {
        BitTrieNode current = root;
        int result = 0;

        for (int i = BITS - 1; i >= 0; i--)
        {
            int bit = (num >> i) & 1;
            int desiredBit = 1 - bit; // opposite bit maximizes XOR at this position

            if (current.Children[desiredBit] != null)
            {
                result |= (1 << i);
                current = current.Children[desiredBit];
            }
            else
            {
                // opposite bit unavailable, forced to take the same bit
                current = current.Children[bit];
            }
        }

        return result;
    }

    public int FindMaximumXOR(int[] nums)
    {
        root = new BitTrieNode();
        foreach (int num in nums)
        {
            Insert(num);
        }

        int maxXor = 0;
        foreach (int num in nums)
        {
            maxXor = Math.Max(maxXor, MaxXorWith(num));
        }

        return maxXor;
    }
}
```

Time Complexity: `O(n * 32)`, i.e., `O(n)` — inserting all `n` numbers takes `O(n * 32)`, and querying the maximum XOR for all `n` numbers takes another `O(n * 32)`; the constant `32` is fixed regardless of `n`.
Space Complexity: `O(n * 32)` in the worst case — each of the `n` numbers can create up to 32 new trie nodes if it shares no prefix with previously inserted numbers, though in practice numbers with common high-order bits share nodes.

**Explanation:**
Dry run on `nums = [3, 10, 5, 25, 2, 8]`. In 5-bit binary (enough to represent values up to 31):
- `3  = 00011`
- `10 = 01010`
- `5  = 00101`
- `25 = 11001`
- `2  = 00010`
- `8  = 01000`

All six numbers are inserted into the bit trie first, so the trie contains all these 5-bit paths from the root.

Now consider querying for `num = 5 = 00101`, walking bit-by-bit from the most significant bit (position 4 down to 0):
- **Bit 4** (value 16): `5`'s bit is `0`. Desired opposite bit is `1`. Does the trie have a `1` child at the root for bit 4? Yes — `25 = 11001` starts with `1`. Go to the `1` branch. `result` so far accumulates `16`.
- **Bit 3** (value 8): Continuing on the branch that has `1` at bit 4 (i.e., numbers starting `1...`, which is only `25 = 11001`), `5`'s bit 3 is `0`. Desired opposite is `1`. Since we're now confined to the sub-trie containing only `25`'s remaining bits `1001`, the next bit available is `1`. Take it. `result` accumulates `8` more, total `24`.
- **Bit 2** (value 4): `5`'s bit 2 is `1`. Desired opposite is `0`. The remaining path (from `25`'s bits `001`) has next bit `0`. Take it. `result` accumulates `4` more, total `28`.
- **Bit 1** (value 2): `5`'s bit 1 is `0`. Desired opposite is `1`. Remaining path (`25`'s bits `01`) has next bit `0`, not `1`, so opposite is unavailable — forced to take same bit `0`. No addition to `result`. Total remains `28`.
- **Bit 0** (value 1): `5`'s bit 0 is `1`. Desired opposite is `0`. Remaining path (`25`'s last bit `1`) has next bit `1`, not `0` — forced to take same bit `1`. No addition. Total remains `28`.

Final `result = 28`, matching `5 XOR 25 = 00101 XOR 11001 = 11100 = 28`. Repeating this walk for every number in the array and tracking the running maximum yields the overall answer of `28`, achieved by the pair `(5, 25)`.

---

## 3. Maximum XOR With an Element From Array (Queries)

**Problem Statement:**
Given an array `nums` and a list of queries, where each query is a pair `(x, maxLimit)`, for each query find the **maximum XOR** of `x` with any element `y` in `nums` such that `y <= maxLimit`. If no such element exists (i.e., no element of `nums` is `<= maxLimit`), the answer for that query is `-1`.

**Example:**
- Input: `nums = [0, 1, 2, 3, 4]`, `queries = [(3, 1), (1, 3), (5, 6)]`
- Output: `[3, 3, 7]`
- Explanation:
  - Query `(3, 1)`: eligible elements (`<= 1`) are `{0, 1}`. Max XOR of `3` with these is `3 XOR 0 = 3`.
  - Query `(1, 3)`: eligible elements are `{0, 1, 2, 3}`. Max XOR of `1` with these is `1 XOR 2 = 3`.
  - Query `(5, 6)`: eligible elements are `{0, 1, 2, 3, 4}`. Max XOR of `5` with these is `5 XOR 2 = 7`.

**Brute Force Approach:**
For each query `(x, maxLimit)`, scan the entire array and check every element `y <= maxLimit`, computing `x XOR y` and tracking the maximum. If no eligible element is found, output `-1`.

```csharp
using System;
using System.Collections.Generic;

public class MaxXorQueriesBruteForce
{
    public int[] MaximizeXor(int[] nums, int[][] queries)
    {
        int q = queries.Length;
        int[] answers = new int[q];

        for (int i = 0; i < q; i++)
        {
            int x = queries[i][0];
            int maxLimit = queries[i][1];
            int best = -1;

            foreach (int y in nums)
            {
                if (y <= maxLimit)
                {
                    best = Math.Max(best, x ^ y);
                }
            }

            answers[i] = best;
        }

        return answers;
    }
}
```

Time Complexity: `O(n * q)` — for each of the `q` queries, we scan all `n` array elements.
Space Complexity: `O(q)` for the answers array (excluding input storage).

**Optimized Trie-Based Approach:**
Sort both `nums` and the `queries` (keeping track of original query indices) by their limiting value in increasing order — `nums` by value, `queries` by `maxLimit`. Then process queries in increasing order of `maxLimit`, maintaining a pointer into the sorted `nums`. Before answering a query, insert into the bit trie every array element that is `<= maxLimit` and hasn't been inserted yet (advancing the pointer). This ensures the trie only ever contains elements eligible for the current (and all previous, smaller-limit) queries. Then run the same greedy bit-trie walk as in Problem 2 to find the max XOR with `x`. If the trie is still empty when a query is processed, the answer is `-1`.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;

public class BitTrieNodeQ
{
    public BitTrieNodeQ[] Children = new BitTrieNodeQ[2];
}

public class MaxXorQueriesTrie
{
    private const int BITS = 32;
    private BitTrieNodeQ root = new BitTrieNodeQ();

    private void Insert(int num)
    {
        BitTrieNodeQ current = root;
        for (int i = BITS - 1; i >= 0; i--)
        {
            int bit = (num >> i) & 1;
            if (current.Children[bit] == null)
            {
                current.Children[bit] = new BitTrieNodeQ();
            }
            current = current.Children[bit];
        }
    }

    private int MaxXorWith(int x)
    {
        BitTrieNodeQ current = root;
        int result = 0;

        for (int i = BITS - 1; i >= 0; i--)
        {
            int bit = (x >> i) & 1;
            int desiredBit = 1 - bit;

            if (current.Children[desiredBit] != null)
            {
                result |= (1 << i);
                current = current.Children[desiredBit];
            }
            else
            {
                current = current.Children[bit];
            }
        }

        return result;
    }

    public int[] MaximizeXor(int[] nums, int[][] queries)
    {
        root = new BitTrieNodeQ();

        Array.Sort(nums);

        int q = queries.Length;
        // Pair each query with its original index so we can output in the original order
        int[] queryOrder = Enumerable.Range(0, q).ToArray();
        Array.Sort(queryOrder, (a, b) => queries[a][1].CompareTo(queries[b][1]));

        int[] answers = new int[q];
        int pointer = 0;
        bool anyInserted = false;

        foreach (int qi in queryOrder)
        {
            int x = queries[qi][0];
            int maxLimit = queries[qi][1];

            // Insert all eligible elements (<= maxLimit) that haven't been inserted yet
            while (pointer < nums.Length && nums[pointer] <= maxLimit)
            {
                Insert(nums[pointer]);
                anyInserted = true;
                pointer++;
            }

            if (!anyInserted)
            {
                answers[qi] = -1;
            }
            else
            {
                answers[qi] = MaxXorWith(x);
            }
        }

        return answers;
    }
}
```

Time Complexity: `O((n + q) log(n + q) + (n + q) * 32)` — sorting `nums` takes `O(n log n)` and sorting queries takes `O(q log q)` (both bounded by `O((n+q) log(n+q))`); each array element is inserted into the trie once (`O(n * 32)` total), and each query performs one greedy walk (`O(q * 32)` total).
Space Complexity: `O(n * 32)` for the trie in the worst case (up to 32 nodes per inserted number), plus `O(q)` for the query-order array and answers array, plus `O(n)`/`O(q)` for the sorted copies.

**Explanation:**
The reason Problem 3 **cannot** simply build one bit trie upfront (inserting all of `nums` before any query, as in Problem 2) is the `maxLimit` constraint: each query is only allowed to XOR against elements `<= maxLimit`, and different queries have different limits. If we inserted every array element into the trie before answering any query, a query with a small `maxLimit` could incorrectly match against a large, ineligible element during the greedy walk, since the trie has no notion of "this node came from a value greater than my limit." Removing elements from a trie after a query (to "undo" ineligible insertions) is awkward and not naturally supported by simple trie deletion.

The clean fix is the **offline sort-by-maxLimit technique**: since eligibility only ever *grows* as `maxLimit` increases (an element eligible for a smaller limit remains eligible for every larger limit), we sort queries by `maxLimit` ascending and process them in that order, incrementally inserting newly-eligible array elements (also processed in sorted order via a pointer) right before each query is answered. This guarantees that at the moment we answer a query, the trie contains **exactly** the set of elements `<= maxLimit` — no more, no less — without ever needing to delete anything. This is the same "sort + sweep with a pointer" idea used in other offline query techniques (e.g., offline range/Mo's-algorithm-style problems), adapted here to keep the greedy XOR trie always in a valid state for the current query's constraint.

Dry run on `nums = [0, 1, 2, 3, 4]`, `queries = [(3, 1), (1, 3), (5, 6)]`:
- Sorted `nums`: `[0, 1, 2, 3, 4]` (already sorted).
- Sorted queries by `maxLimit`: `(3, 1)` (limit 1), then `(1, 3)` (limit 3), then `(5, 6)` (limit 6).
- Process `(3, 1)`: insert elements `<= 1` → insert `0`, insert `1`. Pointer now at index 2. Trie contains `{0, 1}`. Greedy walk for `x=3`: best match is `0`, giving `3 XOR 0 = 3`. Answer for this query = `3`.
- Process `(1, 3)`: insert elements `<= 3` not yet inserted → insert `2`, insert `3`. Pointer now at index 4. Trie contains `{0, 1, 2, 3}`. Greedy walk for `x=1`: best match is `2`, giving `1 XOR 2 = 3`. Answer = `3`.
- Process `(5, 6)`: insert elements `<= 6` not yet inserted → insert `4`. Pointer now at index 5 (end). Trie contains `{0, 1, 2, 3, 4}`. Greedy walk for `x=5`: best match is `2`, giving `5 XOR 2 = 7`. Answer = `7`.

Mapping answers back to original query order gives `[3, 3, 7]`, matching the expected output.
