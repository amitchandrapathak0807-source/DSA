# Binary Trees — Basic Traversals

## Node Definition

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

## Concept: Tree Traversals

A binary tree can be traversed in several standard orders, each useful for different problems:

- **Preorder (Root → Left → Right):** Visit the current node first, then recursively traverse the left subtree, then the right subtree. Useful for copying a tree or producing a prefix-style representation.
- **Inorder (Left → Root → Right):** Recursively traverse the left subtree, visit the current node, then recursively traverse the right subtree. For a Binary Search Tree, inorder traversal visits nodes in sorted (ascending) order.
- **Postorder (Left → Right → Root):** Recursively traverse the left subtree, then the right subtree, then visit the current node. Useful when children must be processed before the parent (e.g., deleting a tree, evaluating expression trees).
- **Level Order (Breadth-First Search):** Visit nodes level by level, from top to bottom and left to right within each level, using a queue instead of recursion/a stack.

All three DFS orders (preorder, inorder, postorder) visit every node exactly once and differ only in *when* the current node is processed relative to its children. Level order differs fundamentally — it processes nodes in "waves" (by depth) rather than diving deep first.

---

## 1. Preorder Traversal (Recursive and Iterative using a stack)

**Problem Statement:** Given the root of a binary tree, return the preorder traversal (Root → Left → Right) of its node values.

