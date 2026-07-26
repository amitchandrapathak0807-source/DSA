# Binary Trees — Tree Properties and Views

Assumed node definition used throughout this document:

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

---

## 1. Height (Maximum Depth) of a Binary Tree

**Problem Statement:** Given the root of a binary tree, find its maximum depth, i.e., the number of nodes along the longest path from the root node down to the farthest leaf node.

**Example:**
- Input: A tree with root `1`, left child `2`, right child `3`, and `2` has a left child `4`.
- Output: `3` (path `1 -> 2 -> 4`)

**Approach:** Use simple recursion (DFS). The height of a tree rooted at `node` is `1 + max(height(left), height(right))`. Base case: a `null` node has height `0`. This is a classic bottom-up recursion where each call returns information to its parent.

```csharp
public class Solution {
    public int MaxDepth(TreeNode root) {
        if (root == null) return 0;
        int leftHeight = MaxDepth(root.left);
        int rightHeight = MaxDepth(root.right);
        return 1 + Math.Max(leftHeight, rightHeight);
    }
}
```

Time Complexity: O(n) — every node is visited exactly once.
Space Complexity: O(h) — recursion stack, where `h` is the height of the tree (O(n) worst case for a skewed tree, O(log n) for a balanced tree).

**Explanation:** Dry run on tree `1 -> (2 -> 4), 3`:
- `MaxDepth(1)` calls `MaxDepth(2)` and `MaxDepth(3)`.
- `MaxDepth(2)` calls `MaxDepth(4)` and `MaxDepth(null)`.
  - `MaxDepth(4)` calls `MaxDepth(null)` twice → returns `1 + max(0,0) = 1`.
  - `MaxDepth(null)` returns `0`.
  - So `MaxDepth(2)` returns `1 + max(1,0) = 2`.
- `MaxDepth(3)` calls `MaxDepth(null)` twice → returns `1 + max(0,0) = 1`.
- `MaxDepth(1)` returns `1 + max(2,1) = 3`.

---

## 2. Check if a Binary Tree is Height-Balanced

**Problem Statement:** A binary tree is height-balanced if, for every node, the height difference between its left and right subtrees is at most 1. Determine whether the given binary tree is height-balanced.

**Example:**
- Input: A tree with root `1`, left child `2` (which has left child `4`), right child `3`.
- Output: `true` (max height difference at any node is 1)

**Approach:** A naive solution computes height separately for every node — O(n) per node, leading to O(n^2) overall. Instead, use a single bottom-up DFS pass that simultaneously computes height and checks balance. The recursive function returns the height of the subtree; if any subtree is found unbalanced, it returns `-1` as a sentinel to short-circuit the rest of the computation.

```csharp
public class Solution {
    public bool IsBalanced(TreeNode root) {
        return DfsHeight(root) != -1;
    }

    private int DfsHeight(TreeNode node) {
        if (node == null) return 0;

        int leftHeight = DfsHeight(node.left);
        if (leftHeight == -1) return -1; // left subtree already unbalanced

        int rightHeight = DfsHeight(node.right);
        if (rightHeight == -1) return -1; // right subtree already unbalanced

        if (Math.Abs(leftHeight - rightHeight) > 1) return -1; // unbalanced here

        return 1 + Math.Max(leftHeight, rightHeight);
    }
}
```

Time Complexity: O(n) — each node visited once, single pass.
Space Complexity: O(h) — recursion stack.

**Explanation:** Dry run on tree `1 -> (2 -> (4, null)), 3`:
- `DfsHeight(1)` calls `DfsHeight(2)`.
  - `DfsHeight(2)` calls `DfsHeight(4)` → returns `1` (leaf).
  - `DfsHeight(2)` calls `DfsHeight(null)` → returns `0`.
  - `|1 - 0| = 1`, balanced here → returns `1 + max(1,0) = 2`.
- `DfsHeight(1)` calls `DfsHeight(3)` → returns `1`.
- `|2 - 1| = 1`, balanced at root → returns `1 + max(2,1) = 3`.
- Since `DfsHeight(1) = 3 != -1`, `IsBalanced` returns `true`.

