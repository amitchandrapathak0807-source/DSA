# Binary Trees — Morris Traversal

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

## Concept: Threaded Binary Trees (Morris Traversal)

Every recursive or stack-based tree traversal needs extra memory to remember "where to come back to" once a subtree is finished — this is the O(h) call stack (or explicit `Stack<TreeNode>`), where `h` is the height of the tree. Morris Traversal removes this need entirely, achieving **O(1) auxiliary space**, by temporarily turning the tree into a **threaded binary tree**.

**The core idea:**

For any node `curr` that has a left child, its **inorder predecessor** is the rightmost node in `curr`'s left subtree (keep going left once, then right as far as possible). Normally that predecessor's `right` pointer is `null` (it has no right child, since it's the rightmost node). Morris Traversal exploits this wasted `null` pointer:

1. **Create a thread:** Before descending into `curr.left`, find `curr`'s inorder predecessor and set `predecessor.right = curr`. This is the "thread" — a temporary link that lets us jump back up to `curr` later, exactly the way a stack frame or recursive return would have.
2. **Use the thread:** Once we finish processing `curr`'s left subtree and eventually walk down to the predecessor again, its `right` pointer now points back to `curr` instead of being `null`. Following that thread brings us right back up to `curr` — no stack needed.
3. **Remove the thread:** As soon as we arrive back at `curr` via the thread, we detect it (the predecessor's `right` is no longer `null` and points to `curr`), restore `predecessor.right = null`, and move on. This restores the original tree structure so the tree looks unmodified once traversal completes.

By doing this for every node with a left child, we effectively simulate the "return to parent" behavior of recursion using the tree's own existing (otherwise-unused) `null` pointers, instead of an explicit stack. Each edge in the tree is traversed at most twice (once to create/find the thread, once to use and remove it), so the overall time complexity remains **O(n)**, but the space complexity drops from **O(h)** to **O(1)**.

---

## Morris Inorder Traversal

### 1. Morris Inorder Traversal

**Problem Statement:** Given the root of a binary tree, return the **inorder traversal** (Left → Node → Right) of its node values using **O(1) extra space** (no recursion, no explicit stack), by threading the tree via Morris Traversal.

**Example:**
- Input: A tree with root `1`, `1.left = 2`, `2.left = 4`, `2.right = 5`, `1.right = 3`
  ```
          1
         / \
        2   3
       / \
      4   5
  ```
- Output: `[4, 2, 5, 1, 3]`

**Standard Approach (for comparison):** Iterative traversal using an explicit stack (equivalent to what recursion does implicitly), which uses O(h) space for the stack.

```csharp
public IList<int> InorderTraversalStandard(TreeNode root) {
    List<int> result = new List<int>();
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode curr = root;

    while (curr != null || stack.Count > 0) {
        // Go as far left as possible, pushing nodes onto the stack
        while (curr != null) {
            stack.Push(curr);
            curr = curr.left;
        }
        // Process the node
        curr = stack.Pop();
        result.Add(curr.val);
        // Move to the right subtree
        curr = curr.right;
    }

    return result;
}
```

Time Complexity: O(n) — every node is pushed and popped exactly once.
Space Complexity: O(h) — the stack can hold up to `h` nodes (height of the tree), which is O(log n) for a balanced tree but O(n) for a skewed tree.

**Morris (Optimized) Approach:** For each node `curr`, if it has no left child, visit it and move right. Otherwise, find its inorder predecessor (rightmost node in the left subtree). If the predecessor's right pointer is `null`, thread it to `curr` and move left (this is a "first visit"). If the predecessor's right pointer already points to `curr`, that means we've come back via the thread — remove the thread, visit `curr`, and move right.

```csharp
public IList<int> InorderTraversalMorris(TreeNode root) {
    List<int> result = new List<int>();
    TreeNode curr = root;

    while (curr != null) {
        if (curr.left == null) {
            // No left subtree: visit current node, move to right child
            result.Add(curr.val);
            curr = curr.right;
        } else {
            // Find the inorder predecessor of curr
            TreeNode predecessor = curr.left;
            while (predecessor.right != null && predecessor.right != curr) {
                predecessor = predecessor.right;
            }

            if (predecessor.right == null) {
                // Create the thread and move into the left subtree
                predecessor.right = curr;
                curr = curr.left;
            } else {
                // Thread already exists: we've returned from the left subtree.
                // Remove the thread, visit curr, then move right.
                predecessor.right = null;
                result.Add(curr.val);
                curr = curr.right;
            }
        }
    }

    return result;
}
```

Time Complexity: O(n) — Although finding the predecessor looks like it adds extra work on top of a simple traversal, each edge of the tree is visited at most twice: once while walking down to locate/create the thread, and once while walking the thread back up before it is removed. Summed across all nodes, the total work is still bounded by O(n) (amortized), not O(n · h).
Space Complexity: O(1) — no recursion stack or explicit stack is used; only a few pointer variables (`curr`, `predecessor`) are maintained.

**Explanation:** Step-by-step dry run of **Morris Inorder Traversal** on this tree:

```
            1
          /   \
         2     3
        / \
       4   5
        \
         6
```

(`1.left=2, 1.right=3, 2.left=4, 2.right=5, 4.right=6`)

Expected inorder output: `[4, 6, 2, 5, 1, 3]`

| Step | curr | Action |
|------|------|--------|
| 1 | 1 | `1.left = 2` (not null) → find predecessor of `1`: start at `2`, go right → `5`, go right → null. Predecessor = `5`. `5.right` is null, so **create thread**: `5.right = 1`. Move `curr = curr.left = 2`. |
| 2 | 2 | `2.left = 4` (not null) → find predecessor of `2`: start at `4`, go right → `6`, go right → null. Predecessor = `6`. `6.right` is null, so **create thread**: `6.right = 2`. Move `curr = curr.left = 4`. |
| 3 | 4 | `4.left = null` → **visit 4** (result = `[4]`). Move `curr = curr.right = 6` (the thread just created). |
| 4 | 6 | `6.left = null` → **visit 6** (result = `[4, 6]`). Move `curr = curr.right`. `6.right` currently points to `2` (the thread from step 2) — this is just the pointer, traversal doesn't "know" it's a thread here; it simply moves `curr = 2`. |
| 5 | 2 | `2.left = 4` (not null) → find predecessor of `2` again: start at `4`, go right → `6`, go right → `2` (equals `curr`!). Since `predecessor.right == curr`, this means the thread is being used to return. **Remove thread**: `6.right = null`. **Visit 2** (result = `[4, 6, 2]`). Move `curr = curr.right = 5`. |
| 6 | 5 | `5.left = null` → **visit 5** (result = `[4, 6, 2, 5]`). Move `curr = curr.right`. `5.right` currently points to `1` (the thread from step 1) → `curr = 1`. |
| 7 | 1 | `1.left = 2` (not null) → find predecessor of `1` again: start at `2`, go right → `5`, go right → `1` (equals `curr`!). **Remove thread**: `5.right = null`. **Visit 1** (result = `[4, 6, 2, 5, 1]`). Move `curr = curr.right = 3`. |
| 8 | 3 | `3.left = null` → **visit 3** (result = `[4, 6, 2, 5, 1, 3]`). Move `curr = curr.right = null`. |
| 9 | null | Loop ends. |

Final result: `[4, 6, 2, 5, 1, 3]` — correct inorder traversal, and the tree is fully restored (both threads were removed) by the time the loop finishes.

**Key takeaway on thread lifecycle:** A thread is *created* the first time we pass through a node with a left child (on the way down, before exploring the left subtree). It is *used and removed* the second time we reach that same predecessor node — which happens only after the entire left subtree has been visited, naturally bringing control back to the parent, exactly like a stack pop would.

---

## Morris Preorder Traversal

### 2. Morris Preorder Traversal

**Problem Statement:** Given the root of a binary tree, return the **preorder traversal** (Node → Left → Right) of its node values using **O(1) extra space**, again using Morris threading.

**Example:**
- Input: Same tree as above:
  ```
          1
         / \
        2   3
       / \
      4   5
  ```
- Output: `[1, 2, 4, 5, 3]`

**Standard Approach (for comparison):** Iterative traversal using an explicit stack.

```csharp
public IList<int> PreorderTraversalStandard(TreeNode root) {
    List<int> result = new List<int>();
    if (root == null) return result;

    Stack<TreeNode> stack = new Stack<TreeNode>();
    stack.Push(root);

    while (stack.Count > 0) {
        TreeNode curr = stack.Pop();
        result.Add(curr.val);

        // Push right first so left is processed first (LIFO order)
        if (curr.right != null) stack.Push(curr.right);
        if (curr.left != null) stack.Push(curr.left);
    }

    return result;
}
```

Time Complexity: O(n) — every node is pushed and popped exactly once.
Space Complexity: O(h) in the average/balanced case, O(n) worst case (skewed tree), due to the explicit stack.

**Morris (Optimized) Approach:** The structure is almost identical to Morris Inorder. The only change is **when** we visit the node: in preorder, a node must be output *before* its left subtree is explored. So instead of visiting the node when we detect the thread has been used (on the way back up), we visit it when we first create the thread (on the way down, before descending left).

```csharp
public IList<int> PreorderTraversalMorris(TreeNode root) {
    List<int> result = new List<int>();
    TreeNode curr = root;

    while (curr != null) {
        if (curr.left == null) {
            // No left subtree: visit current node, move to right child
            result.Add(curr.val);
            curr = curr.right;
        } else {
            // Find the inorder predecessor of curr
            TreeNode predecessor = curr.left;
            while (predecessor.right != null && predecessor.right != curr) {
                predecessor = predecessor.right;
            }

            if (predecessor.right == null) {
                // First time visiting curr: visit it now (preorder: node before left),
                // then create the thread and move into the left subtree
                result.Add(curr.val);
                predecessor.right = curr;
                curr = curr.left;
            } else {
                // Returning via thread: remove it and move right.
                // (curr was already visited when the thread was created, so no visit here.)
                predecessor.right = null;
                curr = curr.right;
            }
        }
    }

    return result;
}
```

Time Complexity: O(n) — same reasoning as inorder: each edge is traversed at most twice (once descending to build/find the thread, once returning along the thread before removing it), giving an amortized O(n) total despite the repeated predecessor searches.
Space Complexity: O(1) — only pointer variables `curr` and `predecessor` are used; no recursion or explicit stack.

**Explanation:** The dry run structure is identical to the inorder walkthrough above — the same sequence of predecessor searches, thread creations, and thread removals happens on the same tree. Using the earlier example tree:

```
            1
          /   \
         2     3
        / \
       4   5
        \
         6
```

The **only difference** from Morris Inorder is *when* the "visit" (add to result) happens:

- **Morris Inorder:** visit happens when we detect `predecessor.right == curr` (i.e., the *second* time we reach a node via its thread, after its entire left subtree has already been processed) — this correctly places the node *after* its left subtree's output, matching Left → Node → Right.
- **Morris Preorder:** visit happens when we detect `predecessor.right == null` (i.e., the *first* time we reach the node and are about to create the thread and descend left) — this correctly places the node *before* its left subtree's output, matching Node → Left → Right.

Tracing it: at `curr = 1`, since `1.left != null`, we find predecessor `5`. `5.right` is null → this is a first visit, so with Morris Preorder we **visit 1 immediately** (result = `[1]`), then thread `5.right = 1` and descend to `curr = 2`. Similarly at `curr = 2`, predecessor `6` has `right == null` → **visit 2** (result = `[1, 2]`), thread `6.right = 2`, descend to `curr = 4`. At `curr = 4` (`left == null`) → **visit 4** (result = `[1, 2, 4]`), move to `curr = 6` via `4.right` (a plain existing link, not a thread here since 4 has no left child of its own to thread). At `curr = 6` (`left == null`) → **visit 6** (result = `[1, 2, 4, 6]`), move `curr = 6.right = 2` (the thread). At `curr = 2` again, predecessor search reaches `6`, finds `6.right == curr (2)` → thread detected, **no visit** (already visited earlier), remove thread `6.right = null`, move `curr = 2.right = 5`. At `curr = 5` (`left == null`) → **visit 5** (result = `[1, 2, 4, 6, 5]`), move `curr = 5.right = 1` (thread). At `curr = 1` again, predecessor search reaches `5`, finds `5.right == curr (1)` → thread detected, **no visit**, remove thread `5.right = null`, move `curr = 1.right = 3`. At `curr = 3` (`left == null`) → **visit 3** (result = `[1, 2, 4, 6, 5, 3]`), move `curr = null`, loop ends.

Final preorder result: `[1, 2, 4, 6, 5, 3]`, confirming the correct Node → Left → Right order, achieved purely by shifting the visit point to the thread-creation moment instead of the thread-removal moment.
