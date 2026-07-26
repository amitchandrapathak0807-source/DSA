# Binary Search Trees — Basics

Node definition used throughout this document:

```csharp
public class TreeNode {
    public int val;
    public TreeNode left;
    public TreeNode right;
    public TreeNode(int val) { this.val = val; left = null; right = null; }
}
```

## Concept: Binary Search Tree Property

A **Binary Search Tree (BST)** is a binary tree that maintains the following invariant at *every* node:

- All values in the **left subtree** of a node are **strictly less than** the node's value.
- All values in the **right subtree** of a node are **strictly greater than** the node's value.
- This rule applies **recursively** — the left and right subtrees are themselves valid BSTs, not just the immediate children.

So for any node `x`:

```
max(left subtree of x) < x.val < min(right subtree of x)
```

An important consequence: an **inorder traversal (left, node, right) of a BST always yields values in sorted (ascending) order**. This property is heavily exploited in problems like finding the Kth smallest element, checking BST validity, and converting a BST to a sorted array.

**Why this enables O(h) operations instead of O(n):**

In a plain binary tree (no ordering guarantee), searching for a value requires checking every node in the worst case — O(n), because there's no information to tell you which branch might contain the target.

In a BST, at every node you compare the target with the node's value:
- If `target == node.val` → found it.
- If `target < node.val` → the target (if it exists) *must* be in the left subtree — the entire right subtree can be discarded.
- If `target > node.val` → the target (if it exists) *must* be in the right subtree — the entire left subtree can be discarded.

This is exactly the same idea as **binary search on a sorted array**: at each step, you eliminate half of the remaining search space (one whole subtree) without even looking at it. Instead of visiting O(n) nodes, you only ever walk a single root-to-leaf path, so the cost is proportional to the **height `h`** of the tree, not the number of nodes.

- If the tree is **balanced** (height-balanced, like an AVL tree or a randomly-built BST), `h = O(log n)`, giving very fast O(log n) search, insert, and delete.
- If the tree is **skewed** (e.g., values inserted in strictly increasing order, degenerating into a linked list), `h = O(n)` in the worst case, so operations degrade to O(n).

This is why self-balancing BSTs (AVL, Red-Black trees) exist — they guarantee `h = O(log n)` by rebalancing after insertions/deletions. Plain BSTs, as covered here, do not make that guarantee.

---

## 1. Search a Given Value in a BST

**Problem Statement:** Given the root of a BST and an integer `target`, return the subtree rooted at the node whose value equals `target`. If no such node exists, return `null`.

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]` (root 8, left child 3, right child 10, 3's children 1 and 6, 10's right child 14), target = `6`
- Output: The subtree rooted at node `6` (a leaf node with `val = 6`)

**Approach:** Use the BST property to decide direction at each node instead of exploring both subtrees. Start at the root:
- If `target == node.val`, return `node`.
- If `target < node.val`, recurse (or iterate) into `node.left` — the right subtree can never contain `target`.
- If `target > node.val`, recurse (or iterate) into `node.right` — the left subtree can never contain `target`.
- If you reach `null`, the value doesn't exist — return `null`.

This is a direct application of binary search: each comparison eliminates one entire subtree.

```csharp
public TreeNode SearchBST(TreeNode root, int target) {
    TreeNode current = root;
    while (current != null && current.val != target) {
        current = target < current.val ? current.left : current.right;
    }
    return current;
}
```

Time Complexity: O(h) where h is the height of the tree — O(log n) for a balanced BST, O(n) worst-case for a skewed BST. Space Complexity: O(1) for the iterative version shown (O(h) if implemented recursively, due to the call stack).

**Explanation:** Dry run on the example BST with `target = 6`:
1. `current = 8` (root). `6 < 8` → go left. `current = 3`.
2. `current = 3`. `6 > 3` → go right. `current = 6`.
3. `current = 6`. `6 == 6` → match found, return this node.

Only 3 nodes were visited out of 6 total nodes, and no comparison ever needed to look at node `10` or `14` — the entire right subtree of the root was discarded after the very first comparison.

---

## 2. Find the Minimum and Maximum Value in a BST

**Problem Statement:** Given the root of a non-empty BST, find the minimum and maximum values stored in it.

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]`
- Output: Minimum = `1`, Maximum = `14`

**Approach:** Because of the BST invariant, the **minimum value is always the leftmost node** (keep following `.left` until it's `null`), and the **maximum value is always the rightmost node** (keep following `.right` until it's `null`). There is no need to compare values at all — the structure guarantees the answer's location, so you simply walk down one path.

