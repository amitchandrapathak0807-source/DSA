# Linked List — Doubly Linked List Basics

## Node Definition

```csharp
public class DLLNode {
    public int val;
    public DLLNode next;
    public DLLNode prev;
    public DLLNode(int val) { this.val = val; this.next = null; this.prev = null; }
}
```

A doubly linked list (DLL) node carries two pointers instead of one: `next` links forward to the successor and `prev` links backward to the predecessor. This lets us traverse in both directions and delete a node in O(1) once we have a reference to it (no need to find the predecessor by walking from the head, unlike a singly linked list).

---

## 1. Construct/Traverse a Doubly Linked List

**Problem Statement:**
Build a doubly linked list by supporting insertion at the head and at the tail, then print the list both forward (head to tail) and backward (tail to head) to verify that `next` and `prev` pointers are consistent.

**Example:**
- Input: Insert `10` at tail, `20` at tail, `30` at tail, then insert `5` at head.
- Output: Forward: `5 <-> 10 <-> 20 <-> 30`, Backward: `30 <-> 20 <-> 10 <-> 5`
- Explanation: Each tail insertion appends after the current last node; the head insertion prepends before the current first node. Printing backward from the tail should be the exact reverse of the forward traversal, confirming `prev` pointers were wired correctly.

**Approach:**
Maintain `head` and `tail` references.
- `InsertAtHead`: create a new node, point its `next` to the current head, point the current head's `prev` to the new node, then make the new node the head. If the list was empty, the new node becomes both head and tail.
- `InsertAtTail`: symmetric — create a new node, point its `prev` to the current tail, point the current tail's `next` to the new node, then make the new node the tail.
- `PrintForward`: start at `head`, follow `next` until null.
- `PrintBackward`: start at `tail`, follow `prev` until null.

```csharp
public class DoublyLinkedList {
    public DLLNode head;
    public DLLNode tail;

    public void InsertAtHead(int val) {
        DLLNode node = new DLLNode(val);
        if (head == null) {
            head = tail = node;
            return;
        }
        node.next = head;
        head.prev = node;
        head = node;
    }

    public void InsertAtTail(int val) {
        DLLNode node = new DLLNode(val);
        if (tail == null) {
            head = tail = node;
            return;
        }
        node.prev = tail;
        tail.next = node;
        tail = node;
    }

    public List<int> PrintForward() {
        List<int> result = new List<int>();
        DLLNode curr = head;
        while (curr != null) {
            result.Add(curr.val);
            curr = curr.next;
        }
        return result;
    }

    public List<int> PrintBackward() {
        List<int> result = new List<int>();
        DLLNode curr = tail;
        while (curr != null) {
            result.Add(curr.val);
            curr = curr.prev;
        }
        return result;
    }
}
```

Time Complexity: O(1) per insertion at head/tail, O(n) per full traversal.
Space Complexity: O(1) extra space for insertion/traversal (O(n) to store the output list if collected).

**Explanation:**
Starting from an empty list, `InsertAtTail(10)` sets `head = tail = Node(10)`. `InsertAtTail(20)` links `10.next = 20` and `20.prev = 10`, then `tail = 20`. `InsertAtTail(30)` links `20.next = 30`, `30.prev = 20`, `tail = 30`. Now `InsertAtHead(5)` links `5.next = 10`, `10.prev = 5`, then `head = 5`. The list is `5 <-> 10 <-> 20 <-> 30`. Forward traversal from `head` follows `next`: `5 -> 10 -> 20 -> 30`. Backward traversal from `tail` follows `prev`: `30 -> 20 -> 10 -> 5`, exactly the reverse, confirming every `next`/`prev` pair was set consistently.

---

## 2. Insert a Node in a Doubly Linked List at a Given Position

**Problem Statement:**
Given a doubly linked list and a 0-indexed position `pos`, insert a new node with a given value so that it becomes the node at index `pos` after insertion (i.e., insert before the node currently at that position). Handle insertion at the head, in the middle, and at the tail.

**Example:**
- Input: `1 <-> 2 <-> 4 <-> 5`, insert value `3` at position `2`
- Output: `1 <-> 2 <-> 3 <-> 4 <-> 5`
- Explanation: The node currently at index 2 is `4`. The new node `3` is inserted right before it, becoming the new index-2 node, and `4`, `5` shift one position later.

**Approach:**
Walk from `head` to find the node currently occupying `pos` (call it `curr`). The new node must be spliced in between `curr.prev` (call it `prevNode`) and `curr`. Update four links: `newNode.prev = prevNode`, `newNode.next = curr`, `curr.prev = newNode`, and if `prevNode` exists, `prevNode.next = newNode`, otherwise the new node becomes the new `head`. Special-case `pos == 0` (insert at head) and inserting past the end (append at tail) as needed.