If instead `4` had a child `5`, then at node `2`, `leftHeight=2, rightHeight=0`, difference `2 > 1`, so `DfsHeight(2)` returns `-1`, which propagates up through `DfsHeight(1)`, immediately giving `IsBalanced = false` without wasted recomputation.

---

## 3. Diameter of a Binary Tree

**Problem Statement:** The diameter of a binary tree is the length (number of edges) of the longest path between any two nodes in the tree. This path may or may not pass through the root.

**Example:**
- Input: A tree with root `1`, left child `2` (with children `4` and `5`), right child `3`.
- Output: `3` (path `4 -> 2 -> 1 -> 3` or `5 -> 2 -> 1 -> 3`, both have 3 edges)

**Approach:** Naively computing height at every node to find the diameter is O(n^2). Instead, compute height and diameter together in one DFS pass. Use a `ref int` (or a field) to track the running maximum diameter. At each node, the diameter passing through it equals `leftHeight + rightHeight` (in edges). Update the global max as we return heights up the recursion.

```csharp
public class Solution {
    public int DiameterOfBinaryTree(TreeNode root) {
        int diameter = 0;
        Height(root, ref diameter);
        return diameter;
    }

    private int Height(TreeNode node, ref int diameter) {
        if (node == null) return 0;

        int leftHeight = Height(node.left, ref diameter);
        int rightHeight = Height(node.right, ref diameter);

        // path through this node, measured in edges
        diameter = Math.Max(diameter, leftHeight + rightHeight);

        return 1 + Math.Max(leftHeight, rightHeight);
    }
}
```

Time Complexity: O(n) — single DFS pass.
Space Complexity: O(h) — recursion stack.

**Explanation:** Dry run on tree `1 -> (2 -> (4, 5)), 3`, `diameter = 0` initially:
- `Height(1)` calls `Height(2)`.
  - `Height(2)` calls `Height(4)` → leaf, returns `1`, `diameter = max(0, 0+0) = 0`.
  - `Height(2)` calls `Height(5)` → leaf, returns `1`, `diameter = max(0, 0+0) = 0`.
  - Back in `Height(2)`: `leftHeight=1, rightHeight=1`, `diameter = max(0, 1+1) = 2`.
  - `Height(2)` returns `1 + max(1,1) = 2`.
- `Height(1)` calls `Height(3)` → leaf, returns `1`, `diameter` unchanged (`3` is a leaf so no children update).
- Back in `Height(1)`: `leftHeight=2 (from node 2), rightHeight=1 (from node 3)`, `diameter = max(2, 2+1) = 3`.
- Final `diameter = 3`, matching path `4 -> 2 -> 1 -> 3`.

---

## 4. Maximum Path Sum in a Binary Tree

**Problem Statement:** Given a non-empty binary tree, find the maximum path sum. A path is defined as any sequence of nodes from some starting node to any node in the tree along parent-child connections, and it does not need to pass through the root. The path must contain at least one node.

**Example:**
- Input: A tree with root `-10`, left child `9`, right child `20` (which has children `15` and `7`).
- Output: `42` (path `15 -> 20 -> 7`)

**Approach:** Use post-order DFS. For each node, compute the maximum sum contribution it can offer to a path going through its parent — this is `node.val + max(0, leftContribution, rightContribution)` (we ignore negative contributions by clamping at 0). Separately, at each node, compute the best path that *bends* at this node (uses both children) as `node.val + max(0, leftContribution) + max(0, rightContribution)`, and update a global maximum with this value. The function only *returns* the single-branch contribution (since a path passed up to the parent cannot use both children), but the global max tracks the best two-branch answer seen anywhere.

