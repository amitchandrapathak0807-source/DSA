# Binary Trees — Construction and Advanced Problems

Node definition used throughout this document:

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

---

## 1. Root to Node Path in a Binary Tree

**Problem Statement:** Given the root of a binary tree and a target value, find the path (sequence of node values) from the root to the node containing the target value.

**Example:**
- Input: Tree = `1 -> (2 -> (4, 5), 3)` i.e. `1` has children `2, 3`; `2` has children `4, 5`. Target = `5`.
- Output: `[1, 2, 5]`

**Approach:** Use DFS with backtracking. Maintain a path list. At each node, push the current node's value onto the path, then check if it's the target — if so, the path so far is the answer. Otherwise recurse into left and right children. If neither subtree contains the target, pop the current node off the path before returning (backtrack), since it doesn't lie on the path to the target.

```csharp
public class Solution {
    public IList<int> RootToNodePath(TreeNode root, int target) {
        List<int> path = new List<int>();
        FindPath(root, target, path);
        return path;
    }

    private bool FindPath(TreeNode node, int target, List<int> path) {
        if (node == null) return false;

        path.Add(node.val);

        if (node.val == target) return true;

        if (FindPath(node.left, target, path) || FindPath(node.right, target, path)) {
            return true;
        }

        // Backtrack: this node is not on the path to target
        path.RemoveAt(path.Count - 1);
        return false;
    }
}
```

**Time Complexity:** O(N) in the worst case — we may visit every node once.
**Space Complexity:** O(H) for recursion stack plus O(H) for the path list, where H is the height of the tree (worst case O(N) for a skewed tree).

**Explanation:** The key insight is backtracking — we optimistically add every node we visit to the path, and only remove it if neither of its subtrees leads to the target. This guarantees that when we finally find the target, the `path` list contains exactly the nodes from root to target, in order.

---

## 2. Lowest Common Ancestor (LCA) in a Binary Tree

**Problem Statement:** Given the root of a binary tree and two node values `p` and `q` (both guaranteed to exist in the tree), find their lowest common ancestor — the deepest node that has both `p` and `q` as descendants (a node can be a descendant of itself).

**Example:**
- Input: Tree = `3 -> (5 -> (6, 2 -> (7, 4)), 1 -> (0, 8))`. p = `5`, q = `1`.
- Output: `3`

**Approach:** Use the classic recursive trick: at each node, if the node is `null`, or matches `p`, or matches `q`, return the node itself. Otherwise recurse into left and right subtrees. If both left and right recursive calls return non-null, it means `p` and `q` were found in different subtrees, so the current node is the LCA — return it. If only one side returns non-null, propagate that result upward (the LCA lies entirely in that subtree, or one of p/q is an ancestor of the other and was already found).

```csharp
public class Solution {
    public TreeNode LowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) {
            return root;
        }

        TreeNode leftLCA = LowestCommonAncestor(root.left, p, q);
        TreeNode rightLCA = LowestCommonAncestor(root.right, p, q);

        if (leftLCA != null && rightLCA != null) {
            // p found in one subtree, q found in the other -> current node is LCA
            return root;
        }

        // Either both null, or only one side found something - propagate it up
        return leftLCA != null ? leftLCA : rightLCA;
    }
}
```

**Time Complexity:** O(N) — every node is visited at most once.
**Space Complexity:** O(H) for the recursion stack, where H is the tree height.

**Explanation:** This trick works because of how the recursion propagates results. When a call returns non-null, it means "somewhere in my subtree I found p, q, or their LCA." If both children report a non-null find, the current node must be the split point — hence the LCA. If a node returns non-null that node itself is either p, q, or an already-discovered LCA, and it keeps bubbling up unchanged through ancestors until one has non-null on both sides (or until it reaches the root untouched, meaning it IS the answer).

---

## 3. Maximum Width of a Binary Tree

**Problem Statement:** Given the root of a binary tree, find the maximum width of the tree. The width of a level is the number of nodes between the leftmost and rightmost non-null nodes at that level (inclusive), counting null nodes in between as if the tree were a complete binary tree.