```csharp
public int FindMin(TreeNode root) {
    if (root == null) throw new ArgumentException("Tree is empty");
    TreeNode current = root;
    while (current.left != null) {
        current = current.left;
    }
    return current.val;
}

public int FindMax(TreeNode root) {
    if (root == null) throw new ArgumentException("Tree is empty");
    TreeNode current = root;
    while (current.right != null) {
        current = current.right;
    }
    return current.val;
}
```

Time Complexity: O(h) where h is the height — O(log n) balanced, O(n) worst-case skewed. Space Complexity: O(1) iterative.

**Explanation:** For minimum: start at `8` → go left to `3` → go left to `1` → `1.left` is `null`, so `1` is the minimum. For maximum: start at `8` → go right to `10` → go right to `14` → `14.right` is `null`, so `14` is the maximum. Each walk only touches the nodes along a single path, never branching into the other subtree.

---

## 3. Find the Ceil of a Given Value in a BST

**Problem Statement:** Given the root of a BST and an integer `target`, find the **ceil** of `target` in the BST — the smallest value in the tree that is `>= target`. If no such value exists, return `-1` (or an appropriate sentinel).

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]`, target = `7`
- Output: `8` (smallest value in the tree that is `>= 7`)

**Approach:** Walk down from the root using BST comparisons, keeping track of the best (smallest-so-far) candidate `>= target` seen along the way.
- If `node.val == target`, that's the exact ceil — return immediately.
- If `node.val < target`, this node is too small to be a ceil; the ceil (if any) must be in the right subtree — move right without recording this node.
- If `node.val > target`, this node is a *candidate* ceil (it's `>= target`); record it, then try to find something smaller by moving left — a smaller valid ceil might exist there.

```csharp
public int FindCeil(TreeNode root, int target) {
    int ceil = -1;
    TreeNode current = root;
    while (current != null) {
        if (current.val == target) {
            return current.val;
        } else if (current.val < target) {
            current = current.right;
        } else {
            ceil = current.val;
            current = current.left;
        }
    }
    return ceil;
}
```

Time Complexity: O(h) where h is the height — O(log n) balanced, O(n) worst-case skewed. Space Complexity: O(1) iterative.

**Explanation:** Dry run with `target = 7` on `[8, 3, 10, 1, 6, null, 14]`:
1. `current = 8`. `8 > 7` → candidate `ceil = 8`, move left to `3`.
2. `current = 3`. `3 < 7` → too small, move right to `6`.
3. `current = 6`. `6 < 7` → too small, move right to `null`.
4. Loop ends. Return `ceil = 8`.

Each comparison either tightens the candidate or eliminates a subtree that provably cannot contain a better answer.

---

## 4. Find the Floor of a Given Value in a BST

**Problem Statement:** Given the root of a BST and an integer `target`, find the **floor** of `target` in the BST — the largest value in the tree that is `<= target`. If no such value exists, return `-1` (or an appropriate sentinel).

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]`, target = `7`
- Output: `6` (largest value in the tree that is `<= 7`)