```csharp
public class Solution {
    public int MaxPathSum(TreeNode root) {
        int maxSum = int.MinValue;
        MaxContribution(root, ref maxSum);
        return maxSum;
    }

    private int MaxContribution(TreeNode node, ref int maxSum) {
        if (node == null) return 0;

        // Only take positive contributions from children; ignore negative ones.
        int leftGain = Math.Max(MaxContribution(node.left, ref maxSum), 0);
        int rightGain = Math.Max(MaxContribution(node.right, ref maxSum), 0);

        // Best path that "bends" through this node, using both children.
        int pathThroughNode = node.val + leftGain + rightGain;
        maxSum = Math.Max(maxSum, pathThroughNode);

        // Return the best single-branch contribution to the parent.
        return node.val + Math.Max(leftGain, rightGain);
    }
}
```

Time Complexity: O(n) — single DFS pass.
Space Complexity: O(h) — recursion stack.

**Explanation:** Dry run on tree `-10 -> (9), (20 -> (15, 7))`, `maxSum = int.MinValue` initially:
- `MaxContribution(-10)` calls `MaxContribution(9)` → leaf, `leftGain=0, rightGain=0`, `pathThroughNode = 9`, `maxSum = max(MinValue, 9) = 9`. Returns `9 + max(0,0) = 9`. So `leftGain (of -10) = max(9, 0) = 9`.
- `MaxContribution(-10)` calls `MaxContribution(20)`.
  - `MaxContribution(15)` → leaf, `pathThroughNode = 15`, `maxSum = max(9, 15) = 15`. Returns `15`.
  - `MaxContribution(7)` → leaf, `pathThroughNode = 7`, `maxSum = max(15, 7) = 15`. Returns `7`.
  - Back in `Height(20)`: `leftGain = max(15,0) = 15`, `rightGain = max(7,0) = 7`, `pathThroughNode = 20 + 15 + 7 = 42`, `maxSum = max(15, 42) = 42`.
  - Returns `20 + max(15,7) = 35`. So `rightGain (of -10) = max(35, 0) = 35`.
- Back in `Height(-10)`: `pathThroughNode = -10 + 9 + 35 = 34`, `maxSum = max(42, 34) = 42` (unchanged).
- Final answer: `maxSum = 42`.

---

## 5. Check if Two Trees are Identical

**Problem Statement:** Given the roots of two binary trees, determine if they are structurally identical, meaning the nodes have the same value at corresponding positions.

**Example:**
- Input: Tree A = `1 -> (2, 3)`, Tree B = `1 -> (2, 3)`.
- Output: `true`

**Approach:** Simple recursive comparison. Two trees are identical if: both are `null` (true), one is `null` and the other isn't (false), or both roots have the same value and their left subtrees are identical and their right subtrees are identical.

```csharp
public class Solution {
    public bool IsSameTree(TreeNode p, TreeNode q) {
        if (p == null && q == null) return true;
        if (p == null || q == null) return false;
        if (p.val != q.val) return false;

        return IsSameTree(p.left, q.left) && IsSameTree(p.right, q.right);
    }
}
```

Time Complexity: O(min(n, m)) — traversal stops early on the first mismatch; O(n) when trees are identical (n = number of nodes).
Space Complexity: O(h) — recursion stack.

**Explanation:** Dry run on `p = 1 -> (2, 3)`, `q = 1 -> (2, 3)`:
- `IsSameTree(1, 1)`: values match, recurse on children.
- `IsSameTree(2, 2)`: both leaves, values match, recurse on `null, null` (true) twice → returns `true`.
- `IsSameTree(3, 3)`: same as above → returns `true`.
- Both subtree checks return `true`, so the overall result is `true`.

---

## 6. Zigzag (Spiral) Level Order Traversal

**Problem Statement:** Given the root of a binary tree, return the zigzag level order traversal of its nodes' values, i.e., traverse level 1 left-to-right, level 2 right-to-left, level 3 left-to-right, and so on, alternating direction at each level.

**Example:**
- Input: A tree with root `3`, left child `9`, right child `20` (which has children `15` and `7`).
- Output: `[[3], [20, 9], [15, 7]]`

**Approach:** Perform a standard level-order BFS using a `Queue<TreeNode>`. For each level, collect node values into a list. If the current level index is odd, reverse that list before adding it to the result (or use a boolean flag toggled every level).