**Example:**
- Input: `[1, null, 2, 3]` (root = 1, right child = 2, 2's left child = 3)
- Output: `[1, 2, 3]`
- Explanation: Visit root `1` first, there's no left subtree, so move to right subtree rooted at `2`. Visit `2`, then its left child `3`. Result: `1, 2, 3`.

**Approach:**

*Recursive:* Visit the node's value, then recursively call on the left child, then recursively call on the right child. The call stack implicitly manages backtracking.

*Iterative (stack):* Use an explicit `Stack<TreeNode>`. Push the root. While the stack is not empty, pop a node, record its value, then push its right child first and its left child second (so the left child is popped and processed before the right child, preserving root-left-right order).

**Logic (Steps):**
1. *Recursive:* if `node == null`, return; otherwise record `node.val`, then recurse into `node.left`, then recurse into `node.right`.
2. *Iterative:* push `root` onto the stack (if non-null).
3. While the stack is non-empty, pop a node and record its value (this is the preorder visit).
4. Push the popped node's right child first, then its left child, so the left child is popped and processed next.
5. Repeat until the stack is empty; return the recorded values.

```csharp
// Recursive
public List<int> PreorderRecursive(TreeNode root) {
    List<int> result = new List<int>();
    PreorderHelper(root, result);
    return result;
}

private void PreorderHelper(TreeNode node, List<int> result) {
    if (node == null) return;
    result.Add(node.val);
    PreorderHelper(node.left, result);
    PreorderHelper(node.right, result);
}

// Iterative using a stack
public List<int> PreorderIterative(TreeNode root) {
    List<int> result = new List<int>();
    if (root == null) return result;

    Stack<TreeNode> stack = new Stack<TreeNode>();
    stack.Push(root);

    while (stack.Count > 0) {
        TreeNode node = stack.Pop();
        result.Add(node.val);

        // Push right first so left is processed first (LIFO order)
        if (node.right != null) stack.Push(node.right);
        if (node.left != null) stack.Push(node.left);
    }

    return result;
}
```

Time Complexity: O(n) — every node is visited and processed exactly once.
Space Complexity: O(h) for the recursion call stack / explicit stack in the average/best case (h = height of the tree), which degrades to O(n) for a skewed tree; ignoring the output list, the auxiliary stack never holds more than O(h) nodes on a balanced tree but can hold up to O(n) on a completely unbalanced (skewed) one.

**Walkthrough:** (Iterative dry run of inorder and the combined single-stack technique are covered in detail in problems 2 and 5 below; here we dry run preorder briefly.) Consider the tree `[1, null, 2, 3]`: `1` has no left child and right child `2`; `2` has left child `3`.

Stack starts as `[1]`.
1. Pop `1` → result = `[1]`. Push right (`2`), push left (`null`, skipped). Stack = `[2]`.
2. Pop `2` → result = `[1, 2]`. Push right (`null`, skipped), push left (`3`). Stack = `[3]`.
3. Pop `3` → result = `[1, 2, 3]`. No children. Stack = `[]`.

Final result: `[1, 2, 3]`, matching the expected output.

---

## 2. Inorder Traversal (Recursive and Iterative using a stack)

**Problem Statement:** Given the root of a binary tree, return the inorder traversal (Left → Root → Right) of its node values.

**Example:**
- Input: `[1, null, 2, 3]` (root = 1, right child = 2, 2's left child = 3)
- Output: `[1, 3, 2]`
- Explanation: `1` has no left subtree, so visit `1` first. Then traverse the right subtree rooted at `2`: within it, visit left child `3` first, then `2` itself. Result: `1, 3, 2`.

**Approach:**

*Recursive:* Recursively traverse the left subtree, visit the current node's value, then recursively traverse the right subtree.

*Iterative (stack):* Use a `Stack<TreeNode>` and a `current` pointer starting at the root. Repeatedly push nodes while moving left as far as possible. When you can't go further left, pop a node, record its value, then move to its right child and repeat. This mimics the recursive call stack: pushing simulates "descending" into the left recursive call, and popping simulates "returning" to process the node and then descend right.

**Logic (Steps):**
1. *Recursive:* if `node == null`, return; otherwise recurse into `node.left`, then record `node.val`, then recurse into `node.right`.
2. *Iterative:* start `current` at the root; loop while `current != null` or the stack is non-empty.
3. Push nodes while moving left (`current = current.left`) until `current` is `null`.
4. Pop the top of the stack, record its value (the inorder visit), then move `current = current.right` to resume the process on that subtree.
5. Repeat until both `current` is `null` and the stack is empty; return the recorded values.

```csharp
// Recursive
public List<int> InorderRecursive(TreeNode root) {
    List<int> result = new List<int>();
    InorderHelper(root, result);
    return result;
}

private void InorderHelper(TreeNode node, List<int> result) {
    if (node == null) return;
    InorderHelper(node.left, result);
    result.Add(node.val);
    InorderHelper(node.right, result);
}

// Iterative using a stack
public List<int> InorderIterative(TreeNode root) {
    List<int> result = new List<int>();
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode current = root;

    while (current != null || stack.Count > 0) {
        // Go as far left as possible, pushing nodes along the way
        while (current != null) {
            stack.Push(current);
            current = current.left;
        }

        // Backtrack: process the node, then move to its right subtree
        current = stack.Pop();
        result.Add(current.val);
        current = current.right;
    }

    return result;
}
```

Time Complexity: O(n) — each node is pushed and popped exactly once.
Space Complexity: O(h), where h is the height of the tree, since at most one root-to-leaf path of nodes sits on the stack at any time (O(log n) for a balanced tree, O(n) worst case for a skewed tree).

**Explanation:** Dry run on the tree `[1, null, 2, 3]` (`1` → right `2` → left `3`):

| Step | Action | Stack (bottom→top) | current | result |
|---|---|---|---|---|
| 1 | current = 1, push 1, current = 1.left = null | `[1]` | null | `[]` |
| 2 | inner loop ends (current is null); pop 1, add to result; current = 1.right = 2 | `[]` | 2 | `[1]` |
| 3 | current = 2, push 2, current = 2.left = 3 | `[2]` | 3 | `[1]` |
| 4 | current = 3, push 3, current = 3.left = null | `[2, 3]` | null | `[1]` |
| 5 | inner loop ends; pop 3, add to result; current = 3.right = null | `[2]` | null | `[1, 3]` |
| 6 | outer loop continues (stack not empty); inner loop does nothing since current is null; pop 2, add to result; current = 2.right = null | `[]` | null | `[1, 3, 2]` |
| 7 | current is null and stack is empty → loop ends | `[]` | null | `[1, 3, 2]` |

Final result: `[1, 3, 2]`.

---

## 3. Postorder Traversal (Recursive, Iterative using Two Stacks, and Iterative using One Stack)

**Problem Statement:** Given the root of a binary tree, return the postorder traversal (Left → Right → Root) of its node values.

**Example:**
- Input: `[1, null, 2, 3]` (root = 1, right child = 2, 2's left child = 3)
- Output: `[3, 2, 1]`
- Explanation: `1` has no left subtree, so descend into the right subtree rooted at `2`. Within that, visit left child `3` first, then `2` (no right child), and finally visit root `1` last. Result: `3, 2, 1`.

**Approach:**

*Recursive:* Recursively traverse the left subtree, recursively traverse the right subtree, then visit the current node's value last.

*Iterative — Two Stacks:* Push the root onto `stack1`. While `stack1` is not empty, pop a node, push it onto `stack2`, then push its left child and then its right child onto `stack1` (note: left before right here, the reverse of the preorder trick). This produces a "root-right-left" order on `stack1`'s pop sequence, which when reversed (by popping everything from `stack2` at the end) gives "left-right-root" — exactly postorder.

*Iterative — One Stack:* Use a single stack with a `lastVisited` pointer. Traverse left as far as possible (like inorder), pushing nodes. When you can't go left, peek at the top node: if it has no right child, or its right child was already visited (`lastVisited == node.right`), pop it, record its value, and update `lastVisited`. Otherwise, move to its right child and continue.

```csharp
// Recursive
public List<int> PostorderRecursive(TreeNode root) {
    List<int> result = new List<int>();
    PostorderHelper(root, result);
    return result;
}

private void PostorderHelper(TreeNode node, List<int> result) {
    if (node == null) return;
    PostorderHelper(node.left, result);
    PostorderHelper(node.right, result);
    result.Add(node.val);
}

// Iterative using Two Stacks
public List<int> PostorderTwoStacks(TreeNode root) {
    List<int> result = new List<int>();
    if (root == null) return result;

    Stack<TreeNode> stack1 = new Stack<TreeNode>();
    Stack<TreeNode> stack2 = new Stack<TreeNode>();
    stack1.Push(root);

    while (stack1.Count > 0) {
        TreeNode node = stack1.Pop();
        stack2.Push(node);

        if (node.left != null) stack1.Push(node.left);
        if (node.right != null) stack1.Push(node.right);
    }

    while (stack2.Count > 0) {
        result.Add(stack2.Pop().val);
    }

    return result;
}

// Iterative using One Stack
public List<int> PostorderOneStack(TreeNode root) {
    List<int> result = new List<int>();
    if (root == null) return result;

    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode current = root;
    TreeNode lastVisited = null;

    while (current != null || stack.Count > 0) {
        if (current != null) {
            stack.Push(current);
            current = current.left;
        } else {
            TreeNode peekNode = stack.Peek();
            // If right child exists and hasn't been processed yet, go right
            if (peekNode.right != null && lastVisited != peekNode.right) {
                current = peekNode.right;
            } else {
                result.Add(peekNode.val);
                lastVisited = stack.Pop();
            }
        }
    }

    return result;
}
```

Time Complexity: O(n) for all three approaches — every node is processed a constant number of times.
Space Complexity: O(h) for the recursive and one-stack approaches (auxiliary stack bounded by tree height). The two-stack approach uses O(n) space in the worst case since both stacks can collectively hold up to n nodes (in addition to the output list).

**Explanation:** Dry run of the one-stack postorder on `[1, null, 2, 3]` (`1` → right `2` → left `3`):

| Step | Action | Stack (bottom→top) | current | lastVisited | result |
|---|---|---|---|---|---|
| 1 | current = 1, push 1, current = 1.left = null | `[1]` | null | null | `[]` |
| 2 | current is null; peek = 1; 1.right = null → pop 1, add to result | `[]` | null | 1 | `[1]`... |

Wait — this would be wrong since `1` should be visited last. Let's redo carefully: `1.left` is `null`, but `1.right = 2` is **not** null, so at step 2 we must check `peekNode.right != null` first.

Corrected dry run:

| Step | Action | Stack (bottom→top) | current | lastVisited | result |
|---|---|---|---|---|---|
| 1 | current=1: push 1, current = 1.left = null | `[1]` | null | null | `[]` |
| 2 | current null; peek=1; 1.right=2, lastVisited(null)≠2 → current = 2 | `[1]` | 2 | null | `[]` |
| 3 | current=2: push 2, current = 2.left = 3 | `[1, 2]` | 3 | null | `[]` |
| 4 | current=3: push 3, current = 3.left = null | `[1, 2, 3]` | null | null | `[]` |
| 5 | current null; peek=3; 3.right=null → pop 3, add to result, lastVisited=3 | `[1, 2]` | null | 3 | `[3]` |
| 6 | current null; peek=2; 2.right=null → pop 2, add to result, lastVisited=2 | `[1]` | null | 2 | `[3, 2]` |
| 7 | current null; peek=1; 1.right=2, lastVisited(2)==2 → pop 1, add to result, lastVisited=1 | `[]` | null | 1 | `[3, 2, 1]` |
| 8 | current null, stack empty → loop ends | `[]` | null | 1 | `[3, 2, 1]` |

Final result: `[3, 2, 1]`. The key trick is `lastVisited`: it remembers the most recently popped node so that when we return to a parent whose right subtree we already fully processed, we know not to re-descend into it and instead pop the parent itself.

---

## 4. Level Order Traversal (BFS, return list of lists per level)

**Problem Statement:** Given the root of a binary tree, return the level order traversal of its node values, i.e., grouped level by level from left to right, as a list of lists.

**Example:**
- Input: `[1, null, 2, 3]` (root = 1, right child = 2, 2's left child = 3)
- Output: `[[1], [2], [3]]`
- Explanation: Level 0 contains just `1`. Level 1 contains `2` (the only child of `1`). Level 2 contains `3` (the left child of `2`).

**Approach:** Use a `Queue<TreeNode>` for BFS. Enqueue the root. On each iteration of the outer loop, capture the current queue size (`levelSize`) — this is exactly the number of nodes at the current level. Dequeue that many nodes, recording their values into a level list and enqueueing their non-null children for the next level. Repeat until the queue is empty.

```csharp
public List<List<int>> LevelOrder(TreeNode root) {
    List<List<int>> result = new List<List<int>>();
    if (root == null) return result;

    Queue<TreeNode> queue = new Queue<TreeNode>();
    queue.Enqueue(root);

    while (queue.Count > 0) {
        int levelSize = queue.Count;
        List<int> currentLevel = new List<int>();

        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.Dequeue();
            currentLevel.Add(node.val);

            if (node.left != null) queue.Enqueue(node.left);
            if (node.right != null) queue.Enqueue(node.right);
        }

        result.Add(currentLevel);
    }

    return result;
}
```

Time Complexity: O(n) — every node is enqueued and dequeued exactly once.
Space Complexity: O(n) for the queue in the worst case — a wide/complete tree can have up to roughly n/2 nodes on its widest level, all sitting in the queue simultaneously, which is O(n) rather than O(h) as with the DFS-based traversals.

**Explanation:** Dry run on `[1, null, 2, 3]`:

1. Queue = `[1]`. `levelSize = 1`. Dequeue `1` → currentLevel = `[1]`; enqueue right child `2` (left is null). Queue = `[2]`. result = `[[1]]`.
2. Queue = `[2]`. `levelSize = 1`. Dequeue `2` → currentLevel = `[2]`; enqueue left child `3` (right is null). Queue = `[3]`. result = `[[1], [2]]`.
3. Queue = `[3]`. `levelSize = 1`. Dequeue `3` → currentLevel = `[3]`; no children to enqueue. Queue = `[]`. result = `[[1], [2], [3]]`.
4. Queue is empty → loop ends.

Final result: `[[1], [2], [3]]`.

---

## 5. Preorder, Inorder, and Postorder Traversal in a Single Pass Using One Stack

**Problem Statement:** Given the root of a binary tree, compute the preorder, inorder, and postorder traversals simultaneously, visiting each node only once overall (i.e., in a single traversal pass), using one explicit stack.

**Example:**
- Input: `[1, null, 2, 3]` (root = 1, right child = 2, 2's left child = 3)
- Output: Preorder = `[1, 2, 3]`, Inorder = `[1, 3, 2]`, Postorder = `[3, 2, 1]`
- Explanation: A node passes through the stack up to three times: once when first pushed (preorder moment, before either child is explored), once after its left subtree is done (inorder moment, before the right child is explored), and once after its right subtree is done (postorder moment, ready to be removed).

**Approach:** Push `(node, state)` pairs onto the stack, where `state` tracks how many times this node has been "seen" (1, 2, or 3). Start by pushing `(root, 1)`.
- When popped with `state == 1`: this is the **preorder** moment — record the value into the preorder list, push the node back with `state = 2`, then push its left child (if any) with `state = 1`.
- When popped with `state == 2`: this is the **inorder** moment — record the value into the inorder list, push the node back with `state = 3`, then push its right child (if any) with `state = 1`.
- When popped with `state == 3`: this is the **postorder** moment — record the value into the postorder list, and do not push the node again (it's fully processed).

This state machine exactly mirrors what the recursive call structure does at three distinct points in a single recursive call: on entry (preorder), between the left and right recursive calls (inorder), and on exit after both recursive calls return (postorder).

```csharp
public (List<int> pre, List<int> inOrder, List<int> post) TraverseSinglePass(TreeNode root) {
    List<int> preorder = new List<int>();
    List<int> inorder = new List<int>();
    List<int> postorder = new List<int>();

    if (root == null) return (preorder, inorder, postorder);

    // Each stack entry is (node, state): state 1 = pre, 2 = in, 3 = post
    Stack<(TreeNode node, int state)> stack = new Stack<(TreeNode, int)>();
    stack.Push((root, 1));

    while (stack.Count > 0) {
        var (node, state) = stack.Pop();

        if (state == 1) {
            preorder.Add(node.val);
            stack.Push((node, 2));
            if (node.left != null) stack.Push((node.left, 1));
        } else if (state == 2) {
            inorder.Add(node.val);
            stack.Push((node, 3));
            if (node.right != null) stack.Push((node.right, 1));
        } else { // state == 3
            postorder.Add(node.val);
        }
    }

    return (preorder, inorder, postorder);
}
```

Time Complexity: O(n) — each node is pushed and popped exactly three times total (once per state), which is still a constant number of operations per node, so the overall work is linear in the number of nodes.
Space Complexity: O(h) — at any point the stack holds at most one entry per node along the current root-to-leaf path (each node contributes at most one "pending" entry above its ancestor's re-pushed entries), bounded by the tree's height h; worst case O(n) for a completely skewed tree.

**Explanation:** Dry run the single-stack combined technique on `[1, null, 2, 3]` (`1` → right `2` → left `3`, no other children):

| Step | Pop | State | Action | Push (top last) | pre | in | post |
|---|---|---|---|---|---|---|---|
| 1 | (1,1) | 1 | pre.Add(1); push (1,2); 1.left=null → nothing more | `[(1,2)]` | `[1]` | `[]` | `[]` |
| 2 | (1,2) | 2 | in.Add(1); push (1,3); push (2,1) since 1.right=2 | `[(1,3),(2,1)]` | `[1]` | `[1]` | `[]` |
| 3 | (2,1) | 1 | pre.Add(2); push (2,2); 2.left=3 → push (3,1) | `[(1,3),(2,2),(3,1)]` | `[1,2]` | `[1]` | `[]` |
| 4 | (3,1) | 1 | pre.Add(3); push (3,2); 3.left=null → nothing more | `[(1,3),(2,2),(3,2)]` | `[1,2,3]` | `[1]` | `[]` |
| 5 | (3,2) | 2 | in.Add(3); push (3,3); 3.right=null → nothing more | `[(1,3),(2,2),(3,3)]` | `[1,2,3]` | `[1,3]` | `[]` |
| 6 | (3,3) | 3 | post.Add(3); nothing pushed | `[(1,3),(2,2)]` | `[1,2,3]` | `[1,3]` | `[3]` |
| 7 | (2,2) | 2 | in.Add(2); push (2,3); 2.right=null → nothing more | `[(1,3),(2,3)]` | `[1,2,3]` | `[1,3,2]` | `[3]` |
| 8 | (2,3) | 3 | post.Add(2); nothing pushed | `[(1,3)]` | `[1,2,3]` | `[1,3,2]` | `[3,2]` |
| 9 | (1,3) | 3 | post.Add(1); nothing pushed | `[]` | `[1,2,3]` | `[1,3,2]` | `[3,2,1]` |

Final results: Preorder = `[1, 2, 3]`, Inorder = `[1, 3, 2]`, Postorder = `[3, 2, 1]` — matching the individually computed traversals from problems 1–3. The trick is that the `state` value tags *which* of the three "visits" to a node is currently happening, letting one stack and one loop reproduce all three orderings that the recursive calls would normally interleave across separate traversals.