```csharp
public void InsertAtPosition(int pos, int val) {
    DLLNode node = new DLLNode(val);

    // Insert at head
    if (pos == 0 || head == null) {
        node.next = head;
        if (head != null) head.prev = node;
        head = node;
        if (tail == null) tail = node;
        return;
    }

    // Find node currently at 'pos' (or fall off the end -> insert at tail)
    DLLNode curr = head;
    int index = 0;
    while (curr != null && index < pos) {
        curr = curr.next;
        index++;
    }

    if (curr == null) {
        // pos is at/after the end: insert at tail
        node.prev = tail;
        if (tail != null) tail.next = node;
        tail = node;
        if (head == null) head = node;
        return;
    }

    DLLNode prevNode = curr.prev;
    node.prev = prevNode;
    node.next = curr;
    curr.prev = node;
    prevNode.next = node;
}
```

Time Complexity: O(n) to walk to the position, O(1) for the pointer rewiring.
Space Complexity: O(1).

**Explanation:**
List: `1 <-> 2 <-> 4 <-> 5`, insert `3` at `pos = 2`. Walking from `head`, `index 0 = node(1)`, `index 1 = node(2)`, `index 2 = node(4)` — loop stops with `curr = node(4)`. `prevNode = curr.prev = node(2)`. Rewire: `node(3).prev = node(2)`, `node(3).next = node(4)`, `node(4).prev = node(3)`, `node(2).next = node(3)`. Final chain: `1 <-> 2 <-> 3 <-> 4 <-> 5`, and both directions are consistent since all four pointers touching the splice point were updated.

---

## 3. Delete a Node from a Given Position in a Doubly Linked List

**Problem Statement:**
Given a doubly linked list and a 0-indexed position `pos`, remove the node at that position and reconnect its neighbors. Handle deleting the head, a middle node, and the tail, including the case where the list becomes empty.

**Example:**
- Input: `1 <-> 2 <-> 3 <-> 4 <-> 5`, delete position `2`
- Output: `1 <-> 2 <-> 4 <-> 5`
- Explanation: The node at index 2 (`3`) is removed. Its predecessor (`2`) and successor (`4`) are linked directly to each other in both directions.

**Approach:**
Locate the node `curr` at `pos`. Let `prevNode = curr.prev` and `nextNode = curr.next`. If `prevNode` exists, set `prevNode.next = nextNode`, otherwise `curr` was the head so `head = nextNode`. If `nextNode` exists, set `nextNode.prev = prevNode`, otherwise `curr` was the tail so `tail = prevNode`. Finally detach `curr` by clearing its own `next`/`prev` (good hygiene, helps GC and avoids stale references).

```csharp
public void DeleteAtPosition(int pos) {
    if (head == null) return;

    DLLNode curr = head;
    int index = 0;
    while (curr != null && index < pos) {
        curr = curr.next;
        index++;
    }
    if (curr == null) return; // position out of range

    DLLNode prevNode = curr.prev;
    DLLNode nextNode = curr.next;

    if (prevNode != null) prevNode.next = nextNode;
    else head = nextNode; // curr was head

    if (nextNode != null) nextNode.prev = prevNode;
    else tail = prevNode; // curr was tail

    curr.next = null;
    curr.prev = null;
}
```

Time Complexity: O(n) to reach the position, O(1) for the actual unlink.
Space Complexity: O(1).

**Explanation:**
List: `1 <-> 2 <-> 3 <-> 4 <-> 5`, delete `pos = 2`. Walking gives `curr = node(3)`, `prevNode = node(2)`, `nextNode = node(4)`. Since `prevNode` exists, `node(2).next = node(4)`. Since `nextNode` exists, `node(4).prev = node(2)`. `node(3)` is now unreachable from either direction; its own pointers are cleared. Result: `1 <-> 2 <-> 4 <-> 5`, consistent forward and backward. If instead `pos = 0` (deleting `1`), `prevNode` would be null, so `head` is updated directly to `nextNode`; symmetric logic handles deleting the tail.

---

## 4. Reverse a Doubly Linked List

**Problem Statement:**
Reverse a doubly linked list in place so that the original tail becomes the new head and the original head becomes the new tail, without allocating new nodes.

**Example:**
- Input: `1 <-> 2 <-> 3 <-> 4 <-> 5`
- Output: `5 <-> 4 <-> 3 <-> 2 <-> 1`
- Explanation: Every node's `next` and `prev` are swapped, and the list's `head`/`tail` references are swapped, so traversal direction is flipped end to end.

**Approach:**
Walk through every node once. For each node, swap its `next` and `prev` pointers (what used to be "forward" is now "backward" and vice versa). Keep a temporary `prev` reference to know where to move next, since `curr.next` is being overwritten — after swapping, the node that should be visited next is now sitting in `curr.prev` (the pre-swap `next`). After the loop, swap the list's `head` and `tail` references.

