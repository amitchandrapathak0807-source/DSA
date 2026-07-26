# Tries — Trie Basics

## Concept: Trie (Prefix Tree)

A **Trie** (pronounced "try", from re**trie**val) is a tree-based data structure used to store a dynamic set of strings, where keys are usually lowercase strings. Unlike a binary search tree, a Trie does not store the full string in a single node. Instead:

- Each **node** represents a single character position.
- Each **edge** from a node to its child represents a character.
- A **path from the root** down to some node spells out a prefix; if that node is marked as an "end of word", the path spells out a complete word that was inserted.
- Multiple words that share a common prefix **share the same chain of nodes** for that prefix, and only diverge at the point where their characters differ.

For example, inserting `"apple"` and `"app"` results in a single shared path `a -> p -> p`, with the node at `p` (index 2, i.e. after `"app"`) marked as end-of-word, and the path continuing `l -> e` for `"apple"`, whose final `e` node is also marked as end-of-word.

### Why Tries are efficient for prefix operations

For a word of length `L`:
- **Insert**, **Search**, and **StartsWith** all take **O(L)** time, because each operation simply walks down at most `L` nodes, one per character.
- Crucially, this cost is **independent of how many words are already stored** in the Trie (`N`). Compare this to searching a list of `N` strings, which costs `O(N * L)` in the worst case. A Trie effectively turns "does any word have this prefix" into a direct tree walk rather than a scan.
- Because words with common prefixes share nodes, prefix-based queries (like "list all words starting with `pre`" or "count words starting with `pre`") become cheap: you just walk to the node representing the prefix and then explore/aggregate from there, without re-checking unrelated words.

### Typical node structure

A standard Trie node for lowercase English letters (`a`–`z`) typically contains:

- `children`: an array of size **26**, where `children[i]` points to the child node for character `('a' + i)`, or `null`/`None` if no word uses that character at this position. (A `Dictionary<char, TrieNode>` can be used instead for a sparser or more general alphabet.)
- `isEndOfWord`: a boolean flag marking whether some inserted word ends exactly at this node.

Enhanced versions (used in "Trie II") add small integer counters at each node:

- `countEndsHere`: how many times a word ending exactly at this node was inserted (handles duplicate insertions).
- `countPrefixesHere`: how many inserted words pass through this node as a prefix (incremented once per insertion, for every node visited along the way).

These counters make operations like `CountWordsEqualTo` and `CountWordsStartingWith` O(L) instead of requiring a traversal/enumeration of the subtree.

---

## 1. Implement Trie (Insert, Search, StartsWith)

**Problem Statement:**
Implement a Trie (prefix tree) data structure that supports three operations:
- `Insert(word)`: inserts the string `word` into the Trie.
- `Search(word)`: returns `true` if `word` was previously inserted into the Trie (as a complete word), `false` otherwise.
- `StartsWith(prefix)`: returns `true` if there is at least one previously inserted word that has `prefix` as a prefix, `false` otherwise.

**Example:**
- Input: `Insert("apple"), Search("apple") -> true, Search("app") -> false, StartsWith("app") -> true, Insert("app"), Search("app") -> true`
- Output: `true, false, true, true`

**Approach:**
Each `TrieNode` holds a `children` array of size 26 (one slot per lowercase letter) and a boolean `IsEndOfWord`.

- **Insert:** Start at the root. For each character `c` in `word`, compute `index = c - 'a'`. If `children[index]` is `null`, create a new `TrieNode` there. Move into that child. After processing all characters, mark the final node's `IsEndOfWord = true`.
- **Search:** Start at the root. For each character `c` in `word`, compute `index = c - 'a'`. If `children[index]` is `null`, the word does not exist — return `false` immediately. Otherwise move into that child. After processing all characters, return the final node's `IsEndOfWord` (it must be a complete word, not just a prefix of some other word).
- **StartsWith:** Identical traversal to `Search`, but after processing all characters, simply return `true` (we only need to reach the node, we don't care if it's marked end-of-word).