```csharp
public class Solution {
    public IList<IList<int>> ZigzagLevelOrder(TreeNode root) {
        IList<IList<int>> result = new List<IList<int>>();
        if (root == null) return result;

        Queue<TreeNode> queue = new Queue<TreeNode>();
        queue.Enqueue(root);
        bool leftToRight = true;

        while (queue.Count > 0) {
            int levelSize = queue.Count;
            List<int> currentLevel = new List<int>(new int[levelSize]);

            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.Dequeue();

                // Place at the correct index depending on direction.
                int index = leftToRight ? i : levelSize - 1 - i;
                currentLevel[index] = node.val;

                if (node.left != null) queue.Enqueue(node.left);
                if (node.right != null) queue.Enqueue(node.right);
            }

            result.Add(currentLevel);
            leftToRight = !leftToRight;
        }

        return result;
    }
}
```

Time Complexity: O(n) — every node enqueued and dequeued once.
Space Complexity: O(n) — queue and result storage.

**Explanation:** Dry run on tree `3 -> (9), (20 -> (15, 7))`:
- Level 0: queue = `[3]`, `leftToRight = true`. Process `3` → `currentLevel = [3]`. Enqueue `9, 20`. Result: `[[3]]`. Toggle to `false`.
- Level 1: queue = `[9, 20]`, `leftToRight = false`, `levelSize = 2`. `i=0` (node `9`) → index `2-1-0=1` → `currentLevel[1]=9`. `i=1` (node `20`) → index `2-1-1=0` → `currentLevel[0]=20`. `currentLevel = [20, 9]`. Enqueue `15, 7`. Result: `[[3],[20,9]]`. Toggle to `true`.
- Level 2: queue = `[15, 7]`, `leftToRight = true`. `currentLevel = [15, 7]`. Result: `[[3],[20,9],[15,7]]`.

---

## 7. Boundary Traversal of a Binary Tree

**Problem Statement:** Given a binary tree, return the values of the nodes forming the boundary of the tree in anti-clockwise order, starting from the root: the left boundary (top-down, excluding leaves), then all leaf nodes (left-to-right), then the right boundary (bottom-up, excluding leaves).

**Example:**
- Input: A tree with root `1`, left child `2` (with children `4`, `5`), right child `3` (with children `6`, `7`), and `5` has children `8`, `9`.
- Output: `[1, 2, 4, 8, 9, 6, 7, 3]` (root+left boundary, leaves left-to-right, right boundary bottom-up)

**Approach:** Split into three parts:
1. **Left boundary** (excluding leaves): starting from `root.left`, keep going — prefer left child, else right child — adding each non-leaf node, stop when a leaf is reached.
2. **Leaf nodes**: a simple DFS (pre-order) that adds every leaf node value, left to right.
3. **Right boundary** (excluding leaves): starting from `root.right`, keep going — prefer right child, else left child — collecting non-leaf nodes into a temporary list, then reverse it before appending (since we want bottom-up order).

