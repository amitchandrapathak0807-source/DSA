# Binary Search Trees — Medium and Hard Problems

Node definition used throughout this document:

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

## 1. Check Whether a Binary Tree is a Valid BST

**Problem Statement:** Given the root of a binary tree, determine if it is a valid Binary Search Tree (BST). A valid BST is defined as follows: the left subtree of a node contains only nodes with values strictly less than the node's value, the right subtree contains only nodes with values strictly greater than the node's value, and both the left and right subtrees must also be valid BSTs (i.e., every node in the left subtree must be less than the current node, not just its immediate left child, and similarly for the right subtree).

**Example:**
- Input: `root = [5, 1, 4, null, null, 3, 6]` (5 has left child 1, right child 4; 4 has left child 3, right child 6)
- Output: `false` — because node 4's left subtree contains 3 (valid so far), but node 4 is the right child of 5, so everything in 4's subtree must be `> 5`. Node 3 is `< 5`, so the tree is invalid.

**Brute Force Approach:** Perform an inorder traversal of the tree and store the visited values in a `List<int>`. A BST's inorder traversal must be strictly increasing. After collecting the list, iterate through it and check that every element is strictly greater than the previous one. If any pair violates this, the tree is not a valid BST.

```csharp
public class SolutionBrute {
    public bool IsValidBST(TreeNode root) {
        List<int> inorder = new List<int>();
        InorderTraversal(root, inorder);

        for (int i = 1; i < inorder.Count; i++) {
            if (inorder[i] <= inorder[i - 1]) {
                return false;
            }
        }
        return true;
    }

    private void InorderTraversal(TreeNode node, List<int> inorder) {
        if (node == null) return;
        InorderTraversal(node.left, inorder);
        inorder.Add(node.val);
        InorderTraversal(node.right, inorder);
    }
}
```

Time Complexity: `O(n)` to traverse all nodes plus `O(n)` to scan the list, overall `O(n)`.
Space Complexity: `O(n)` for the list storing all node values, plus `O(h)` recursion stack (h = height).

**Optimized Approach:** Instead of materializing the entire inorder sequence, perform a single recursive DFS that carries a valid `(min, max)` range for each node. The root can be any value, so its range is `(-infinity, +infinity)`. When recursing into the left child, tighten the upper bound to the current node's value; when recursing into the right child, tighten the lower bound to the current node's value. A node is valid only if its value lies strictly within its inherited `(min, max)` range. This propagates ancestor constraints down through every level, so a node several levels deep is still checked against all of its ancestors, not just its immediate parent.

```csharp
public class SolutionOptimal {
    public bool IsValidBST(TreeNode root) {
        return Validate(root, long.MinValue, long.MaxValue);
    }

    private bool Validate(TreeNode node, long min, long max) {
        if (node == null) return true;

        if (node.val <= min || node.val >= max) {
            return false;
        }

        return Validate(node.left, min, node.val) &&
               Validate(node.right, node.val, max);
    }
}
```

Time Complexity: `O(n)` — each node is visited exactly once.
Space Complexity: `O(h)` for the recursion stack (no auxiliary list is stored), where `h` is the height of the tree; `O(log n)` for a balanced tree and `O(n)` in the worst-case skewed tree.

**Explanation:** Consider the earlier example: `root = [5, 1, 4, null, null, 3, 6]`, where 5 has left child 1 and right child 4, and 4 has left child 3 and right child 6.

A naive "compare only with immediate children" check would look at node 4 and its children 3 and 6: `3 < 4` and `6 > 4`, so node 4 alone looks fine, and the check would wrongly report the tree as valid.

The min/max range-bound approach catches this correctly:
- Start: `Validate(5, -inf, +inf)`. `5` is within range, valid. Recurse left with `(-inf, 5)`, recurse right with `(5, +inf)`.
- Left: `Validate(1, -inf, 5)`. `1` is within `(-inf, 5)`, valid. Both children are null, return true.
- Right: `Validate(4, 5, +inf)`. Here the range is `(5, +inf)` because node 4 is the right child of 5, meaning everything under 4 must be strictly greater than 5. But `4` itself is not greater than `5` (`4 <= 5`), so this call immediately returns `false`.

The traversal never even needs to reach node 3 to detect the violation — node 4 itself already fails because it inherited the constraint "must be `> 5`" from its ancestor two levels up (well, one level up here, but the same mechanism scales to arbitrarily deep violations). This demonstrates why range propagation, not just parent-child comparison, is required: a node deep in a subtree can be locally consistent with its immediate parent yet violate a constraint imposed by a grandparent or higher ancestor, and only carrying the range down through recursion catches that.

---

## 2. Lowest Common Ancestor (LCA) in a BST

**Problem Statement:** Given the root of a BST and two nodes `p` and `q` that exist in the tree, find their lowest common ancestor (LCA). The LCA is defined as the lowest (deepest) node in the tree that has both `p` and `q` as descendants (a node can be a descendant of itself).

**Example:**
- Input: BST `root = [6, 2, 8, 0, 4, 7, 9, null, null, 3, 5]`, `p = 2`, `q = 8`
- Output: `6` — node 6 is the lowest node that has both 2 and 8 in its subtrees (2 is in the left subtree, 8 is in the right subtree).