```csharp
using System;

public class TrieNode
{
    public TrieNode[] Children { get; } = new TrieNode[26];
    public bool IsEndOfWord { get; set; } = false;
}

public class Trie
{
    private readonly TrieNode root;

    public Trie()
    {
        root = new TrieNode();
    }

    public void Insert(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                current.Children[index] = new TrieNode();
            }
            current = current.Children[index];
        }
        current.IsEndOfWord = true;
    }

    public bool Search(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return false;
            }
            current = current.Children[index];
        }
        return current.IsEndOfWord;
    }

    public bool StartsWith(string prefix)
    {
        TrieNode current = root;
        foreach (char c in prefix)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return false;
            }
            current = current.Children[index];
        }
        return true;
    }
}
```

Time Complexity: O(L) per operation (L = length of word). Space Complexity: O(total characters across all inserted words) worst case, O(26) per node.

**Explanation:**
Dry run inserting `"apple"` then `"app"`:

1. `Insert("apple")`: Starting at `root`, we walk `a -> p -> p -> l -> e`. None of these children exist yet, so 5 new `TrieNode`s are created: one for `a` (child of root), one for `p` (child of `a`), one for `p` (child of previous `p`), one for `l`, and one for `e`. The final node (for the second `e`... actually the last character `e`) has `IsEndOfWord` set to `true`.
2. `Insert("app")`: Walk `a -> p -> p`. The nodes for `a`, first `p`, and second `p` **already exist** (created during the previous insert), so no new nodes are created — we simply reuse/traverse them. After reaching the node for the second `p`, we set its `IsEndOfWord = true`. Note this node already had a child `l` leading further to `"apple"`; that subtree is untouched.

So after both inserts, the shared path `a -> p -> p` has two "end of word" markers hanging off it: one at the second `p` node (for `"app"`) and one further down at `e` (for `"apple"`).

3. `Search("apple")`: Walk `a -> p -> p -> l -> e`, all children exist, and the final node's `IsEndOfWord` is `true` → returns `true`.
4. `Search("app")`: Walk `a -> p -> p`, all children exist, and that node's `IsEndOfWord` is `true` (set in step 2) → returns `true`.
5. `StartsWith("appl")`: Walk `a -> p -> p -> l`, all children exist → returns `true`, regardless of whether `"appl"` itself was inserted as a full word.

---

## 2. Implement Trie II (CountWordsEqualTo, CountWordsStartingWith, Erase)

**Problem Statement:**
Implement an enhanced Trie supporting:
- `Insert(word)`: inserts `word` into the Trie (duplicates allowed — inserting the same word twice should be tracked).
- `CountWordsEqualTo(word)`: returns how many times `word` has been inserted exactly (as a complete word).
- `CountWordsStartingWith(prefix)`: returns how many inserted words (counting duplicates) have `prefix` as a prefix.
- `Erase(word)`: removes one occurrence of `word` from the Trie (assume `word` exists in the Trie when `Erase` is called).

**Example:**
- Input: `Insert("apple"), Insert("apple"), Insert("app"), CountWordsEqualTo("apple") -> 2, CountWordsStartingWith("app") -> 3, Erase("apple"), CountWordsEqualTo("apple") -> 1, CountWordsStartingWith("app") -> 2`
- Output: `2, 3, 1, 2`

**Approach:**
Each `TrieNode` now stores two counters instead of a single boolean:
- `CountEndsHere`: number of times a word has been inserted that ends exactly at this node.
- `CountPrefixesHere`: number of inserted words that pass through this node (i.e., have the prefix represented by this node).

