# Linked List — Singly Linked List Basics

## Node Definition

```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; this.next = null; }
}
```

### 1. Construct/Traverse a Singly Linked List

**Problem Statement:** Given a sequence of integers, build a singly linked list by inserting nodes at the head and/or tail, then traverse the list from the head to print all elements in order.

**Example:**
- Input: Insert `1, 2, 3, 4, 5` at the tail one by one (starting from an empty list)
- Output: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Explanation: Each new value is appended after the current last node, so the final list preserves insertion order.

**Approach:** Maintain a `head` pointer for the list.
- **Insert at head:** Create a new node, set its `next` to the current `head`, then update `head` to point to the new node.
- **Insert at tail:** If the list is empty, the new node becomes `head`. Otherwise, walk from `head` until `next` is `null` (the last node), then set that node's `next` to the new node.
- **Traverse/print:** Start a pointer `curr` at `head` and move `curr = curr.next` until `curr` is `null`, printing `curr.val` at each step.

```csharp
public class SinglyLinkedList {
    public ListNode head;

    // Insert a new node at the head of the list
    public void InsertAtHead(int val) {
        ListNode newNode = new ListNode(val);
        newNode.next = head;
        head = newNode;
    }

    // Insert a new node at the tail of the list
    public void InsertAtTail(int val) {
        ListNode newNode = new ListNode(val);
        if (head == null) {
            head = newNode;
            return;
        }
        ListNode curr = head;
        while (curr.next != null) {
            curr = curr.next;
        }
        curr.next = newNode;
    }

    // Traverse and print the list
    public void PrintList() {
        ListNode curr = head;
        while (curr != null) {
            Console.Write(curr.val + " -> ");
            curr = curr.next;
        }
        Console.WriteLine("NULL");
    }
}
```

**Time Complexity:** `InsertAtHead` is O(1); `InsertAtTail` and `PrintList` are O(n), where n is the number of nodes.
**Space Complexity:** O(1) extra space (excluding the space used by the newly created nodes).

---

### 2. Find the Length of a Linked List

**Problem Statement:** Given the head of a singly linked list, count and return the total number of nodes in the list.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Output: `5`
- Explanation: There are 5 nodes in the list, so the length is 5.

**Approach:** Use a counter and a pointer starting at `head`. Move the pointer one node at a time until it becomes `null`, incrementing the counter on each step.

```csharp
public class Solution {
    public int GetLength(ListNode head) {
        int count = 0;
        ListNode curr = head;
        while (curr != null) {
            count++;
            curr = curr.next;
        }
        return count;
    }
}
```

**Time Complexity:** O(n), where n is the number of nodes — every node is visited exactly once.
**Space Complexity:** O(1), only a counter and a pointer are used.

---

### 3. Search an Element in a Linked List

**Problem Statement:** Given the head of a singly linked list and a target value, determine whether the target exists in the list.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL, target = 4`
- Output: `true`
- Explanation: The value `4` is present at the fourth node, so the search returns `true`.

**Approach:** Traverse the list from `head`, comparing each node's `val` with the target. If a match is found, return `true` immediately. If the traversal reaches `null` without a match, return `false`.

```csharp
public class Solution {
    public bool Search(ListNode head, int target) {
        ListNode curr = head;
        while (curr != null) {
            if (curr.val == target) {
                return true;
            }
            curr = curr.next;
        }
        return false;
    }
}
```

**Time Complexity:** O(n) in the worst case, where n is the number of nodes.
**Space Complexity:** O(1), no extra data structures are used.

---

### 4. Insert a Node at a Given Position

**Problem Statement:** Given the head of a singly linked list, a value to insert, and a 1-based position, insert a new node containing that value at the given position. Position `1` means the new node becomes the new head.

**Example:**
- Input: `1 -> 2 -> 4 -> 5 -> NULL, value = 3, position = 3`
- Output: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`
- Explanation: The new node with value `3` is inserted so that it becomes the third node in the list.