**Approach:** Symmetric to the ceil approach. Walk down from the root, keeping track of the best (largest-so-far) candidate `<= target`.
- If `node.val == target`, that's the exact floor — return immediately.
- If `node.val > target`, this node is too large to be a floor; move left without recording it.
- If `node.val < target`, this node is a *candidate* floor (it's `<= target`); record it, then try to find something larger by moving right.

```csharp
public int FindFloor(TreeNode root, int target) {
    int floor = -1;
    TreeNode current = root;
    while (current != null) {
        if (current.val == target) {
            return current.val;
        } else if (current.val > target) {
            current = current.left;
        } else {
            floor = current.val;
            current = current.right;
        }
    }
    return floor;
}
```

Time Complexity: O(h) where h is the height — O(log n) balanced, O(n) worst-case skewed. Space Complexity: O(1) iterative.

**Explanation:** Dry run with `target = 7` on `[8, 3, 10, 1, 6, null, 14]`:
1. `current = 8`. `8 > 7` → too large, move left to `3`.
2. `current = 3`. `3 < 7` → candidate `floor = 3`, move right to `6`.
3. `current = 6`. `6 < 7` → candidate `floor = 6`, move right to `null`.
4. Loop ends. Return `floor = 6`.

---

## 5. Insert a Given Node into a BST

**Problem Statement:** Given the root of a BST and a value `val`, insert `val` into the BST such that the resulting tree is still a valid BST. Return the root of the tree after insertion. (Assume `val` does not already exist, or that duplicates are simply not re-inserted.)

**Example:**
- Input: BST `[8, 3, 10, 1, 6]`, val = `7`
- Output: BST `[8, 3, 10, 1, 6, null, null, null, null, null, 7]` — `7` is inserted as the right child of `6`, since `7 > 3` (go right... actually `7 < 8` go left, `7 > 3` go right, `7 > 6` go right) → becomes `6`'s right child.

**Approach:** New values are always inserted as a **leaf**. Walk down from the root exactly like `SearchBST`, comparing `val` against each node to decide left or right, until you fall off the tree (reach `null`) — that `null` spot is exactly where the new node belongs, since the BST property guarantees no ordering violation results from placing it there. Attach the new node at that position.

```csharp
public TreeNode InsertIntoBST(TreeNode root, int val) {
    if (root == null) {
        return new TreeNode(val);
    }

    TreeNode current = root;
    while (true) {
        if (val < current.val) {
            if (current.left == null) {
                current.left = new TreeNode(val);
                break;
            }
            current = current.left;
        } else {
            if (current.right == null) {
                current.right = new TreeNode(val);
                break;
            }
            current = current.right;
        }
    }
    return root;
}
```

Time Complexity: O(h) where h is the height — O(log n) balanced, O(n) worst-case skewed. Space Complexity: O(1) iterative (O(h) if done recursively).

**Explanation:** Dry run inserting `7` into `[8, 3, 10, 1, 6]`:
1. `current = 8`. `7 < 8` → go left; `8.left = 3` is not null, so `current = 3`.
2. `current = 3`. `7 > 3` → go right; `3.right = 6` is not null, so `current = 6`.
3. `current = 6`. `7 > 6` → go right; `6.right` is `null` → attach `new TreeNode(7)` as `6.right`, done.

The BST property is preserved automatically: `7` sits between `6` and `8` in sorted order, and its placement as `6`'s right child (deep inside `3`'s subtree, which is `8`'s left subtree) is consistent with `7 < 8` and `7 > 3` and `7 > 6`.

---

## 6. Delete a Node from a BST

**Problem Statement:** Given the root of a BST and a value `key`, delete the node with value `key` from the BST, preserving the BST property, and return the (possibly new) root.

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]`, key = `3` (has two children: `1` and `6`)
- Output: BST with `3` removed and replaced appropriately, e.g. `[8, 6, 10, 1, null, null, 14]` (`6`, the inorder successor of `3`, takes `3`'s place)

**Approach:** First locate the node to delete using standard BST search (go left if `key < node.val`, right if `key > node.val`). Once found, there are **three cases** based on how many children the node has:

1. **Leaf node (no children):** Simply remove it — replace it with `null` in its parent.
2. **One child (only left or only right):** Replace the node with its single child — the child subtree already satisfies the BST property relative to the parent, so it can be spliced in directly.
3. **Two children:** You cannot just delete the node, since two subtrees would need to be reattached. Instead, find a replacement value that preserves ordering: the **inorder successor** (the smallest value in the right subtree, i.e., the leftmost node of the right subtree) — or equivalently the inorder predecessor (largest value in the left subtree). Copy that successor's value into the current node, then recursively delete the successor from the right subtree (where it is guaranteed to be a leaf or have only a right child, making that deletion simpler).

```csharp
public TreeNode DeleteNode(TreeNode root, int key) {
    if (root == null) return null;

    if (key < root.val) {
        root.left = DeleteNode(root.left, key);
    } else if (key > root.val) {
        root.right = DeleteNode(root.right, key);
    } else {
        // Found the node to delete.

        // Case 1: leaf node, or Case 2: only right child exists.
        if (root.left == null) {
            return root.right;
        }
        // Case 2: only left child exists.
        if (root.right == null) {
            return root.left;
        }

        // Case 3: two children — find inorder successor (min of right subtree).
        TreeNode successor = root.right;
        while (successor.left != null) {
            successor = successor.left;
        }

        // Copy successor's value into current node.
        root.val = successor.val;

        // Delete the successor from the right subtree.
        root.right = DeleteNode(root.right, successor.val);
    }
    return root;
}
```

Time Complexity: O(h) where h is the height — O(log n) balanced, O(n) worst-case skewed (finding the node plus finding/removing the successor are both bounded by a root-to-leaf path). Space Complexity: O(h) recursion stack (can be rewritten iteratively for O(1) auxiliary space).

**Explanation:** Dry run deleting `key = 3` from `[8, 3, 10, 1, 6, null, 14]` (node `3` has two children: `1` on the left, `6` on the right):

1. Start at `8`: `3 < 8` → recurse left into `3`'s subtree.
2. At `3`: `3 == 3` → this is the node to delete. It has both `left = 1` and `right = 6`, so this is the **two-children case**.
3. Find the inorder successor: start at `root.right = 6`, and go left as far as possible. `6.left` is `null`, so `6` itself is the leftmost node of the right subtree → successor `= 6`.
4. Copy the successor's value into the current node: node originally holding `3` now holds `val = 6`.
5. Recursively delete `6` from the right subtree (`root.right`, which was the subtree rooted at `6`). Since `6` has no children here, this recursive call hits the leaf case and returns `null`, so `root.right` becomes `null`.
6. Final structure at that position: a node with `val = 6`, `left = 1` (unchanged), `right = null` (successor removed from its original spot).

Result: the tree now reads `8 → (6 → (1, null), 10 → (null, 14))`. The BST property still holds: `1 < 6 < 8`, and `6`'s old position is empty since its value was "moved" into the deleted node's slot.

---

## 7. Find the Kth Smallest/Largest Element in a BST

**Problem Statement:** Given the root of a BST and an integer `k`, find the `k`-th smallest element (1-indexed) in the BST. (The `k`-th largest is symmetric — either do a reversed inorder traversal (right, node, left), or find the `(n - k + 1)`-th smallest.)

**Example:**
- Input: BST `[8, 3, 10, 1, 6, null, 14]`, k = `3`
- Output: `6` (sorted order of all values is `1, 3, 6, 8, 10, 14`; the 3rd smallest is `6`)

**Approach:** Recall from the BST property that an **inorder traversal (left, node, right) visits nodes in strictly ascending sorted order**. So the k-th smallest element is simply the k-th node visited during an inorder traversal. Rather than generating the entire sorted list (which costs extra space and doesn't stop early), use an **iterative inorder traversal with an explicit stack**, maintaining a counter that increments each time a node is "visited" (popped and processed) — as soon as the counter reaches `k`, stop immediately and return that node's value, without traversing the rest of the tree. For k-th largest, do the mirror traversal (right, node, left), which visits nodes in descending order.

```csharp
public int KthSmallest(TreeNode root, int k) {
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode current = root;
    int count = 0;

    while (current != null || stack.Count > 0) {
        // Go as far left as possible, pushing nodes along the way.
        while (current != null) {
            stack.Push(current);
            current = current.left;
        }

        // Visit the node.
        current = stack.Pop();
        count++;
        if (count == k) {
            return current.val;
        }

        // Move to the right subtree.
        current = current.right;
    }

    throw new ArgumentException("k is out of range for this BST");
}