**Example:**
- Input: Tree = `1 -> (3 -> (5, 3), 2 -> (null, 9))`
- Output: `4` (the bottom level has positional indices spanning a width of 4)

**Approach:** Use BFS (level order traversal) with positional indexing, treating the tree as if it were a complete binary tree stored in an array: for a node at index `i`, its left child is at `2*i + 1` and right child at `2*i + 2` (0-indexed) or `2*i` / `2*i + 1` (1-indexed). Push `(node, index)` pairs into a queue. For each level, the width is `lastIndex - firstIndex + 1`. To avoid integer overflow when indices grow exponentially with tree depth, re-index each level so the leftmost node of that level starts at index 0 (subtract the first index of the level from every index used within that level).

```csharp
public class Solution {
    public int WidthOfBinaryTree(TreeNode root) {
        if (root == null) return 0;

        int maxWidth = 0;
        Queue<(TreeNode node, long index)> queue = new Queue<(TreeNode, long)>();
        queue.Enqueue((root, 0));

        while (queue.Count > 0) {
            int levelSize = queue.Count;
            long firstIndex = queue.Peek().index;
            long lastIndex = firstIndex;

            for (int i = 0; i < levelSize; i++) {
                var (node, index) = queue.Dequeue();
                // Re-index relative to the first index of this level to avoid overflow
                long curIndex = index - firstIndex;
                lastIndex = curIndex;

                if (node.left != null) {
                    queue.Enqueue((node.left, 2 * curIndex));
                }
                if (node.right != null) {
                    queue.Enqueue((node.right, 2 * curIndex + 1));
                }
            }

            maxWidth = Math.Max(maxWidth, (int)(lastIndex - 0 + 1));
        }

        return maxWidth;
    }
}
```

**Time Complexity:** O(N) — each node is enqueued and dequeued exactly once.
**Space Complexity:** O(N) for the queue in the worst case (a fully filled last level).

**Explanation:** By re-indexing at the start of every level (subtracting `firstIndex`), the indices used for computing `2*i` and `2*i+1` stay small and bounded within each level, preventing overflow that would otherwise occur on deep skewed trees where indices would double at every level.

---

## 4. Check Children Sum Property

**Problem Statement:** A binary tree satisfies the "children sum property" if, for every node that has at least one child, the node's value equals the sum of the values of its left and right children (treating a missing child as contributing 0). Verify whether a given tree satisfies this property.

**Example:**
- Input: Tree = `10 -> (4 -> (2, 2), 6 -> (3, 3))`
- Output: `true` (4 = 2+2, 6 = 3+3, 10 = 4+6)

**Approach:** Use DFS validation. For each node, if it is a leaf or `null`, the property trivially holds (nothing to check). Otherwise compute `leftVal = node.left?.val ?? 0` and `rightVal = node.right?.val ?? 0`, check that `node.val == leftVal + rightVal`, and recursively validate both subtrees. A short-circuiting `&&` ensures we stop early on the first violation. There is also a common "fix" variant of this problem where, instead of validating, you adjust child values on the way down (or parent values on the way up during a post-order pass) to force the property to hold; that variant is noted below the main solution.

```csharp
public class Solution {
    public bool IsChildrenSumProperty(TreeNode root) {
        if (root == null || (root.left == null && root.right == null)) {
            return true;
        }

        int leftVal = root.left != null ? root.left.val : 0;
        int rightVal = root.right != null ? root.right.val : 0;

        bool currentHolds = root.val == leftVal + rightVal;

        return currentHolds
            && IsChildrenSumProperty(root.left)
            && IsChildrenSumProperty(root.right);
    }

    // Variant: "fix" the tree in-place so the property holds everywhere,
    // by pushing the excess/deficit down to a single child, then fixing
    // leaf values on the way back up (post-order) by summing children.
    public void FixChildrenSumProperty(TreeNode root) {
        if (root == null) return;

        int childSum = 0;
        if (root.left != null) childSum += root.left.val;
        if (root.right != null) childSum += root.right.val;

        if (childSum >= root.val) {
            root.val = childSum;
        } else {
            // Push root's value down to whichever child exists
            if (root.left != null) root.left.val = root.val;
            else if (root.right != null) root.right.val = root.val;
        }

        FixChildrenSumProperty(root.left);
        FixChildrenSumProperty(root.right);

        // Post-order fix-up: recompute this node's value from (possibly updated) children
        int total = 0;
        if (root.left != null) total += root.left.val;
        if (root.right != null) total += root.right.val;
        if (root.left != null || root.right != null) root.val = total;
    }
}
```