The root itself is added first (if it's not a leaf, or trivially if the whole tree is just the root).

```csharp
public class Solution {
    public IList<int> BoundaryTraversal(TreeNode root) {
        List<int> result = new List<int>();
        if (root == null) return result;

        if (!IsLeaf(root)) result.Add(root.val);

        AddLeftBoundary(root.left, result);
        AddLeaves(root, result);
        AddRightBoundary(root.right, result);

        return result;
    }

    private bool IsLeaf(TreeNode node) {
        return node.left == null && node.right == null;
    }

    private void AddLeftBoundary(TreeNode node, List<int> result) {
        while (node != null) {
            if (!IsLeaf(node)) result.Add(node.val);
            node = node.left != null ? node.left : node.right;
        }
    }

    private void AddLeaves(TreeNode node, List<int> result) {
        if (node == null) return;
        if (IsLeaf(node)) {
            result.Add(node.val);
            return;
        }
        AddLeaves(node.left, result);
        AddLeaves(node.right, result);
    }

    private void AddRightBoundary(TreeNode node, List<int> result) {
        List<int> temp = new List<int>();
        while (node != null) {
            if (!IsLeaf(node)) temp.Add(node.val);
            node = node.right != null ? node.right : node.left;
        }
        temp.Reverse();
        result.AddRange(temp);
    }
}
```

Time Complexity: O(n) — each node visited a constant number of times across the three passes.
Space Complexity: O(h) for recursion stack in `AddLeaves`, plus O(n) for the result/temp lists.

**Explanation:** Dry run on the example tree (`1 -> (2 -> (4, 5 -> (8,9))), (3 -> (6,7))`):
- Root `1` is not a leaf → `result = [1]`.
- `AddLeftBoundary(2, ...)`: `2` is not a leaf → add `2`. Move to `2.left = 4`. `4` is a leaf → don't add, loop ends (since `4.left` and `4.right` are both null, `node` becomes `null`). `result = [1, 2]`.
- `AddLeaves(1, ...)`: recurse down; leaves in left-to-right order are `4, 8, 9, 6, 7`. `result = [1, 2, 4, 8, 9, 6, 7]`.
- `AddRightBoundary(3, ...)`: `3` is not a leaf → `temp = [3]`. Move to `3.right = 7`. `7` is a leaf → stop (loop ends as `7` has no children). `temp = [3]`, reversed is still `[3]`. `result = [1, 2, 4, 8, 9, 6, 7, 3]`.

---

## 8. Vertical Order Traversal of a Binary Tree

**Problem Statement:** Given a binary tree, return its vertical order traversal. Group nodes by horizontal distance (column) from the root (root has horizontal distance 0, left child is `hd-1`, right child is `hd+1`). Within a column, nodes should be ordered by their level (top to bottom, i.e., row/depth); if multiple nodes share the same column and row, order them by value ascending.

**Example:**
- Input: A tree with root `3` (hd=0), left `9` (hd=-1), right `20` (hd=1), and `20` has children `15` (hd=0) and `7` (hd=2).
- Output: `[[9], [3, 15], [20], [7]]` (columns from leftmost hd=-1 to rightmost hd=2)

**Approach:** Perform a BFS using a `Queue<(TreeNode node, int hd, int level)>`, tracking horizontal distance and level (depth) for each node as we traverse. Maintain a `Dictionary<int, List<(int level, int val)>>` keyed by horizontal distance, appending `(level, val)` pairs as nodes are dequeued. After the BFS completes, for each horizontal distance key (processed in ascending order), sort its list by `(level, val)` and extract the values into the final column list.

```csharp
public class Solution {
    public IList<IList<int>> VerticalTraversal(TreeNode root) {
        IList<IList<int>> result = new List<IList<int>>();
        if (root == null) return result;

        // Map: horizontal distance -> list of (level, value)
        Dictionary<int, List<(int level, int val)>> columnMap =
            new Dictionary<int, List<(int level, int val)>>();

        Queue<(TreeNode node, int hd, int level)> queue = new Queue<(TreeNode, int, int)>();
        queue.Enqueue((root, 0, 0));

        int minHd = 0, maxHd = 0;

        while (queue.Count > 0) {
            var (node, hd, level) = queue.Dequeue();

            if (!columnMap.ContainsKey(hd)) columnMap[hd] = new List<(int, int)>();
            columnMap[hd].Add((level, node.val));

            minHd = Math.Min(minHd, hd);
            maxHd = Math.Max(maxHd, hd);

            if (node.left != null) queue.Enqueue((node.left, hd - 1, level + 1));
            if (node.right != null) queue.Enqueue((node.right, hd + 1, level + 1));
        }

        for (int hd = minHd; hd <= maxHd; hd++) {
            var column = columnMap[hd];
            column.Sort((a, b) => a.level != b.level ? a.level - b.level : a.val - b.val);
            result.Add(column.Select(c => c.val).ToList());
        }

        return result;
    }
}
```

Time Complexity: O(n log n) — BFS is O(n), but sorting each column by `(level, val)` costs O(k log k) per column, summing to O(n log n) in the worst case.
Space Complexity: O(n) — for the dictionary, queue, and result.

**Explanation:** Dry run on tree `3 -> (9), (20 -> (15, 7))`:
- Enqueue `(3, hd=0, level=0)`.
- Dequeue `(3,0,0)`: `columnMap[0] = [(0,3)]`. Enqueue `(9, hd=-1, level=1)`, `(20, hd=1, level=1)`. `minHd=0, maxHd=0` initially, updated as we go (`minHd=0, maxHd=0` from this node; will update below).
- Dequeue `(9,-1,1)`: `columnMap[-1] = [(1,9)]`. `minHd = -1`. `9` has no children.
- Dequeue `(20,1,1)`: `columnMap[1] = [(1,20)]`. `maxHd = 1`. Enqueue `(15, hd=0, level=2)`, `(7, hd=2, level=2)`.
- Dequeue `(15,0,2)`: `columnMap[0] = [(0,3), (1,15)]`.
- Dequeue `(7,2,2)`: `columnMap[2] = [(1,7)]`. `maxHd = 2`.
- Final map: `{-1: [(1,9)], 0: [(0,3),(1,15)], 1: [(1,20)], 2: [(1,7)]}`.
- Iterate `hd` from `-1` to `2`, sorting each column (already sorted here): `[[9], [3,15], [20], [7]]`.

---

## 9. Top View of a Binary Tree

**Problem Statement:** Given a binary tree, return the top view — the set of nodes visible when the tree is viewed from above. For each horizontal distance (column), only the topmost (first-encountered in level order) node should appear.

**Example:**
- Input: A tree with root `1` (hd=0), left `2` (hd=-1), right `3` (hd=1), and `2` has a right child `4` (hd=0, level 2).
- Output: `[2, 1, 3]` (node `4` is hidden behind `1` since `1` is higher at hd=0)

**Approach:** Use BFS with a `Queue<(TreeNode node, int hd)>`, tracking horizontal distance only (level doesn't matter here — we just need the first node seen per column). Maintain a `Dictionary<int, int>` mapping horizontal distance to the first node's value encountered at that distance. Since BFS processes level by level, the first time a horizontal distance is seen corresponds to the topmost node at that column — insert into the dictionary only if the key doesn't already exist. Finally, output values sorted by horizontal distance ascending.

```csharp
public class Solution {
    public IList<int> TopView(TreeNode root) {
        List<int> result = new List<int>();
        if (root == null) return result;

        Dictionary<int, int> topAtHd = new Dictionary<int, int>();
        Queue<(TreeNode node, int hd)> queue = new Queue<(TreeNode, int)>();
        queue.Enqueue((root, 0));

        int minHd = 0, maxHd = 0;

        while (queue.Count > 0) {
            var (node, hd) = queue.Dequeue();

            // Only record the first node seen at this horizontal distance.
            if (!topAtHd.ContainsKey(hd)) {
                topAtHd[hd] = node.val;
            }

            minHd = Math.Min(minHd, hd);
            maxHd = Math.Max(maxHd, hd);

            if (node.left != null) queue.Enqueue((node.left, hd - 1));
            if (node.right != null) queue.Enqueue((node.right, hd + 1));
        }

        for (int hd = minHd; hd <= maxHd; hd++) {
            result.Add(topAtHd[hd]);
        }

        return result;
    }
}
```

Time Complexity: O(n) — single BFS pass, dictionary operations are O(1) average.
Space Complexity: O(n) — dictionary and queue.

**Explanation:** Dry run on tree `1 -> (2 -> (null, 4)), 3` where `1` has hd=0, `2` has hd=-1, `3` has hd=1, `4` has hd=0 (level 2):
- Enqueue `(1, 0)`.
- Dequeue `(1,0)`: `topAtHd = {0: 1}`. Enqueue `(2,-1)`, `(3,1)`.
- Dequeue `(2,-1)`: `topAtHd = {0:1, -1:2}`. Enqueue `(4, 0)` (from `2.right`).
- Dequeue `(3,1)`: `topAtHd = {0:1, -1:2, 1:3}`.
- Dequeue `(4,0)`: hd `0` already in `topAtHd` (value `1`), so `4` is skipped — it's hidden.
- `minHd=-1, maxHd=1`. Output in order: `topAtHd[-1]=2, topAtHd[0]=1, topAtHd[1]=3` → `[2, 1, 3]`.

---

## 10. Bottom View of a Binary Tree

**Problem Statement:** Given a binary tree, return the bottom view — the set of nodes visible when the tree is viewed from below. For each horizontal distance (column), the bottommost (last-encountered in level order) node should appear.

**Example:**
- Input: Same tree as Top View: root `1` (hd=0), left `2` (hd=-1), right `3` (hd=1), and `2` has a right child `4` (hd=0, level 2).
- Output: `[2, 4, 3]` (node `4` overwrites node `1` at hd=0 since it's encountered later/deeper)

**Approach:** Nearly identical to Top View, but instead of keeping only the *first* node seen at each horizontal distance, we overwrite the dictionary entry every time a node is seen at that horizontal distance. Since BFS processes strictly level by level, the last write for a given horizontal distance corresponds to the deepest (bottommost) node in that column.

```csharp
public class Solution {
    public IList<int> BottomView(TreeNode root) {
        List<int> result = new List<int>();
        if (root == null) return result;

        Dictionary<int, int> bottomAtHd = new Dictionary<int, int>();
        Queue<(TreeNode node, int hd)> queue = new Queue<(TreeNode, int)>();
        queue.Enqueue((root, 0));

        int minHd = 0, maxHd = 0;

        while (queue.Count > 0) {
            var (node, hd) = queue.Dequeue();

            // Always overwrite -- the latest node at this hd is the deepest so far.
            bottomAtHd[hd] = node.val;

            minHd = Math.Min(minHd, hd);
            maxHd = Math.Max(maxHd, hd);

            if (node.left != null) queue.Enqueue((node.left, hd - 1));
            if (node.right != null) queue.Enqueue((node.right, hd + 1));
        }

        for (int hd = minHd; hd <= maxHd; hd++) {
            result.Add(bottomAtHd[hd]);
        }

        return result;
    }
}
```

Time Complexity: O(n) — single BFS pass.
Space Complexity: O(n) — dictionary and queue.

**Explanation:** Dry run on the same tree `1 -> (2 -> (null, 4)), 3`:
- Dequeue `(1,0)`: `bottomAtHd = {0:1}`.
- Dequeue `(2,-1)`: `bottomAtHd = {0:1, -1:2}`.
- Dequeue `(3,1)`: `bottomAtHd = {0:1, -1:2, 1:3}`.
- Dequeue `(4,0)`: hd `0` is overwritten → `bottomAtHd = {0:4, -1:2, 1:3}`.
- Output in order of hd `-1, 0, 1`: `[2, 4, 3]`.

---

## 11. Left View and Right View of a Binary Tree

**Problem Statement:** Given a binary tree, the left view consists of the first node visible at each level when looking from the left side; the right view consists of the first node visible at each level when looking from the right side.

**Example:**
- Input: A tree with root `1`, left `2` (with right child `4`), right `3`.
- Output: Left View = `[1, 2, 4]`, Right View = `[1, 3, 4]`

**Approach:** Two common techniques:
1. **BFS (level order):** For each level, take the first node dequeued for the left view, or the last node dequeued for the right view.
2. **DFS with depth tracking:** Recursively traverse (for right view: root, right, left; for left view: root, left, right), passing the current depth. Maintain a `List<int>` result; whenever the current depth equals `result.Count`, it means this is the first node encountered at this depth via this traversal order, so add it.

Both are shown below; the DFS version is often preferred for its O(h) extra space (aside from output) instead of O(width) queue.

```csharp
public class Solution {
    // ---- BFS approach ----
    public IList<int> LeftView(TreeNode root) {
        List<int> result = new List<int>();
        if (root == null) return result;

        Queue<TreeNode> queue = new Queue<TreeNode>();
        queue.Enqueue(root);

        while (queue.Count > 0) {
            int levelSize = queue.Count;
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.Dequeue();
                if (i == 0) result.Add(node.val); // first node of the level
                if (node.left != null) queue.Enqueue(node.left);
                if (node.right != null) queue.Enqueue(node.right);
            }
        }
        return result;
    }

    // ---- DFS approach ----
    public IList<int> RightView(TreeNode root) {
        List<int> result = new List<int>();
        RightViewDfs(root, 0, result);
        return result;
    }

    private void RightViewDfs(TreeNode node, int depth, List<int> result) {
        if (node == null) return;

        // First time we reach this depth, it must be the rightmost node
        // because we visit right before left.
        if (depth == result.Count) {
            result.Add(node.val);
        }

        RightViewDfs(node.right, depth + 1, result);
        RightViewDfs(node.left, depth + 1, result);
    }
}
```

Time Complexity: O(n) for both approaches — every node visited once.
Space Complexity: BFS: O(w) where `w` is the maximum width of the tree (queue size). DFS: O(h) recursion stack.

**Explanation:** Dry run of `RightViewDfs` (right view) on tree `1 -> (2 -> (null, 4)), 3`:
- `RightViewDfs(1, 0, [])`: `depth(0) == result.Count(0)` → add `1`. `result = [1]`.
- Recurse right first: `RightViewDfs(3, 1, [1])`: `depth(1) == result.Count(1)` → add `3`. `result = [1, 3]`. `3` has no children.
- Recurse left: `RightViewDfs(2, 1, [1,3])`: `depth(1) != result.Count(2)` → skip adding.
  - Recurse right: `RightViewDfs(4, 2, [1,3])`: `depth(2) == result.Count(2)` → add `4`. `result = [1, 3, 4]`.
  - Recurse left: `RightViewDfs(null, 2, ...)` → returns immediately.
- Final Right View: `[1, 3, 4]`. (Left View via BFS on the same tree gives `[1, 2, 4]`.)

---

## 12. Check if a Binary Tree is Symmetric

**Problem Statement:** Given the root of a binary tree, check whether it is a mirror of itself, i.e., symmetric around its center.

**Example:**
- Input: A tree with root `1`, left child `2` (with children `3`, `4`), right child `2` (with children `4`, `3`).
- Output: `true`

**Approach:** A tree is symmetric if its left and right subtrees are mirror images of each other. Write a helper function that compares two subtrees for mirror-equality: they are mirrors if both are `null`, or both have equal values and the left subtree of one mirrors the right subtree of the other (and vice versa). Call this helper on `root.left` and `root.right`.

```csharp
public class Solution {
    public bool IsSymmetric(TreeNode root) {
        if (root == null) return true;
        return IsMirror(root.left, root.right);
    }

    private bool IsMirror(TreeNode t1, TreeNode t2) {
        if (t1 == null && t2 == null) return true;
        if (t1 == null || t2 == null) return false;
        if (t1.val != t2.val) return false;

        // Outer pair (t1.left with t2.right) and inner pair (t1.right with t2.left)
        return IsMirror(t1.left, t2.right) && IsMirror(t1.right, t2.left);
    }
}
```

Time Complexity: O(n) — every node compared once.
Space Complexity: O(h) — recursion stack.

**Explanation:** Dry run on tree `1 -> (2 -> (3, 4)), (2 -> (4, 3))`:
- `IsSymmetric(1)` calls `IsMirror(leftChild=2a, rightChild=2b)` where `2a` has children `(3, 4)` and `2b` has children `(4, 3)`.
- `IsMirror(2a, 2b)`: values equal (`2 == 2`). Recurse `IsMirror(2a.left=3, 2b.right=3)` and `IsMirror(2a.right=4, 2b.left=4)`.
  - `IsMirror(3, 3)`: values equal, both leaves, children are all `null` → `IsMirror(null,null)` twice → `true`.
  - `IsMirror(4, 4)`: values equal, both leaves → `true`.
- Both recursive calls return `true`, so `IsMirror(2a, 2b) = true`.
- `IsSymmetric` returns `true`.