```csharp
public void Reverse() {
    DLLNode curr = head;
    DLLNode temp = null;

    while (curr != null) {
        temp = curr.prev;
        curr.prev = curr.next;
        curr.next = temp;
        curr = curr.prev; // this is the old 'next', due to swap above
    }

    // swap head and tail
    temp = head;
    head = tail;
    tail = temp;
}
```

Time Complexity: O(n) — every node visited once.
Space Complexity: O(1) — only a constant number of temporary references used.

**Explanation:**
List: `1 <-> 2 <-> 3`, with `head = 1`, `tail = 3`.
- `curr = 1`: `temp = 1.prev = null`; `1.prev = 1.next = 2`; `1.next = temp = null`. Now node 1 has `prev = 2, next = null`. Move `curr = 1.prev = 2`.
- `curr = 2`: `temp = 2.prev = 1` (unchanged so far); `2.prev = 2.next = 3`; `2.next = temp = 1`. Now node 2 has `prev = 3, next = 1`. Move `curr = 2.prev = 3`.
- `curr = 3`: `temp = 3.prev = 2` (unchanged); `3.prev = 3.next = null`; `3.next = temp = 2`. Now node 3 has `prev = null, next = 2`. Move `curr = 3.prev = null` — loop ends.

Finally swap `head` and `tail`: `head = 3`, `tail = 1`. Traversing forward from the new head: `3 -> 2 -> 1` (following the new `next` pointers), which matches `5 <-> 4 <-> 3 <-> 2 <-> 1`-style reversal for the 5-node example, and backward traversal from the new tail retraces `1 -> 2 -> 3`, confirming both directions were flipped correctly.

---

## 5. Delete All Occurrences of a Key in a Doubly Linked List

**Problem Statement:**
Given a doubly linked list and a target value `key`, remove every node whose value equals `key`, correctly updating `head`, `tail`, and all neighboring `next`/`prev` pointers, including cases where matching nodes are adjacent or at the boundaries.

**Example:**
- Input: `2 <-> 3 <-> 2 <-> 2 <-> 5 <-> 2`, key = `2`
- Output: `3 <-> 5`
- Explanation: All four nodes with value `2` (positions 0, 2, 3, and 5) are removed, including two adjacent occurrences in the middle and one at each boundary (head and tail), leaving only `3` and `5` linked directly to each other.

**Approach:**
Traverse the list once with a `curr` pointer. Before moving on, save `curr.next` (since `curr` may be deleted and its `next` cleared). If `curr.val == key`, unlink it exactly like single-node deletion: reconnect `curr.prev` and `curr.next` to each other (or update `head`/`tail` if `curr` is a boundary node), then clear `curr`'s own pointers. Advance to the saved next node regardless of whether a deletion happened.

```csharp
public void DeleteAllOccurrences(int key) {
    DLLNode curr = head;

    while (curr != null) {
        DLLNode nextNode = curr.next; // save before potential unlink

        if (curr.val == key) {
            DLLNode prevNode = curr.prev;

            if (prevNode != null) prevNode.next = nextNode;
            else head = nextNode; // curr was head

            if (nextNode != null) nextNode.prev = prevNode;
            else tail = prevNode; // curr was tail

            curr.next = null;
            curr.prev = null;
        }

        curr = nextNode;
    }
}
```

Time Complexity: O(n) — single pass over the list.
Space Complexity: O(1) — only pointer variables used.

**Explanation:**
List: `2 <-> 3 <-> 2 <-> 2 <-> 5 <-> 2`, key = `2`. Label nodes `A(2) <-> B(3) <-> C(2) <-> D(2) <-> E(5) <-> F(2)`, with `head = A`, `tail = F`.
- `curr = A`: `nextNode = B`. `A.val == 2`, so delete. `prevNode = A.prev = null` → `head = B`. `nextNode = B` is non-null → `B.prev = null`. Clear `A`'s pointers. Move `curr = B`.
- `curr = B`: value `3`, no deletion. `curr = C`.
- `curr = C`: `nextNode = D`. `C.val == 2`, delete. `prevNode = C.prev = B` → `B.next = D`. `nextNode = D` non-null → `D.prev = B`. Move `curr = D`.
- `curr = D`: `nextNode = E`. `D.val == 2`, delete. `prevNode = D.prev = B` (just updated) → `B.next = E`. `nextNode = E` non-null → `E.prev = B`. Move `curr = E`.
- `curr = E`: value `5`, no deletion. `curr = F`.
- `curr = F`: `nextNode = null`. `F.val == 2`, delete. `prevNode = F.prev = E` → `E.next = null`. `nextNode = null` → `tail = E`. Move `curr = null`, loop ends.

Final list: `head = B(3)`, `B.next = E(5)`, `E.prev = B`, `tail = E`. Traversal forward: `3 -> 5`; backward: `5 -> 3`. Result matches `3 <-> 5`, with all four `2`-nodes (including the two adjacent ones, C and D) correctly spliced out.