**Time Complexity:** O(N) — each node visited once.
**Space Complexity:** O(H) for the recursion stack.

**Explanation:** The validation check is purely local at each node (comparing the node's value against its two immediate children), which is why a single top-down DFS pass with early termination is sufficient — there's no need to look further down than one level at each step.

---

## 5. Print All Nodes at Distance K from a Given Node

**Problem Statement:** Given the root of a binary tree, a target node, and an integer `K`, print/return the values of all nodes that are exactly at distance `K` from the target node. Distance can go upward (through ancestors), downward (through descendants), or sideways (through a sibling subtree).

**Example:**
- Input: Tree = `3 -> (5 -> (6, 2 -> (7, 4)), 1 -> (0, 8))`. Target = `5`, K = `2`.
- Output: `[7, 4, 1]`

**Approach:** A plain binary tree only has downward (parent -> child) pointers, but distance-K nodes may require moving upward too. So first do a BFS/DFS pass to build a `Dictionary<TreeNode, TreeNode>` mapping each node to its parent. This effectively turns the tree into an undirected graph. Then run a standard BFS starting from the target node, treating left child, right child, and parent as the three possible neighbors, using a `HashSet<TreeNode>` to track visited nodes and avoid revisiting the node we came from. After exactly `K` BFS layers, the nodes remaining in the queue are the answer.

```csharp
public class Solution {
    public IList<int> DistanceK(TreeNode root, TreeNode target, int k) {
        Dictionary<TreeNode, TreeNode> parentMap = new Dictionary<TreeNode, TreeNode>();
        BuildParentMap(root, null, parentMap);

        Queue<TreeNode> queue = new Queue<TreeNode>();
        HashSet<TreeNode> visited = new HashSet<TreeNode>();
        queue.Enqueue(target);
        visited.Add(target);

        int currentDistance = 0;
        while (queue.Count > 0 && currentDistance < k) {
            int levelSize = queue.Count;
            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.Dequeue();

                if (node.left != null && !visited.Contains(node.left)) {
                    visited.Add(node.left);
                    queue.Enqueue(node.left);
                }
                if (node.right != null && !visited.Contains(node.right)) {
                    visited.Add(node.right);
                    queue.Enqueue(node.right);
                }
                if (parentMap.ContainsKey(node) && !visited.Contains(parentMap[node])) {
                    visited.Add(parentMap[node]);
                    queue.Enqueue(parentMap[node]);
                }
            }
            currentDistance++;
        }

        List<int> result = new List<int>();
        foreach (TreeNode node in queue) {
            result.Add(node.val);
        }
        return result;
    }

    private void BuildParentMap(TreeNode node, TreeNode parent, Dictionary<TreeNode, TreeNode> parentMap) {
        if (node == null) return;
        parentMap[node] = parent;
        BuildParentMap(node.left, node, parentMap);
        BuildParentMap(node.right, node, parentMap);
    }
}
```

**Time Complexity:** O(N) — O(N) to build the parent map, O(N) for the BFS in the worst case.
**Space Complexity:** O(N) for the parent map, visited set, and queue.

**Explanation:** See the dry run below for why parent pointers are necessary and how the BFS unfolds.

---

## 6. Minimum Time to Burn a Binary Tree Starting from a Given Node

**Problem Statement:** A fire starts at a given target node in a binary tree. Each minute, fire spreads from a burning node to its directly connected neighbors (left child, right child, and parent). Find the minimum time (in minutes) required to burn the entire tree.

**Example:**
- Input: Tree = `1 -> (2 -> (4, 5), 3 -> (null, 6))`. Target = `4`.
- Output: `4` (path 4 -> 2 -> 1 -> 3 -> 6 takes 4 minutes to reach the farthest node `6`)

**Approach:** Identical setup to the distance-K problem: build a parent map via DFS/BFS to allow upward movement, then run BFS from the target node level by level, treating left, right, and parent as neighbors and using a visited set. The total number of BFS levels needed to visit every node in the tree is the answer — it is exactly the maximum distance (in the undirected-graph sense) from the target node to any other node.

```csharp
public class Solution {
    public int MinTimeToBurnTree(TreeNode root, int targetVal) {
        Dictionary<TreeNode, TreeNode> parentMap = new Dictionary<TreeNode, TreeNode>();
        TreeNode targetNode = BuildParentMapAndFindTarget(root, null, targetVal, parentMap);

        HashSet<TreeNode> visited = new HashSet<TreeNode>();
        Queue<TreeNode> queue = new Queue<TreeNode>();
        queue.Enqueue(targetNode);
        visited.Add(targetNode);

        int minutes = 0;
        while (queue.Count > 0) {
            int levelSize = queue.Count;
            bool spreadHappened = false;

            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.Dequeue();

                if (node.left != null && !visited.Contains(node.left)) {
                    visited.Add(node.left);
                    queue.Enqueue(node.left);
                    spreadHappened = true;
                }
                if (node.right != null && !visited.Contains(node.right)) {
                    visited.Add(node.right);
                    queue.Enqueue(node.right);
                    spreadHappened = true;
                }
                if (parentMap.ContainsKey(node) && parentMap[node] != null && !visited.Contains(parentMap[node])) {
                    visited.Add(parentMap[node]);
                    queue.Enqueue(parentMap[node]);
                    spreadHappened = true;
                }
            }

            if (spreadHappened) minutes++;
        }

        return minutes;
    }

    private TreeNode BuildParentMapAndFindTarget(TreeNode node, TreeNode parent, int targetVal, Dictionary<TreeNode, TreeNode> parentMap) {
        if (node == null) return null;

        parentMap[node] = parent;
        TreeNode found = (node.val == targetVal) ? node : null;

        TreeNode leftResult = BuildParentMapAndFindTarget(node.left, node, targetVal, parentMap);
        TreeNode rightResult = BuildParentMapAndFindTarget(node.right, node, targetVal, parentMap);

        return found ?? leftResult ?? rightResult;
    }
}
```

**Time Complexity:** O(N) — building the parent map and the BFS traversal are each O(N).
**Space Complexity:** O(N) for the parent map, visited set, and queue.

**Explanation:** This is essentially the "distance K" problem generalized: instead of stopping at a fixed K, we keep expanding the BFS frontier until no new nodes are discovered, and count how many levels that took. Each BFS level corresponds to exactly one minute of fire spread.

---

## 7. Count Total Nodes in a Complete Binary Tree (Better than O(N))

**Problem Statement:** Given the root of a *complete* binary tree (every level is fully filled except possibly the last, which is filled left to right), count the total number of nodes, in better than O(N) time.

**Example:**
- Input: Tree = `1 -> (2 -> (4, 5), 3 -> (6))`
- Output: `6`

**Approach:** Exploit the complete binary tree structure. For any subtree rooted at `node`, compute the height by going strictly left (`leftHeight`) and strictly right (`rightHeight`). If `leftHeight == rightHeight`, the subtree is a **full/perfect** binary tree, and its node count is `2^leftHeight - 1` — computed directly without recursing further. If they differ, the subtree is not full, so recursively count nodes in the left and right children and add 1 for the current node. Because at least one of the two recursive calls at each level hits the "full subtree" base case (a property of complete binary trees), the recursion depth is O(log N) and at each of those O(log N) levels we do O(log N) work to compute heights, giving O(log^2 N) total.

```csharp
public class Solution {
    public int CountNodes(TreeNode root) {
        if (root == null) return 0;

        int leftHeight = GetLeftHeight(root);
        int rightHeight = GetRightHeight(root);

        if (leftHeight == rightHeight) {
            // Perfect binary tree: node count = 2^h - 1
            return (1 << leftHeight) - 1;
        }

        // Not a perfect subtree here: recurse on both children
        return 1 + CountNodes(root.left) + CountNodes(root.right);
    }

    private int GetLeftHeight(TreeNode node) {
        int height = 0;
        while (node != null) {
            height++;
            node = node.left;
        }
        return height;
    }

    private int GetRightHeight(TreeNode node) {
        int height = 0;
        while (node != null) {
            height++;
            node = node.right;
        }
        return height;
    }
}
```

**Time Complexity:** O(log^2 N). At each recursive call we spend O(log N) computing the left/right heights, and because of the "skip full subtrees" trick, we only ever recurse into one non-full child per level (the other child, if it exists as a separate recursive call, resolves in O(1) via the perfect-tree formula), giving O(log N) levels of recursion, each doing O(log N) height-computation work: O(log N) * O(log N) = O(log^2 N).
**Space Complexity:** O(log N) for the recursion stack (proportional to tree height).

**Explanation of the O(log^2 N) trick:** In a complete binary tree, comparing the height obtained by always going left versus always going right tells us instantly whether the *current* subtree is perfect — if equal, we can use the closed-form formula `2^h - 1` instead of counting node by node. If they're unequal, we know the subtree is not perfect, but critically, in a complete binary tree, *one* of its two children's subtrees must still be perfect (or resolvable this same way), so we never do wasted linear work; we simply pay O(log N) per level for height checks across O(log N) levels of recursion, avoiding the naive O(N) full traversal.

---

## 8. Construct a Binary Tree from Inorder and Preorder Traversal

**Problem Statement:** Given the `preorder` and `inorder` traversal arrays of a binary tree (with unique node values), reconstruct and return the binary tree.

**Example:**
- Input: `preorder = [3, 9, 20, 15, 7]`, `inorder = [9, 3, 15, 20, 7]`
- Output: Tree = `3 -> (9, 20 -> (15, 7))`

**Approach:** In preorder, the first element is always the root. Find that root's position in the inorder array — everything to its left in inorder belongs to the left subtree, everything to its right belongs to the right subtree. Use a `Dictionary<int,int>` to map each value to its index in the inorder array for O(1) lookups (instead of an O(N) linear search each time, which would make the whole algorithm O(N^2)). Track a moving `preorderIndex` (starts at 0, incremented each time we consume a new "root" from preorder) and recurse on the inorder sub-ranges `[inStart, rootIndex-1]` (left) and `[rootIndex+1, inEnd]` (right).

```csharp
public class Solution {
    private int preorderIndex;
    private Dictionary<int, int> inorderIndexMap;

    public TreeNode BuildTree(int[] preorder, int[] inorder) {
        preorderIndex = 0;
        inorderIndexMap = new Dictionary<int, int>();
        for (int i = 0; i < inorder.Length; i++) {
            inorderIndexMap[inorder[i]] = i;
        }

        return Build(preorder, 0, inorder.Length - 1);
    }

    private TreeNode Build(int[] preorder, int inStart, int inEnd) {
        if (inStart > inEnd) return null;

        int rootVal = preorder[preorderIndex];
        preorderIndex++;

        TreeNode root = new TreeNode(rootVal);

        int rootIndexInInorder = inorderIndexMap[rootVal];

        // Build left subtree first (preorder: root, left, right)
        root.left = Build(preorder, inStart, rootIndexInInorder - 1);
        root.right = Build(preorder, rootIndexInInorder + 1, inEnd);

        return root;
    }
}
```

**Time Complexity:** O(N) — each node is processed once, and the HashMap gives O(1) index lookups (versus O(N^2) with a naive linear search per node).
**Space Complexity:** O(N) for the hashmap plus O(H) recursion stack (worst case O(N) for a skewed tree).

**Explanation (dry run):** See the combined dry run below.

---

## 9. Construct a Binary Tree from Inorder and Postorder Traversal

**Problem Statement:** Given the `postorder` and `inorder` traversal arrays of a binary tree (unique values), reconstruct and return the binary tree.

**Example:**
- Input: `inorder = [9, 3, 15, 20, 7]`, `postorder = [9, 15, 7, 20, 3]`
- Output: Tree = `3 -> (9, 20 -> (15, 7))`

**Approach:** In postorder, the *last* element is always the root (postorder visits left, right, root). Use the same `Dictionary<int,int>` inorder-value-to-index trick for O(1) root lookups. Track a moving `postorderIndex` starting at `postorder.Length - 1`, decrementing after each use. Since postorder processes root, then effectively right-subtree-last, we must build the **right** subtree first, then the **left** subtree, mirroring the reversed nature of postorder traversal (root is consumed from the back, and the right subtree's postorder segment immediately precedes the root in the array).

```csharp
public class Solution {
    private int postorderIndex;
    private Dictionary<int, int> inorderIndexMap;

    public TreeNode BuildTree(int[] inorder, int[] postorder) {
        postorderIndex = postorder.Length - 1;
        inorderIndexMap = new Dictionary<int, int>();
        for (int i = 0; i < inorder.Length; i++) {
            inorderIndexMap[inorder[i]] = i;
        }

        return Build(postorder, 0, inorder.Length - 1);
    }

    private TreeNode Build(int[] postorder, int inStart, int inEnd) {
        if (inStart > inEnd) return null;

        int rootVal = postorder[postorderIndex];
        postorderIndex--;

        TreeNode root = new TreeNode(rootVal);

        int rootIndexInInorder = inorderIndexMap[rootVal];

        // Build right subtree first (postorder consumed from the end: ..., left, right, root)
        root.right = Build(postorder, rootIndexInInorder + 1, inEnd);
        root.left = Build(postorder, inStart, rootIndexInInorder - 1);

        return root;
    }
}
```

**Time Complexity:** O(N) — same reasoning as the preorder version, thanks to the O(1) hashmap lookups.
**Space Complexity:** O(N) for the hashmap plus O(H) recursion stack.

**Explanation:** Since postorder is `[left subtree, right subtree, root]`, reading from the back gives `root` first, then elements belonging to the right subtree (in reverse postorder), then the left subtree. This is why the recursive calls must build the right child before the left child — it keeps `postorderIndex` decrementing through the correct segment of the array in sync with the recursion.

---

## 10. Serialize and Deserialize a Binary Tree

**Problem Statement:** Design an algorithm to convert a binary tree into a string (serialize), and convert that string back into the original binary tree structure (deserialize). The tree may contain duplicate values, so traversal order alone (without null markers) is not sufficient.

**Example:**
- Input: Tree = `1 -> (2, 3 -> (4, 5))`
- Serialized: `"1,2,#,#,3,4,#,#,5,#,#"` (using `#` as a null marker, preorder with explicit nulls)
- Output (deserialize): reconstructs the original tree exactly

**Approach:** Use a preorder traversal where every node's value is written to the output, and every `null` child is explicitly written as a sentinel marker (e.g., `"#"`), separated by commas. This "level-marker" encoding removes any ambiguity, since preorder + explicit null markers uniquely determines the tree shape without needing a second traversal (unlike plain preorder/inorder pairs). To deserialize, split the string by the delimiter and consume tokens one at a time, recursively rebuilding: if the current token is the null marker, return `null`; otherwise create a node, recursively build its left subtree, then its right subtree, consuming tokens as you go (use a `Queue<string>` for easy sequential consumption).

```csharp
public class Codec {
    public string Serialize(TreeNode root) {
        StringBuilder sb = new StringBuilder();
        SerializeHelper(root, sb);
        return sb.ToString();
    }

    private void SerializeHelper(TreeNode node, StringBuilder sb) {
        if (node == null) {
            sb.Append("#,");
            return;
        }
        sb.Append(node.val).Append(',');
        SerializeHelper(node.left, sb);
        SerializeHelper(node.right, sb);
    }

    public TreeNode Deserialize(string data) {
        Queue<string> tokens = new Queue<string>(data.Split(','));
        return DeserializeHelper(tokens);
    }

    private TreeNode DeserializeHelper(Queue<string> tokens) {
        string token = tokens.Dequeue();
        if (token == "#") {
            return null;
        }

        TreeNode node = new TreeNode(int.Parse(token));
        node.left = DeserializeHelper(tokens);
        node.right = DeserializeHelper(tokens);
        return node;
    }
}
```

**Time Complexity:** O(N) for both serialize and deserialize — each node is visited/consumed exactly once.
**Space Complexity:** O(N) for the output string / token queue, plus O(H) recursion stack.

**Explanation:** Because every null child is explicitly recorded, the preorder sequence alone is enough to unambiguously reconstruct the tree — there is no need for a second traversal (like inorder) to disambiguate structure, which is required when nulls are omitted.

---

## 11. Flatten a Binary Tree to a Linked List

**Problem Statement:** Given the root of a binary tree, flatten it in-place into a "linked list" that follows the preorder traversal order. After flattening, for every node, the `left` pointer should be `null`, and the `right` pointer should point to the next node in preorder sequence.

**Example:**
- Input: Tree = `1 -> (2 -> (3, 4), 5 -> (null, 6))`
- Output (as a right-only chain): `1 -> 2 -> 3 -> 4 -> 5 -> 6`

**Approach:** The elegant O(1)-extra-space technique uses a **Morris-traversal-style** in-place pointer rewiring, avoiding recursion or an explicit stack. Process nodes with a `cur` pointer starting at root. At each step, if `cur.left` exists, find the rightmost node of `cur`'s left subtree (its inorder predecessor with respect to the right-chain, i.e., the "rightmost" node reachable by going left once then right repeatedly) and attach `cur.right` to that rightmost node's `right` pointer. Then move `cur.left` to `cur.right`, and set `cur.left = null`. Advance `cur = cur.right` and repeat until `cur` is null. An alternative simpler (but O(H) space) approach is recursive: recursively flatten the left and right subtrees first (post-order-ish), then splice the flattened left subtree between the node and its (already flattened) right subtree.

```csharp
public class Solution {
    // Morris-traversal-style, O(1) extra space
    public void Flatten(TreeNode root) {
        TreeNode cur = root;

        while (cur != null) {
            if (cur.left != null) {
                // Find the rightmost node in cur's left subtree
                TreeNode rightmost = cur.left;
                while (rightmost.right != null) {
                    rightmost = rightmost.right;
                }

                // Rewire: attach the original right subtree after the rightmost node
                rightmost.right = cur.right;

                // Move left subtree to the right, clear left
                cur.right = cur.left;
                cur.left = null;
            }

            cur = cur.right;
        }
    }

    // Alternative: recursive, right-subtree-relinking approach, O(H) space
    private TreeNode prev = null;

    public void FlattenRecursive(TreeNode root) {
        if (root == null) return;

        FlattenRecursive(root.right);
        FlattenRecursive(root.left);

        root.right = prev;
        root.left = null;
        prev = root;
    }
}
```

**Time Complexity:** O(N) for both approaches — the Morris-style version visits each node's "rightmost of left subtree" chain at most once overall (amortized O(N) across the whole traversal, similar to Morris inorder traversal), and the recursive version is a straightforward O(N) tree traversal.
**Space Complexity:** Morris-style: O(1) extra space (no recursion stack, no auxiliary structure). Recursive version: O(H) recursion stack.

**Explanation:** In the Morris-style approach, we never lose access to the right subtree: before overwriting `cur.right` with the left subtree, we first attach the old right subtree to the tail (rightmost node) of the left subtree, so no nodes are ever dropped. This is analogous to Morris traversal's technique of temporarily rewiring `null` pointers to enable O(1)-space traversal, except here the rewiring is permanent and is the actual goal, not a temporary scaffold.

---

## Dry Run A: Parent Pointers + BFS for "Nodes at Distance K"

Consider the tree:

```
        3
       / \
      5   1
     / \  / \
    6  2 0   8
      / \
     7   4
```

Target node = `5`, K = `2`.

**Why parent pointers are needed:** The input tree only has `left`/`right` pointers (a directed, downward-only structure). But distance-K neighbors of `5` include `1` (its sibling's parent path) and `7`/`4` (its child's children) — reaching `1` from `5` requires moving *upward* through the parent `3`, which no field on `TreeNode` supports. So we first do one DFS/BFS pass over the whole tree to build `Dictionary<TreeNode, TreeNode> parentMap`, recording each node's parent (root maps to `null`). This effectively converts the tree into an undirected graph where every node has up to 3 neighbors: `left`, `right`, and `parent`.

Parent map (partial, relevant nodes): `5 -> 3`, `3 -> null`, `1 -> 3`, `6 -> 5`, `2 -> 5`, `7 -> 2`, `4 -> 2`, `0 -> 1`, `8 -> 1`.

**BFS from target `5`:**
- Level 0: `{5}` (distance 0)
- Level 1 (distance 1): neighbors of `5` = `left(6)`, `right(2)`, `parent(3)` → `{6, 2, 3}`
- Level 2 (distance 2): expand each of `6`, `2`, `3`:
  - `6`: no left/right, parent is `5` (already visited) → nothing new
  - `2`: left `7`, right `4`, parent `5` (visited) → adds `7`, `4`
  - `3`: left `5` (visited), right `1`, parent `null` → adds `1`
  - Result: `{7, 4, 1}`

Since K=2, we stop here. **Answer: `[7, 4, 1]`**, matching the earlier example. This confirms that without the parent map, `1` (reached by going up from `5` to `3` then down to `1`) would have been unreachable using only `left`/`right` pointers.

---

## Dry Run B: Constructing a Tree from Preorder and Inorder

Given: `preorder = [3, 9, 20, 15, 7]`, `inorder = [9, 3, 15, 20, 7]`.

Build `inorderIndexMap`: `{9:0, 3:1, 15:2, 20:3, 7:4}`. `preorderIndex` starts at 0.

**Call `Build(inStart=0, inEnd=4)`:**
- `rootVal = preorder[0] = 3`, increment `preorderIndex` to 1.
- `rootIndexInInorder = inorderIndexMap[3] = 1`.
- Inorder array split by index 1: left range = `[0, 0]` (values `[9]`), right range = `[2, 4]` (values `[15, 20, 7]`).
- Create node `3`.
- Recurse left: `Build(inStart=0, inEnd=0)`:
  - `rootVal = preorder[1] = 9`, increment `preorderIndex` to 2.
  - `rootIndexInInorder = inorderIndexMap[9] = 0`.
  - Left range `[0, -1]` → empty → `null`. Right range `[1, 0]` → empty → `null`.
  - Returns node `9` (leaf). Node `3`'s left = `9`.
- Recurse right: `Build(inStart=2, inEnd=4)`:
  - `rootVal = preorder[2] = 20`, increment `preorderIndex` to 3.
  - `rootIndexInInorder = inorderIndexMap[20] = 3`.
  - Left range `[2, 2]` (value `[15]`), right range `[4, 4]` (value `[7]`).
  - Create node `20`.
  - Recurse left: `Build(inStart=2, inEnd=2)`: `rootVal = preorder[3] = 15`, `preorderIndex` -> 4. Both sub-ranges empty → leaf node `15`. Node `20`'s left = `15`.
  - Recurse right: `Build(inStart=4, inEnd=4)`: `rootVal = preorder[4] = 7`, `preorderIndex` -> 5. Both sub-ranges empty → leaf node `7`. Node `20`'s right = `7`.
  - Returns node `20` with left `15`, right `7`. Node `3`'s right = `20`.

**Final tree:** `3 -> (left: 9, right: 20 -> (left: 15, right: 7))`, which matches the expected output tree exactly.

The key mechanic: the preorder array always tells us *which* node is the next root to place, while the inorder array (via the hashmap-found index) tells us exactly *how to split* the remaining values into "belongs to the left subtree" versus "belongs to the right subtree" — recursively, until sub-ranges become empty (`inStart > inEnd`), which is the base case producing `null` children.