public int KthLargest(TreeNode root, int k) {
    Stack<TreeNode> stack = new Stack<TreeNode>();
    TreeNode current = root;
    int count = 0;

    while (current != null || stack.Count > 0) {
        // Go as far right as possible, pushing nodes along the way.
        while (current != null) {
            stack.Push(current);
            current = current.right;
        }

        // Visit the node.
        current = stack.Pop();
        count++;
        if (count == k) {
            return current.val;
        }

        // Move to the left subtree.
        current = current.left;
    }

    throw new ArgumentException("k is out of range for this BST");
}
```

Time Complexity: O(h + k) in the general case (at most O(n) if k is close to n) — since only the path down plus k pops are processed before stopping early; the worst case relates to n rather than a clean O(h), but the early stop makes it far cheaper than a full O(n) traversal for small k. Space Complexity: O(h) for the stack — O(log n) balanced, O(n) worst-case skewed.

**Explanation:** Dry run `KthSmallest` with `k = 3` on `[8, 3, 10, 1, 6, null, 14]`:

1. `current = 8`, stack empty. Push `8`, move to `8.left = 3`. Push `3`, move to `3.left = 1`. Push `1`, move to `1.left = null`. Stack: `[8, 3, 1]` (top is `1`).
2. Inner while ends (`current == null`). Pop `1`. `count = 1`. Not `k` yet. Move to `1.right = null`, so `current = null`.
3. Outer loop again: `current` is `null` but stack has `[8, 3]`. Inner while does nothing (current already null). Pop `3`. `count = 2`. Not `k` yet. Move to `3.right = 6`, so `current = 6`.
4. Inner while: push `6`, move to `6.left = null`. Stack: `[8, 6]`.
5. Pop `6`. `count = 3` → matches `k = 3`. Return `6.val = 6`.

The traversal stopped after visiting only `1, 3, 6` — it never needed to touch `8, 10, 14`, demonstrating the early-stop benefit of the iterative approach combined with the guaranteed sorted order that the BST's inorder traversal provides.