**Brute Force Approach:** Treat the BST as a generic binary tree (ignore the BST ordering property). Find the root-to-node path for `p` using a recursive DFS, and separately find the root-to-node path for `q`. Store both paths as lists of nodes from root to target. Then walk both path lists simultaneously from the start; the last node at which both paths still agree is the LCA.

```csharp
public class SolutionBrute {
    public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        List<TreeNode> pathToP = new List<TreeNode>();
        List<TreeNode> pathToQ = new List<TreeNode>();

        FindPath(root, p, pathToP);
        FindPath(root, q, pathToQ);

        TreeNode lca = null;
        int i = 0;
        while (i < pathToP.Count && i < pathToQ.Count && pathToP[i] == pathToQ[i]) {
            lca = pathToP[i];
            i++;
        }
        return lca;
    }

    private bool FindPath(TreeNode node, TreeNode target, List<TreeNode> path) {
        if (node == null) return false;

        path.Add(node);
        if (node == target) return true;

        if (FindPath(node.left, target, path) || FindPath(node.right, target, path)) {
            return true;
        }

        path.RemoveAt(path.Count - 1);
        return false;
    }
}
```

Time Complexity: `O(n)` in the worst case to find each path (n = number of nodes), so `O(n)` overall.
Space Complexity: `O(n)` for the two path lists plus `O(h)` recursion stack.

**Optimized Approach:** Exploit the BST ordering property directly instead of searching for paths. Starting at the root, compare both `p.val` and `q.val` against the current node's value. If both are smaller, the LCA must lie in the left subtree, so move left. If both are larger, the LCA must lie in the right subtree, so move right. If they diverge (one is smaller or equal and the other is larger or equal, i.e., the current node's value lies between them, or the current node equals one of them), the current node is the split point and therefore the LCA. This can be done iteratively without any extra path storage.

```csharp
public class SolutionOptimal {
    public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        TreeNode current = root;

        while (current != null) {
            if (p.val < current.val && q.val < current.val) {
                current = current.left;
            } else if (p.val > current.val && q.val > current.val) {
                current = current.right;
            } else {
                // p and q diverge here (or current equals p or q) -> split point found
                return current;
            }
        }

        return null;
    }
}
```

Time Complexity: `O(h)` where `h` is the height of the tree — `O(log n)` for a balanced BST, `O(n)` worst case for a skewed BST.
Space Complexity: `O(1)` extra space since the walk is iterative and uses no recursion stack or auxiliary storage.

**Explanation:** Using the example BST `[6, 2, 8, 0, 4, 7, 9, null, null, 3, 5]` with `p = 2`, `q = 8`:
- `current = 6`. Is `2 < 6` and `8 < 6`? No (`8` is not `< 6`). Is `2 > 6` and `8 > 6`? No (`2` is not `> 6`). So `p` and `q` diverge at `6` — return `6` as the LCA. This matches the expected output directly in one comparison, illustrating how the BST property collapses the generic path-comparison approach into a simple directional walk.

---

## 3. Construct a BST from a Preorder Traversal

**Problem Statement:** Given an array of distinct integers `preorder` representing the preorder traversal of a BST, construct the BST and return its root.

**Example:**
- Input: `preorder = [8, 5, 1, 7, 10, 12]`
- Output: A BST rooted at 8, with left subtree containing `{5, 1, 7}` and right subtree containing `{10, 12}` — specifically `root=8`, `root.left=5` (`5.left=1`, `5.right=7`), `root.right=10` (`10.right=12`).

**Brute Force Approach:** Simulate BST insertion. Start with an empty tree and insert each value from `preorder`, in order, using the standard BST insert operation (compare with current node, go left if smaller, go right if larger, insert at the first null spot found). Since preorder visits the root before its subtrees, inserting in preorder order naturally reconstructs the correct BST shape.

```csharp
public class SolutionBrute {
    public TreeNode BstFromPreorder(int[] preorder) {
        TreeNode root = null;
        foreach (int val in preorder) {
            root = Insert(root, val);
        }
        return root;
    }

    private TreeNode Insert(TreeNode node, int val) {
        if (node == null) {
            return new TreeNode(val);
        }
        if (val < node.val) {
            node.left = Insert(node.left, val);
        } else {
            node.right = Insert(node.right, val);
        }
        return node;
    }
}
```

Time Complexity: `O(n^2)` worst case — each insertion can take `O(h)` and the tree can degenerate into a skewed shape (e.g., a sorted preorder input), making `h` up to `n`.
Space Complexity: `O(h)` recursion stack per insert; `O(n)` overall tree storage (not counted as "extra" space).

**Optimized Approach:** Use a bounds-based recursion with a single global index pointer into the `preorder` array (a mutable index, e.g., via a wrapped `int[]` of size 1 or a class field). Each recursive call is given an upper bound representing the maximum value that is still valid for the current subtree (inherited from an ancestor that this subtree is the left child of). As long as `preorder[index]` is less than the current bound, it belongs to the current subtree: consume it as the current node, recursively build its left subtree with the current node's value as the new upper bound, then recursively build its right subtree with the same inherited upper bound. This processes each array element exactly once.

```csharp
public class SolutionOptimal {
    private int index = 0;

    public TreeNode BstFromPreorder(int[] preorder) {
        return Build(preorder, int.MaxValue);
    }

    private TreeNode Build(int[] preorder, int bound) {
        if (index == preorder.Length || preorder[index] > bound) {
            return null;
        }

        TreeNode node = new TreeNode(preorder[index]);
        index++;
        node.left = Build(preorder, node.val);
        node.right = Build(preorder, bound);
        return node;
    }
}
```