- **Insert:** Walk down creating nodes as needed (same as before). At **every** node visited along the path (including the root's children, all the way to the final node), increment `CountPrefixesHere` by 1. At the final node only, increment `CountEndsHere` by 1.
- **CountWordsEqualTo:** Walk down the Trie following `word`'s characters. If any child is missing, return `0`. Otherwise, at the final node, return `CountEndsHere`.
- **CountWordsStartingWith:** Walk down following `prefix`'s characters. If any child is missing, return `0`. Otherwise, at the final node, return `CountPrefixesHere`.
- **Erase:** Walk down following `word`'s characters (assumed to exist). At every node visited, decrement `CountPrefixesHere` by 1. At the final node, decrement `CountEndsHere` by 1. (Optionally, nodes whose counters drop to 0 could be pruned/deleted, but it is not required for correctness of the counts.)

```csharp
using System;

public class TrieNode
{
    public TrieNode[] Children { get; } = new TrieNode[26];
    public int CountEndsHere { get; set; } = 0;
    public int CountPrefixesHere { get; set; } = 0;
}

public class TrieII
{
    private readonly TrieNode root;

    public TrieII()
    {
        root = new TrieNode();
    }

    public void Insert(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                current.Children[index] = new TrieNode();
            }
            current = current.Children[index];
            current.CountPrefixesHere++;
        }
        current.CountEndsHere++;
    }

    public int CountWordsEqualTo(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return 0;
            }
            current = current.Children[index];
        }
        return current.CountEndsHere;
    }

    public int CountWordsStartingWith(string prefix)
    {
        TrieNode current = root;
        foreach (char c in prefix)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return 0;
            }
            current = current.Children[index];
        }
        return current.CountPrefixesHere;
    }

    public void Erase(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return; // word not present; nothing to erase
            }
            current = current.Children[index];
            current.CountPrefixesHere--;
        }
        current.CountEndsHere--;
    }
}
```

Time Complexity: O(L) per operation (L = length of word). Space Complexity: O(total characters across all inserted words) worst case, O(26) per node.

**Explanation:**
Dry run inserting `"apple"`, `"apple"` again, and `"app"`:

1. `Insert("apple")` (1st time): Walk `a -> p -> p -> l -> e`, creating 5 new nodes. At each of these 5 nodes, `CountPrefixesHere` becomes `1`. At the final `e` node, `CountEndsHere` becomes `1`.
2. `Insert("apple")` (2nd time): Walk the **same** existing path `a -> p -> p -> l -> e` (no new nodes created). At each of the 5 nodes, `CountPrefixesHere` is incremented again, becoming `2`. At the final `e` node, `CountEndsHere` becomes `2`.
3. `Insert("app")`: Walk `a -> p -> p` (existing nodes, reused). Each of these 3 nodes gets `CountPrefixesHere` incremented — the `a` node goes to `3`, the first `p` to `3`, the second `p` to `3`. At the second `p` node (final for this word), `CountEndsHere` becomes `1`.

Now:
- `CountWordsEqualTo("apple")`: walk to the `e` node, return `CountEndsHere = 2` — correct, matches the two inserts.
- `CountWordsStartingWith("app")`: walk to the second `p` node, return `CountPrefixesHere`. That node was touched by both `"apple"` insertions and the `"app"` insertion, so its counter is `3` — this directly gives the count **without re-traversing or re-examining every stored word**, because the counter was pre-accumulated during each insert along the shared path.

`Erase("apple")` afterward: walk `a -> p -> p -> l -> e`, decrementing `CountPrefixesHere` at each of the 5 nodes (the shared `a, p, p` nodes drop from `3` to `2`; the unique `l, e` nodes drop from `2` to `1`), and decrement `CountEndsHere` at `e` from `2` to `1`. Now `CountWordsStartingWith("app")` correctly returns `2` (one remaining `"apple"` plus one `"app"`).

---

## 3. Longest String with All Prefixes Present (Complete String)

**Problem Statement:**
Given an array of `N` words, find the **longest word** in the array such that **every prefix** of that word (of every length from `1` up to its full length) is also present in the array as a separate word. Such a word is called a "complete string". If there are multiple longest words satisfying this, return the lexicographically smallest one. If no such word exists, return an empty string.

**Example:**
- Input: `words = ["n", "ni", "nin", "ninj", "ninja", "ninga"]`
- Output: `"ninja"`
  - Explanation: `"ninja"`'s prefixes are `"n", "ni", "nin", "ninj", "ninja"` — all present in the array. `"ninga"`'s prefixes include `"ning"`, which is **not** present, so `"ninga"` is disqualified. `"ninja"` is the longest valid complete string.

**Approach:**
Use a Trie where each node has an `IsEndOfWord` flag (as in problem 1). First, **insert every word** in the array into the Trie. Then, for **each word** in the array, walk it character by character from the root, and at **every node along the path** (not just the last one) check whether `IsEndOfWord` is `true`. If at any point along the walk a node is found where `IsEndOfWord` is `false`, that word fails the "complete string" check (some prefix of it was never inserted as its own word) — stop checking that word. If the walk completes and every single node visited had `IsEndOfWord == true`, the word qualifies as a complete string. Among all qualifying words, track the one with the maximum length, breaking ties by choosing the lexicographically smaller one (checking words in a consistent order, e.g. sorted, or explicitly comparing, makes tie-breaking simple).

```csharp
using System;
using System.Collections.Generic;

public class TrieNode
{
    public TrieNode[] Children { get; } = new TrieNode[26];
    public bool IsEndOfWord { get; set; } = false;
}

public class Trie
{
    private readonly TrieNode root;

    public Trie()
    {
        root = new TrieNode();
    }

    public void Insert(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                current.Children[index] = new TrieNode();
            }
            current = current.Children[index];
        }
        current.IsEndOfWord = true;
    }

    // Returns true if every prefix of 'word' (each length from 1..word.Length)
    // was independently inserted as a complete word in the Trie.
    public bool AllPrefixesArePresent(string word)
    {
        TrieNode current = root;
        foreach (char c in word)
        {
            int index = c - 'a';
            if (current.Children[index] == null)
            {
                return false;
            }
            current = current.Children[index];
            if (!current.IsEndOfWord)
            {
                return false;
            }
        }
        return true;
    }
}

public class Solution
{
    public string LongestCompleteString(string[] words)
    {
        Trie trie = new Trie();
        foreach (string word in words)
        {
            trie.Insert(word);
        }

        string best = "";
        foreach (string word in words)
        {
            if (trie.AllPrefixesArePresent(word))
            {
                if (word.Length > best.Length ||
                    (word.Length == best.Length && string.CompareOrdinal(word, best) < 0))
                {
                    best = word;
                }
            }
        }
        return best;
    }
}
```

Time Complexity: O(N * L) to insert all words plus O(N * L) to validate all words, where N = number of words and L = average/max word length — overall O(N * L). Space Complexity: O(N * L) worst case for Trie nodes, O(26) per node.

**Explanation:**
Using `words = ["n", "ni", "nin", "ninj", "ninja", "ninga"]`:

**Insertion phase:** All six words are inserted, building a Trie where the path `n -> i -> n -> j -> a` covers `"ninja"`, and it branches at the 4th node (`n->i->n`) into `g -> a` for `"ninga"`. Because `"n"`, `"ni"`, `"nin"`, `"ninj"`, `"ninja"` were each inserted individually, every node along the `n -> i -> n -> j -> a` path has `IsEndOfWord = true`. However `"ning"` was never inserted as a standalone word (only `"ninga"` was inserted as a whole), so the node for `n -> i -> n -> g` has `IsEndOfWord = false`.

**Validation phase — checking `"ninja"`:**
- Step to `n`: `IsEndOfWord = true` (because `"n"` was inserted). OK.
- Step to `i` (`"ni"`): `IsEndOfWord = true`. OK.
- Step to `n` (`"nin"`): `IsEndOfWord = true`. OK.
- Step to `j` (`"ninj"`): `IsEndOfWord = true`. OK.
- Step to `a` (`"ninja"`): `IsEndOfWord = true`. OK.
- All 5 steps passed → `"ninja"` is a complete string, length 5.

**Validation phase — checking `"ninga"`:**
- Steps to `n`, `i`, `n` all have `IsEndOfWord = true` (same shared prefix nodes as above). OK so far.
- Step to `g` (`"ning"`): `IsEndOfWord = false`, because `"ning"` itself was never inserted as its own array entry. The check fails immediately here — we don't even need to look at the final `a`.
- `"ninga"` is disqualified.

Comparing all qualifying words (`"n"`, `"ni"`, `"nin"`, `"ninj"`, `"ninja"` all qualify since each is a prefix of `"ninja"` and their own prefixes are trivially present), the longest is `"ninja"` with length 5, so it is returned. This shows why checking `IsEndOfWord` **at every node along the walk**, rather than only at the final node, is essential: it verifies that every intermediate prefix was independently a valid word in the list, not merely that the full word exists in the Trie.