**Approach:** If `position == 1`, insert at the head (new node's `next` points to old `head`). Otherwise, walk a pointer `prev` from `head` to the `(position - 1)`th node. Create the new node, point its `next` to `prev.next`, then point `prev.next` to the new node.

```csharp
public class Solution {
    public ListNode InsertAtPosition(ListNode head, int value, int position) {
        ListNode newNode = new ListNode(value);

        if (position == 1) {
            newNode.next = head;
            return newNode;
        }

        ListNode prev = head;
        for (int i = 1; i < position - 1 && prev != null; i++) {
            prev = prev.next;
        }

        if (prev == null) {
            throw new ArgumentOutOfRangeException(nameof(position), "Position is out of bounds.");
        }

        newNode.next = prev.next;
        prev.next = newNode;

        return head;
    }
}
```

**Time Complexity:** O(n) in the worst case, since we may need to walk up to `position - 1` nodes.
**Space Complexity:** O(1) extra space (excluding the new node created).

**Explanation (dry run of pointer rewiring):**
Starting list: `1 -> 2 -> 4 -> 5 -> NULL`, inserting value `3` at position `3`.
1. `position != 1`, so we walk `prev` starting at `head` (node `1`). We need to stop at the `(position - 1)`th = 2nd node.
   - `i = 1`: `prev = prev.next` → `prev` now points to node `2`. Loop condition `i < position - 1` → `1 < 2` is false after increment check, loop ends.
   - So `prev` now points to node `2` (the 2nd node).
2. Create `newNode` with `val = 3`.
3. `newNode.next = prev.next` → `newNode.next` now points to node `4` (previously `2.next`).
4. `prev.next = newNode` → node `2`'s `next` now points to `newNode` (value `3`) instead of node `4`.
5. Resulting chain: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`. The new node was spliced in between nodes `2` and `4` without touching any other links.

---

### 5. Delete a Node at a Given Position (including delete the head)

**Problem Statement:** Given the head of a singly linked list and a 1-based position, delete the node at that position and return the new head. Deleting position `1` means removing the head node itself.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5 -> NULL, position = 1`
- Output: `2 -> 3 -> 4 -> 5 -> NULL`
- Explanation: The node at position 1 (the head, value `1`) is removed, and node `2` becomes the new head.

**Approach:** If `position == 1`, simply move `head` to `head.next` (the old head is dropped). Otherwise, walk `prev` to the `(position - 1)`th node, then set `prev.next = prev.next.next`, which skips over (and effectively deletes) the target node.

```csharp
public class Solution {
    public ListNode DeleteAtPosition(ListNode head, int position) {
        if (head == null) {
            return null;
        }

        if (position == 1) {
            return head.next;
        }

        ListNode prev = head;
        for (int i = 1; i < position - 1 && prev != null; i++) {
            prev = prev.next;
        }

        if (prev == null || prev.next == null) {
            throw new ArgumentOutOfRangeException(nameof(position), "Position is out of bounds.");
        }

        prev.next = prev.next.next;

        return head;
    }
}
```

**Time Complexity:** O(n) in the worst case, since we may need to walk up to `position - 1` nodes.
**Space Complexity:** O(1), no extra space is used.

**Explanation (dry run of pointer rewiring):**
Starting list: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`, deleting position `3` (value `3`).
1. `position != 1`, so we walk `prev` starting at `head` (node `1`) to the `(position - 1)`th = 2nd node.
   - `i = 1`: `prev = prev.next` → `prev` now points to node `2`. Loop ends since `1 < 2` fails after this step.
2. `prev` now points to node `2`, and `prev.next` currently points to node `3` (the node to delete).
3. `prev.next = prev.next.next` → this reads node `3`'s `next` (which is node `4`) and assigns it directly to node `2`'s `next`. Now node `2`'s `next` points to node `4`, completely skipping node `3`.
4. Node `3` is no longer reachable from `head` (it still exists in memory momentarily but has no incoming reference), so it is effectively deleted from the list. In C#, the garbage collector reclaims it automatically.
5. Resulting chain: `1 -> 2 -> 4 -> 5 -> NULL`.

For deleting the head (position `1`) on `1 -> 2 -> 3 -> 4 -> 5 -> NULL`:
1. We simply return `head.next`, which is node `2`.
2. The caller's `head` reference is updated to node `2`. Node `1` is no longer referenced by the list and is reclaimed by garbage collection.
3. Resulting chain: `2 -> 3 -> 4 -> 5 -> NULL`.

---

### 6. Delete the Given Node (when only access to that node is provided, not the head)

**Problem Statement:** Given only a reference to a node (guaranteed not to be the last node) in a singly linked list — without access to the list's head — delete that node from the list.

**Example:**
- Input: List is `1 -> 2 -> 3 -> 4 -> 5 -> NULL`, and the given node reference is the node with value `3` (head is not accessible)
- Output: `1 -> 2 -> 4 -> 5 -> NULL`
- Explanation: Since we cannot reach `node.next.next` from the previous node (no `prev` pointer available and no `head`), we instead "become" the next node by copying its value and then bypass it.

**Approach:** Since we have no access to the previous node, we cannot simply unlink the given node in the usual way. Instead, we use a trick: copy the value of the *next* node into the current (given) node, then set the current node's `next` to skip over the next node (i.e., `node.next = node.next.next`). Effectively, the given node "becomes" a copy of the next node in value, and the actual next node is removed from the chain. This works because the node's identity in memory doesn't matter to the caller — only the sequence of values matters, and the node object corresponding to the last duplicate is discarded.

```csharp
public class Solution {
    public void DeleteGivenNode(ListNode node) {
        // Precondition: node is not null and node.next is not null (not the last node)
        node.val = node.next.val;
        node.next = node.next.next;
    }
}
```

**Time Complexity:** O(1), only a constant number of pointer/value operations are performed.
**Space Complexity:** O(1), no extra space is used.

**Explanation (dry run of pointer rewiring and value-copy trick):**
List: `1 -> 2 -> 3 -> 4 -> 5 -> NULL`. We are given a direct reference to the node with value `3` (call it `node`), but we have no `prev` pointer and no `head`.

The problem: normally, deleting `node` requires setting `prev.next = node.next`, but we cannot reach `prev` since the list is singly linked and we only have `node`.

The trick: instead of removing `node` itself, we overwrite `node`'s data with the next node's data, then remove the next node instead (which we *can* reach, via `node.next`).

1. `node` currently has `val = 3` and `node.next` points to the node with `val = 4`.
2. `node.val = node.next.val` → copy `4` into `node.val`. Now `node.val = 4`. At this point the list conceptually reads `1 -> 2 -> 4 -> 4 -> 5 -> NULL` (a duplicate `4` exists — the original `node`, now holding value `4`, and the real node with value `4` right after it).
3. `node.next = node.next.next` → `node.next` currently points to the real node with value `4`; we read *its* `next` (the node with value `5`) and assign it directly to `node.next`. Now `node.next` points to the node with value `5`, skipping over (and dropping the reference to) the real node that held value `4`.
4. The real node that originally held value `4` is now unreachable from the list (no node points to it anymore), so it is effectively deleted and later garbage collected.
5. Resulting chain, read from the original head (which the caller still holds elsewhere): `1 -> 2 -> 4 -> 5 -> NULL`. The node object that used to represent value `3` now represents value `4`, and the duplicate has been removed — so from the outside, the list looks exactly as if the node with value `3` was deleted.

**Note:** This trick fails if `node` is the last node in the list, since `node.next` would be `null` and there would be no next node's value to copy — hence the precondition that the given node is guaranteed not to be the last node.