Time Complexity: `O(n)` — each element of `preorder` is consumed exactly once via the shared `index`.
Space Complexity: `O(h)` recursion stack only, `O(1)` extra beyond that (no repeated tree traversal/insertion).

**Explanation:** For `preorder = [8, 5, 1, 7, 10, 12]`:
- `Build(bound = +inf)`: `preorder[0] = 8 <= +inf`, consume it, `index = 1`. Create node `8`.
  - `node.left = Build(bound = 8)`: `preorder[1] = 5 <= 8`, consume, `index = 2`. Create node `5`.
    - `left = Build(bound = 5)`: `preorder[2] = 1 <= 5`, consume, `index = 3`. Create node `1`.
      - `left = Build(bound = 1)`: `preorder[3] = 7 > 1`, stop, return `null`.
      - `right = Build(bound = 5)`: `preorder[3] = 7 > 5`, stop, return `null`.
      - Node `1` has no children.
    - `right = Build(bound = 8)`: `preorder[3] = 7 <= 8`, consume, `index = 4`. Create node `7`, both its children see `preorder[4] = 10 > 7`/`> 8` bounds and return null appropriately... actually bound passed to 7's children is `7` and `8` respectively, and `10 > 7` and `10 > 8`, so both are null.
    - Node `5` gets `left = 1`, `right = 7`.
  - `node.right = Build(bound = +inf)`: `preorder[4] = 10 <= +inf`, consume, `index = 5`. Create node `10`.
    - `left = Build(bound = 10)`: `preorder[5] = 12 > 10`, return null.
    - `right = Build(bound = +inf)`: `preorder[5] = 12 <= +inf`, consume, `index = 6`. Create node `12`; index now equals array length, so both its children return null immediately.
  - Final tree: `8` with `left = 5` (`5.left=1`, `5.right=7`) and `right = 10` (`10.right=12`), matching the expected output.

---

## 4. Inorder Successor and Predecessor in a BST

**Problem Statement:** Given the root of a BST and a target value `key`, find the inorder predecessor (the node with the largest value strictly less than `key`) and the inorder successor (the node with the smallest value strictly greater than `key`) of that key in the BST. `key` may or may not itself be present as a node in the tree.

**Example:**
- Input: BST `root = [8, 3, 10, 1, 6, null, 14, null, null, 4, 7, 13]`, `key = 6`
- Output: Predecessor = `4`, Successor = `7`

**Brute Force Approach:** Perform a full inorder traversal of the tree, storing all values into a `List<int>` (this yields a sorted list since it's a BST). Then linearly scan the sorted list to find the predecessor as the last value strictly less than `key`, and the successor as the first value strictly greater than `key`.

```csharp
public class SolutionBrute {
    public (int? predecessor, int? successor) FindPredecessorSuccessor(TreeNode root, int key) {
        List<int> inorder = new List<int>();
        InorderTraversal(root, inorder);

        int? predecessor = null;
        int? successor = null;

        foreach (int val in inorder) {
            if (val < key) {
                predecessor = val; // keep updating; last one before key is the answer
            } else if (val > key && successor == null) {
                successor = val; // first one after key is the answer
            }
        }

        return (predecessor, successor);
    }

    private void InorderTraversal(TreeNode node, List<int> inorder) {
        if (node == null) return;
        InorderTraversal(node.left, inorder);
        inorder.Add(node.val);
        InorderTraversal(node.right, inorder);
    }
}
```

Time Complexity: `O(n)` to build the list plus `O(n)` to scan it, overall `O(n)`.
Space Complexity: `O(n)` for the stored inorder list plus `O(h)` recursion stack.

**Optimized Approach:** Walk down from the root iteratively, using the BST ordering property to decide direction, and update predecessor/successor pointers as you go — no full list is ever materialized. If `node.val < key`, this node is a candidate predecessor (it's smaller than key, and moving right could find something even closer/larger while still `< key`), so record it and move right. If `node.val > key`, this node is a candidate successor, record it and move left. If `node.val == key`, the predecessor is the maximum of the left subtree and the successor is the minimum of the right subtree (found by walking as far left/right as possible from those subtrees).

```csharp
public class SolutionOptimal {
    public (TreeNode predecessor, TreeNode successor) FindPredecessorSuccessor(TreeNode root, int key) {
        TreeNode predecessor = null;
        TreeNode successor = null;
        TreeNode current = root;

        while (current != null) {
            if (current.val == key) {
                // predecessor = max of left subtree
                if (current.left != null) {
                    TreeNode temp = current.left;
                    while (temp.right != null) temp = temp.right;
                    predecessor = temp;
                }
                // successor = min of right subtree
                if (current.right != null) {
                    TreeNode temp = current.right;
                    while (temp.left != null) temp = temp.left;
                    successor = temp;
                }
                break;
            } else if (current.val < key) {
                predecessor = current;
                current = current.right;
            } else {
                successor = current;
                current = current.left;
            }
        }

        return (predecessor, successor);
    }
}
```

Time Complexity: `O(h)` — a single downward walk, `O(log n)` for a balanced BST, `O(n)` worst case for a skewed BST.
Space Complexity: `O(1)` extra space (purely iterative, no recursion stack, no stored list).

**Explanation:** Using `root = [8, 3, 10, 1, 6, null, 14, null, null, 4, 7, 13]` with `key = 6` (tree: 8 has left 3, right 10; 3 has left 1, right 6; 6 has left 4, right 7; 10 has right 14; 14 has left 13):
- `current = 8`: `8 > 6` → `successor = 8`, move left to `3`.
- `current = 3`: `3 < 6` → `predecessor = 3`, move right to `6`.
- `current = 6`: `6 == key`. Left subtree of `6` is node `4` (no children), so `predecessor = 4` (overwriting the earlier candidate `3`, since `4` is closer/larger while still `< 6`). Right subtree of `6` is node `7` (no children), so `successor = 7` (overwriting `8`). Break.
- Final answer: predecessor `= 4`, successor `= 7`, matching the expected output. This shows why walking down and continuously overwriting the candidate pointers works: each time we go right past a smaller node, that node becomes a tighter (larger) predecessor candidate, and symmetrically for successor when going left past larger nodes; when we land exactly on the key, the immediate subtree extremes finalize the answer.

---

## 5. Merge Two BSTs into a Sorted Sequence (or a Balanced BST)

**Problem Statement:** Given the roots of two separate BSTs, merge all their elements into a single sorted sequence (a sorted `List<int>`), and optionally build a single balanced BST from that merged, sorted sequence.

**Example:**
- Input: BST1 `= [3, 1, 5]`, BST2 `= [4, 2, 6]`
- Output: Sorted merged sequence `= [1, 2, 3, 4, 5, 6]`; a balanced BST built from this sequence could be rooted at `4` with left subtree `[1,2,3]` and right subtree `[5,6]`.

**Brute Force Approach:** Perform an inorder traversal of BST1 into `List<int> list1` and BST2 into `List<int> list2` (both come out sorted). Concatenate the two lists into one combined list, then sort the combined list (even though each half is already sorted, this brute-force version just re-sorts everything, ignoring that fact). Optionally, build a balanced BST from the final sorted list by repeatedly picking the middle element as the root.

```csharp
public class SolutionBrute {
    public List<int> MergeBSTs(TreeNode root1, TreeNode root2) {
        List<int> list1 = new List<int>();
        List<int> list2 = new List<int>();
        InorderTraversal(root1, list1);
        InorderTraversal(root2, list2);

        List<int> combined = new List<int>();
        combined.AddRange(list1);
        combined.AddRange(list2);
        combined.Sort(); // O(n log n) even though both halves were already sorted

        return combined;
    }

    private void InorderTraversal(TreeNode node, List<int> result) {
        if (node == null) return;
        InorderTraversal(node.left, result);
        result.Add(node.val);
        InorderTraversal(node.right, result);
    }
}
```

Time Complexity: `O(n log n)` dominated by the sort, where `n = n1 + n2` is the total number of nodes across both trees.
Space Complexity: `O(n)` for the lists.

**Optimized Approach:** Since each BST's inorder traversal already produces a sorted list, avoid re-sorting by using the classic merge step from merge sort: given two already-sorted lists, merge them into one sorted list in linear time using two pointers. Then, to get a balanced BST, recursively pick the middle element of the merged sorted array as the root, and recurse on the left and right halves — this guarantees a height-balanced BST.

```csharp
public class SolutionOptimal {
    public TreeNode MergeBSTs(TreeNode root1, TreeNode root2) {
        List<int> list1 = new List<int>();
        List<int> list2 = new List<int>();
        InorderTraversal(root1, list1);
        InorderTraversal(root2, list2);

        List<int> merged = MergeSortedLists(list1, list2);

        return BuildBalancedBST(merged, 0, merged.Count - 1);
    }

    private void InorderTraversal(TreeNode node, List<int> result) {
        if (node == null) return;
        InorderTraversal(node.left, result);
        result.Add(node.val);
        InorderTraversal(node.right, result);
    }

    private List<int> MergeSortedLists(List<int> a, List<int> b) {
        List<int> merged = new List<int>(a.Count + b.Count);
        int i = 0, j = 0;

        while (i < a.Count && j < b.Count) {
            if (a[i] <= b[j]) merged.Add(a[i++]);
            else merged.Add(b[j++]);
        }
        while (i < a.Count) merged.Add(a[i++]);
        while (j < b.Count) merged.Add(b[j++]);

        return merged;
    }

    private TreeNode BuildBalancedBST(List<int> sorted, int left, int right) {
        if (left > right) return null;

        int mid = left + (right - left) / 2;
        TreeNode node = new TreeNode(sorted[mid]);
        node.left = BuildBalancedBST(sorted, left, mid - 1);
        node.right = BuildBalancedBST(sorted, mid + 1, right);
        return node;
    }
}
```

Time Complexity: `O(n1 + n2)` for both inorder traversals and the linear merge, plus `O(n)` to build the balanced BST — overall `O(n)` where `n = n1 + n2`.
Space Complexity: `O(n)` for the lists (list1, list2, merged) plus `O(log n)` recursion depth for building the balanced tree; strictly better than the brute force's `O(n log n)` time.

**Explanation:** For BST1 `= [3, 1, 5]` (inorder → `[1, 3, 5]`) and BST2 `= [4, 2, 6]` (inorder → `[2, 4, 6]`):
- Two-pointer merge: `i=0 (a[0]=1), j=0 (b[0]=2)` → `1 <= 2`, take `1`, `i=1`.
- `a[1]=3, b[0]=2` → `2 <= 3`, take `2`, `j=1`.
- `a[1]=3, b[1]=4` → `3 <= 4`, take `3`, `i=2`.
- `a[2]=5, b[1]=4` → `4 <= 5`, take `4`, `j=2`.
- `a[2]=5, b[2]=6` → `5 <= 6`, take `5`, `i=3`. `i` exhausted, append remaining `b`: `6`.
- Merged sorted list: `[1, 2, 3, 4, 5, 6]`, matching the expected output, produced in linear time without any comparison-based sort.
- Building the balanced BST: `mid` of `[1,2,3,4,5,6]` (indices 0-5) is index `2` (value `3`)... using `mid = left + (right-left)/2 = 0 + 5/2 = 2`, giving root `3` — note the exact root depends on the midpoint rounding convention; either `3` or `4` is an equally valid balanced-BST root for an even-length array.

---

## 6. Two Sum in a BST — Check if There Exists a Pair with Given Sum K

**Problem Statement:** Given the root of a BST and an integer target `k`, determine whether there exist two distinct nodes in the BST whose values sum exactly to `k`.

**Example:**
- Input: BST `root = [5, 3, 6, 2, 4, null, 7]`, `k = 9`
- Output: `true` — because `2 + 7 = 9` (and also `5 + 4 = 9`).

**Brute Force Approach:** Perform an inorder traversal to collect all node values into a `List<int>`. Then use a nested loop to check every pair of distinct elements to see if any pair sums to `k`.

```csharp
public class SolutionBrute {
    public bool FindTarget(TreeNode root, int k) {
        List<int> values = new List<int>();
        InorderTraversal(root, values);

        for (int i = 0; i < values.Count; i++) {
            for (int j = i + 1; j < values.Count; j++) {
                if (values[i] + values[j] == k) {
                    return true;
                }
            }
        }
        return false;
    }

    private void InorderTraversal(TreeNode node, List<int> result) {
        if (node == null) return;
        InorderTraversal(node.left, result);
        result.Add(node.val);
        InorderTraversal(node.right, result);
    }
}
```

Time Complexity: `O(n^2)` due to the nested pair-checking loop, on top of `O(n)` for the traversal.
Space Complexity: `O(n)` for the stored values list.

**Optimized Approach:** Use two BST iterators acting as two pointers converging from the smallest and largest ends, avoiding any `HashSet` and avoiding materializing a full list. Build a "forward" (next-smallest) iterator using a stack that simulates standard inorder traversal (push all left children, pop, then push right subtree's left spine), and a "backward" (next-largest) iterator using a stack that simulates reverse-inorder traversal (push all right children, pop, then push left subtree's right spine). Initialize `left = BSTIterator.Next()` (smallest value) and `right = BSTIterator.Next()` from the reverse iterator (largest value). While `left < right`: if `left + right == k`, return true; if `left + right < k`, advance the forward iterator (`left = nextSmallest()`); otherwise, advance the backward iterator (`right = nextLargest()`). This mirrors the classic two-pointer technique on a sorted array, but implemented directly on the tree structure using `O(h)` space instead of `O(n)`.

```csharp
public class BSTIterator {
    private Stack<TreeNode> stack = new Stack<TreeNode>();
    private bool reverse; // false = forward (ascending/next-smallest), true = backward (descending/next-largest)

    public BSTIterator(TreeNode root, bool reverse) {
        this.reverse = reverse;
        PushAll(root);
    }

    private void PushAll(TreeNode node) {
        while (node != null) {
            stack.Push(node);
            node = reverse ? node.right : node.left;
        }
    }

    public bool HasNext() {
        return stack.Count > 0;
    }

    public int Next() {
        TreeNode node = stack.Pop();
        if (reverse) {
            PushAll(node.left);
        } else {
            PushAll(node.right);
        }
        return node.val;
    }
}

public class SolutionOptimal {
    public bool FindTarget(TreeNode root, int k) {
        if (root == null) return false;

        BSTIterator forward = new BSTIterator(root, false);  // yields ascending values
        BSTIterator backward = new BSTIterator(root, true);  // yields descending values

        int left = forward.Next();
        int right = backward.Next();

        while (left < right) {
            int sum = left + right;
            if (sum == k) {
                return true;
            } else if (sum < k) {
                left = forward.Next();
            } else {
                right = backward.Next();
            }
        }

        return false;
    }
}
```

Time Complexity: `O(n)` — in the worst case every node is pushed and popped from each stack exactly once across the whole two-pointer sweep.
Space Complexity: `O(h)` for each of the two stacks (so `O(h)` overall, since both are bounded by the tree height), which is significantly better than the `O(n)` a `HashSet`-based or full-list-based approach would need.

**Explanation:** Using `root = [5, 3, 6, 2, 4, null, 7]` (5 has left 3, right 6; 3 has left 2, right 4; 6 has right 7), `k = 9`:
- `forward` iterator (ascending) initial `Next()` walks the left spine from 5 → 3 → 2, pops `2`, so `left = 2`.
- `backward` iterator (descending) initial `Next()` walks the right spine from 5 → 6 → 7, pops `7`, so `right = 7`.
- Loop: `left=2 < right=7`. `sum = 2 + 7 = 9 == k` → return `true` immediately.

This confirms the expected output. To illustrate the pointer-advancing logic on a case that isn't found on the first try, suppose `k = 8`: `left=2, right=7, sum=9 > 8` → advance backward: `right = backward.Next()`. The backward iterator, having popped `7` (a leaf), now backtracks and pushes nothing new from `7.left` (null), so the next pop is whatever is next on its stack, which is `6`. Now `left=2, right=6, sum=8 == k` → return `true`. This dry run shows the two iterators behave exactly like two pointers walking inward over the conceptually sorted (ascending) and reverse-sorted (descending) sequences, narrowing the search window each step without ever building those sequences explicitly.

---

## 7. Recover a BST Where Exactly Two Nodes Are Swapped by Mistake

**Problem Statement:** The values of exactly two nodes in a BST were swapped by mistake, corrupting the BST property. Recover the tree without changing its structure (only fix the values), so that the tree becomes a valid BST again.

**Example:**
- Input: BST `root = [3, 1, 4, null, null, 2]` (3 has left 1, right 4; 4 has left 2) — here `2` and `3` are effectively swapped relative to a correct BST `[2, 1, 4, null, null, 3]`... concretely take `root = [1, 3, null, null, 2]` where node values `1` and `3` are swapped from the correct `[3, 1, null, null, 2]`.
- Output: After recovery, the tree's values become the valid BST (e.g., swapping the two misplaced values back so the inorder traversal is strictly increasing).

**Brute Force Approach:** Perform an inorder traversal, storing not the values alone but references to the actual `TreeNode` objects into a `List<TreeNode>`. Since a correct BST's inorder traversal is strictly increasing, scan this list to find every position where `list[i].val > list[i+1].val` — these mark violations. There will be either one violation (if the swapped nodes are adjacent in the inorder sequence) or two violations (if they are not adjacent). Identify the two actual misplaced nodes from these violation(s), then swap their `.val` fields.

```csharp
public class SolutionBrute {
    public void RecoverTree(TreeNode root) {
        List<TreeNode> inorderNodes = new List<TreeNode>();
        InorderTraversal(root, inorderNodes);

        TreeNode first = null, second = null;

        for (int i = 0; i < inorderNodes.Count - 1; i++) {
            if (inorderNodes[i].val > inorderNodes[i + 1].val) {
                if (first == null) {
                    // first violation: the earlier node is definitely misplaced
                    first = inorderNodes[i];
                }
                // on the (possibly same) violation, the later node is the other candidate
                second = inorderNodes[i + 1];
            }
        }

        if (first != null && second != null) {
            int temp = first.val;
            first.val = second.val;
            second.val = temp;
        }
    }

    private void InorderTraversal(TreeNode node, List<TreeNode> result) {
        if (node == null) return;
        InorderTraversal(node.left, result);
        result.Add(node);
        InorderTraversal(node.right, result);
    }
}
```

Time Complexity: `O(n)` for the traversal plus `O(n)` for the scan, overall `O(n)`.
Space Complexity: `O(n)` for storing all node references in the list, plus `O(h)` recursion stack.

**Optimized Approach:** Use Morris inorder traversal to visit nodes in sorted order using `O(1)` extra space (no stack, no recursion, no list) by temporarily threading the tree: for a node with a left child, find its inorder predecessor (rightmost node in the left subtree) and make that predecessor's right pointer point back to the current node (a "thread"), then move left; when a threaded link is encountered on return, remove the thread and process the current node before moving right. While threading, track the previous visited node and compare its value with the current node's value, exactly as in the brute-force scan, but without ever storing a full list — just three pointers: `first`, `middle` (used only for adjacent-swap edge cases... commonly two pointers `first`/`second` plus `prev` suffice), and `prev`.

```csharp
public class SolutionOptimal {
    public void RecoverTree(TreeNode root) {
        TreeNode first = null;
        TreeNode second = null;
        TreeNode prev = null;
        TreeNode current = root;

        while (current != null) {
            if (current.left == null) {
                // "visit" current in inorder order
                if (prev != null && prev.val > current.val) {
                    if (first == null) {
                        first = prev;
                    }
                    second = current;
                }
                prev = current;
                current = current.right;
            } else {
                // find inorder predecessor of current
                TreeNode predecessor = current.left;
                while (predecessor.right != null && predecessor.right != current) {
                    predecessor = predecessor.right;
                }

                if (predecessor.right == null) {
                    // create the thread and move left
                    predecessor.right = current;
                    current = current.left;
                } else {
                    // thread already exists -> remove it, "visit" current, move right
                    predecessor.right = null;

                    if (prev != null && prev.val > current.val) {
                        if (first == null) {
                            first = prev;
                        }
                        second = current;
                    }
                    prev = current;
                    current = current.right;
                }
            }
        }

        if (first != null && second != null) {
            int temp = first.val;
            first.val = second.val;
            second.val = temp;
        }
    }
}
```

Time Complexity: `O(n)` — Morris traversal visits each edge at most twice (once to create the thread, once to remove it), so total work is linear.
Space Complexity: `O(1)` extra space — no recursion stack, no explicit stack, no list; only a constant number of pointers (`first`, `second`, `prev`, `current`, `predecessor`) are used, and the tree's own null pointers are temporarily repurposed as threads and always restored.

**Explanation:** Take a small BST that should be `[1, 2, 3]` in inorder order but has two non-adjacent... actually let's dry run both the adjacent and non-adjacent cases.

*Adjacent-violation example:* Correct inorder should be `1, 2, 3, 4, 5`, but nodes with values `2` and `3` are swapped, giving inorder sequence `1, 3, 2, 4, 5`.
- `prev=null, current=1`: visit `1`. No `prev` yet. `prev = 1`.
- `current=3`: visit `3`. `prev.val=1 <= 3`, no violation. `prev = 3`.
- `current=2`: visit `2`. `prev.val=3 > 2` → violation found. Since `first == null`, set `first = 3` (the node with value 3, i.e., `prev`). Set `second = 2` (i.e., `current`). `prev = 2`.
- `current=4`: visit `4`. `prev.val=2 <= 4`, no violation. `prev = 4`.
- `current=5`: visit `5`. No violation. `prev = 5`.
- End of traversal. Only one violation was recorded: `first` (node valued 3) and `second` (node valued 2) — exactly the adjacent swapped pair. Swap their values: node that held `3` now holds `2`, and node that held `2` now holds `3`. Inorder becomes `1, 2, 3, 4, 5` — correct. This is the adjacent-violation case: exactly one `prev.val > current.val` event occurs, and `first`/`second` are simply the two nodes involved in that single event.

*Non-adjacent-violation example:* Correct inorder should be `1, 2, 3, 4, 5`, but nodes with values `1` and `4` are swapped, giving inorder sequence `4, 2, 3, 1, 5`.
- `current=4`: visit `4`. No `prev` yet. `prev = 4`.
- `current=2`: visit `2`. `prev.val=4 > 2` → first violation. `first == null`, so set `first = 4` (node holding value 4, i.e., `prev`). Set `second = 2` (i.e., `current`). `prev = 2`.
- `current=3`: visit `3`. `prev.val=2 <= 3`, no violation. `prev = 3`.
- `current=1`: visit `1`. `prev.val=3 > 1` → second violation. `first` is already set (not null), so we do NOT overwrite `first`; we only update `second = 1` (i.e., `current`).
- `current=5`: visit `5`. `prev.val=1 <= 5`, no violation. `prev = 5`.
- End of traversal. `first` = node holding value `4` (captured from the *first* violation's `prev`), `second` = node holding value `1` (captured from the *second* violation's `current`). Swap their values: node that held `4` now holds `1`, node that held `1` now holds `4`. Inorder becomes `1, 2, 3, 4, 5` — correct.

This dry run shows the key logic distinction: on the first violation, both `first` (= `prev`) and `second` (= `current`) are tentatively set; on any subsequent violation, only `second` is updated (to the later violation's `current`), while `first` is left untouched from the first violation. This correctly handles both the single-violation (adjacent swap) case and the two-violation (non-adjacent swap) case with the same unified logic, and it works identically whether the traversal is done via an explicit stack or, as in the optimized solution, via Morris threading with `O(1)` space.

---

## 8. Find the Largest BST Subtree Size in a Binary Tree

**Problem Statement:** Given the root of a binary tree (not necessarily a BST), find the size (number of nodes) of the largest subtree that is a valid BST.

**Example:**
- Input: `root = [10, 5, 15, 1, 8, null, 7]` (10 has left 5, right 15; 5 has left 1, right 8; 15 has right 7)
- Output: `3` — the subtree rooted at `5` (containing `5, 1, 8`) is a valid BST of size 3; the whole tree is not a valid BST because `7` (in the right subtree of 15) violates ordering relative to 10, and 15's right child 7 also breaks 15's own BST property... concretely, the largest valid BST subtree here is `{1, 5, 8}` rooted at 5, size 3.

**Brute Force Approach:** For every single node in the tree, treat that node as the root of a candidate subtree and run the standalone "Is Valid BST" check (from Problem 1) on it. If it is valid, compute its size via a simple node count. Track the maximum size found across all nodes. This means every node is both a "root candidate" (triggering a full validity + size check on its subtree) and also gets revisited many times as part of other candidates' subtree checks.

```csharp
public class SolutionBrute {
    public int LargestBSTSubtree(TreeNode root) {
        int maxSize = 0;
        CheckEveryNode(root, ref maxSize);
        return maxSize;
    }

    private void CheckEveryNode(TreeNode node, ref int maxSize) {
        if (node == null) return;

        if (IsValidBST(node, long.MinValue, long.MaxValue)) {
            int size = CountNodes(node);
            maxSize = Math.Max(maxSize, size);
        }

        CheckEveryNode(node.left, ref maxSize);
        CheckEveryNode(node.right, ref maxSize);
    }

    private bool IsValidBST(TreeNode node, long min, long max) {
        if (node == null) return true;
        if (node.val <= min || node.val >= max) return false;
        return IsValidBST(node.left, min, node.val) && IsValidBST(node.right, node.val, max);
    }

    private int CountNodes(TreeNode node) {
        if (node == null) return 0;
        return 1 + CountNodes(node.left) + CountNodes(node.right);
    }
}
```

Time Complexity: `O(n^2)` worst case — for each of the `n` nodes, the validity check plus size count can take `O(n)` in the worst case (e.g., a tree that is entirely a valid BST, so every subtree check walks the whole remaining structure).
Space Complexity: `O(h)` recursion stack for the nested calls (no extra storage beyond the call stack).

**Optimized Approach:** Perform a single bottom-up post-order DFS that, for each node, returns a combined tuple/result `(isBST, min, max, size)` describing its subtree: whether the subtree rooted here is a valid BST, the minimum and maximum values in that subtree (needed so the parent can check ordering against it), and the size of the largest valid BST found so far anywhere in this subtree (which could be smaller than the whole subtree if the subtree itself isn't a valid BST but contains one). At each node, first recurse into left and right to get their tuples. If both children report `isBST == true`, and the current node's value is strictly greater than the left subtree's max and strictly less than the right subtree's min, then the subtree rooted at the current node is itself a valid BST of size `leftSize + rightSize + 1`; propagate this size upward along with updated min/max. Otherwise, the current node's subtree is not a valid BST as a whole, so propagate `isBST = false` upward and let the size be the max of whatever valid BST sizes were already found in the left and right subtrees (so the answer is never lost even though the parent can no longer participate).

```csharp
public class SolutionOptimal {
    private class NodeInfo {
        public bool IsBST;
        public int Min;
        public int Max;
        public int Size; // size of the largest BST subtree found within this node's subtree

        public NodeInfo(bool isBST, int min, int max, int size) {
            IsBST = isBST;
            Min = min;
            Max = max;
            Size = size;
        }
    }

    private int largest = 0;

    public int LargestBSTSubtree(TreeNode root) {
        Dfs(root);
        return largest;
    }

    private NodeInfo Dfs(TreeNode node) {
        if (node == null) {
            // neutral element: valid BST of size 0, bounds set so any real parent value satisfies min/max checks
            return new NodeInfo(true, int.MaxValue, int.MinValue, 0);
        }

        NodeInfo left = Dfs(node.left);
        NodeInfo right = Dfs(node.right);

        if (left.IsBST && right.IsBST && node.val > left.Max && node.val < right.Min) {
            int size = left.Size + right.Size + 1;
            largest = Math.Max(largest, size);

            int min = Math.Min(node.val, left.Min);
            int max = Math.Max(node.val, right.Max);
            return new NodeInfo(true, min, max, size);
        }

        // not a valid BST rooted here; propagate false and keep the best size already found below
        return new NodeInfo(false, int.MinValue, int.MaxValue, Math.Max(left.Size, right.Size));
    }
}
```

Time Complexity: `O(n)` — each node is visited exactly once in the post-order DFS, and all work per node (comparisons, tuple construction) is `O(1)`.
Space Complexity: `O(h)` recursion stack; `O(1)` extra space per call beyond that (the `NodeInfo` objects are small and not accumulated into any list).

**Explanation:** Using `root = [10, 5, 15, 1, 8, null, 7]` (10 has left 5, right 15; 5 has left 1, right 8; 15 has right 7, no left):

Bottom-up evaluation:
- `Dfs(1)`: leaf. `left = Dfs(null) = (true, +inf, -inf, 0)`, `right = Dfs(null) = (true, +inf, -inf, 0)`. Check: `left.IsBST && right.IsBST` true, `1 > left.Max(-inf)` true, `1 < right.Min(+inf)` true → valid BST, `size = 0+0+1 = 1`. `largest = 1`. Returns `(true, min=1, max=1, size=1)`.
- `Dfs(8)`: leaf, similarly returns `(true, min=8, max=8, size=1)`. `largest` stays `1`.
- `Dfs(5)`: `left = Dfs(1) = (true,1,1,1)`, `right = Dfs(8) = (true,8,8,1)`. Check: both `isBST` true, `5 > left.Max(1)` true, `5 < right.Min(8)` true → valid BST, `size = 1+1+1 = 3`. `largest = 3`. Returns `(true, min=1, max=8, size=3)`.
- `Dfs(7)`: leaf, returns `(true, min=7, max=7, size=1)`. `largest` stays `3`.
- `Dfs(15)`: `left = Dfs(null) = (true, +inf, -inf, 0)`, `right = Dfs(7) = (true,7,7,1)`. Check: both `isBST` true, `15 > left.Max(-inf)` true, but `15 < right.Min(7)`? `15 < 7` is **false** → condition fails. So node 15's subtree is NOT a valid BST as a whole (makes sense: 7 is in 15's right subtree but `7 < 15`, violating BST ordering). Propagate `isBST = false`, and `size = Math.Max(left.Size(0), right.Size(1)) = 1`. `largest` stays `3`. Returns `(false, min=-inf, max=+inf, size=1)`.
- `Dfs(10)`: `left = Dfs(5) = (true,1,8,3)`, `right = Dfs(15) = (false, ..., size=1)`. Check: `right.IsBST` is `false`, so the overall condition fails immediately (short-circuits). Node 10's subtree is not a valid BST. Propagate `isBST=false`, `size = Math.Max(left.Size(3), right.Size(1)) = 3`. `largest` stays `3`.

Final answer: `largest = 3`, matching the expected output — the largest valid BST subtree is the one rooted at node `5` (containing `1, 5, 8`), and the algorithm correctly "loses" node 10 and node 15 from consideration (since their subtrees aren't valid BSTs) while still preserving and propagating the best size (`3`) found deeper in the tree, all in a single bottom-up pass without ever re-validating any node more than once.
