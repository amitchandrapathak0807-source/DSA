# Linked List — Hard Linked List Problems

Standard node definitions used throughout this document:

```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; this.next = null; }
}

public class Node { // for the random-pointer copy list problem
    public int val;
    public Node next;
    public Node random;
    public Node(int val) { this.val = val; this.next = null; this.random = null; }
}
```

---

## 1. Reverse a Linked List in Groups of Size K

**Problem Statement:**
Given the head of a singly linked list and an integer `k`, reverse the nodes of the list `k` at a time and return the modified list. If the number of nodes remaining at the end is less than `k`, leave those nodes as they are (do not reverse them).

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5, k = 2`
- Output: `2 -> 1 -> 4 -> 3 -> 5`
- Explanation: The first group `[1, 2]` is reversed to `[2, 1]`. The second group `[3, 4]` is reversed to `[4, 3]`. Only `[5]` remains, which is fewer than `k = 2` nodes, so it is left untouched.

**Brute Force Approach:**
Traverse the list and push node values into a list/array. Process the array in chunks of size `k`, reversing each full chunk (leaving the last partial chunk as-is), then rebuild a brand-new linked list from the resulting sequence of values.

**Logic (Steps):**
1. Walk `head` to the end, copying each `val` into a `List<int> values`.
2. Loop over `values` in strides of `k` (`i += k`), computing `end = Math.Min(i + k, n)` for each chunk.
3. If the chunk is a full group (`end - i == k`), append its elements to `result` in reverse order (`j` from `end - 1` down to `i`); otherwise append them in original order (partial trailing group).
4. Build a brand-new list from `result` using a `dummy` node and a `curr` pointer, creating a fresh `ListNode` per value.
5. Return `dummy.next`.

```csharp
public ListNode ReverseKGroupBruteForce(ListNode head, int k) {
    List<int> values = new List<int>();
    ListNode temp = head;
    while (temp != null) {
        values.Add(temp.val);
        temp = temp.next;
    }

    int n = values.Count;
    List<int> result = new List<int>();
    for (int i = 0; i < n; i += k) {
        int end = Math.Min(i + k, n);
        if (end - i == k) {
            // full group -> reverse it
            for (int j = end - 1; j >= i; j--) {
                result.Add(values[j]);
            }
        } else {
            // partial group -> keep as is
            for (int j = i; j < end; j++) {
                result.Add(values[j]);
            }
        }
    }

    ListNode dummy = new ListNode(-1);
    ListNode curr = dummy;
    foreach (int v in result) {
        curr.next = new ListNode(v);
        curr = curr.next;
    }
    return dummy.next;
}
```
Time Complexity: O(N) — one pass to collect values, one pass to rebuild the list.
Space Complexity: O(N) — extra array and a brand-new list of nodes are allocated.

**Walkthrough:** On `1 -> 2 -> 3 -> 4 -> 5, k = 2`: `values = [1,2,3,4,5]`, `n = 5`. Chunk `i=0`: `end=2`, full group, so append reversed `[2,1]`. Chunk `i=2`: `end=4`, full group, append reversed `[4,3]`. Chunk `i=4`: `end=5`, partial (only 1 element), append as-is `[5]`. So `result = [2,1,4,3,5]`. Rebuilding a new list from these values gives `2 -> 1 -> 4 -> 3 -> 5`, matching the expected output.

---

**Optimized Approach:**
Do it in-place using O(1) extra space. For each group: first check whether at least `k` nodes remain by walking ahead with a helper (`GetKthNode`); if fewer than `k` nodes remain, stop and leave them untouched. Otherwise, reverse the `k` nodes in place using the classic iterative three-pointer reversal (`prev`, `curr`, `next`), then recursively/iteratively connect the reversed group's tail to the result of processing the rest of the list.

**Logic (Steps):**
1. Call `GetKthNode(head, k)` to find the k-th node ahead; if it returns `null`, fewer than `k` nodes remain, so return `head` unchanged.
2. Otherwise save `groupNext = kth.next` (node after this group) and detach the group by setting `kth.next = null`.
3. Reverse the isolated group with `ReverseGroup(head)`, which uses the classic `prev`/`curr`/`nextNode` three-pointer iterative reversal; this returns `newHead`, the reversed group's new head.
4. Recursively process the remainder via `head.next = ReverseKGroup(groupNext, k)` — note `head` is now the tail of the reversed group, so this links the reversed group's tail to the (recursively reversed) rest of the list.
5. Return `newHead`.

```csharp
public ListNode ReverseKGroup(ListNode head, int k) {
    // Step 1: check if there are at least k nodes left
    ListNode kth = GetKthNode(head, k);
    if (kth == null) {
        // fewer than k nodes remain, leave as-is
        return head;
    }

    ListNode groupNext = kth.next; // node right after this group
    kth.next = null;               // detach the group for isolated reversal

    // Step 2: reverse the current group of k nodes
    ListNode newHead = ReverseGroup(head);

    // Step 3: head is now the tail of the reversed group;
    // recursively process the remaining list and link it
    head.next = ReverseKGroup(groupNext, k);

    return newHead;
}

private ListNode GetKthNode(ListNode head, int k) {
    ListNode curr = head;
    int count = 1;
    while (curr != null && count < k) {
        curr = curr.next;
        count++;
    }
    return curr; // null if fewer than k nodes exist
}

private ListNode ReverseGroup(ListNode head) {
    ListNode prev = null;
    ListNode curr = head;
    while (curr != null) {
        ListNode nextNode = curr.next;
        curr.next = prev;
        prev = curr;
        curr = nextNode;
    }
    return prev; // new head of the reversed group
}
```
Time Complexity: O(N) — every node is visited a constant number of times: once by `GetKthNode` bookkeeping and once during reversal, across all groups combined that is still O(N).
Space Complexity: O(N/k) for the recursion stack (one call per group); can be made O(1) with an iterative version that tracks the previous group's tail directly. All reversal itself is done in-place with pointer rewiring, no extra data structures.

**Walkthrough:** Dry run on `1 -> 2 -> 3 -> 4 -> 5, k = 2`:
1. `GetKthNode(head=1, k=2)` walks 2 nodes ahead and returns node `2`. Since it's not null, we have a full group `[1, 2]`.
2. `groupNext = 3` (node after the group). We cut the group by setting `2.next = null`, isolating `1 -> 2`.
3. `ReverseGroup(1 -> 2)` reverses it in place to `2 -> 1`. `newHead = 2`.
4. `head` (still referring to node `1`, now the tail of this reversed group) gets `head.next = ReverseKGroup(3, 2)` — i.e., we recurse on the rest of the list starting at node `3`.
5. Recursive call: `GetKthNode(head=3, k=2)` walks 2 nodes ahead and returns node `4`. Full group `[3, 4]` found. `groupNext = 5`. Cut `4.next = null`, isolating `3 -> 4`.
6. `ReverseGroup(3 -> 4)` gives `4 -> 3`. `newHead = 4`. Then `3.next = ReverseKGroup(5, 2)`.
7. Next recursive call: `GetKthNode(head=5, k=2)` tries to walk 2 nodes ahead but only finds 1 node (`5`) before running out — returns `null`. Since fewer than `k` nodes remain, this call returns `head` (node `5`) unchanged.
8. Unwinding: `3.next = 5`, so segment is `4 -> 3 -> 5`. Then `1.next = 4`, giving `2 -> 1 -> 4 -> 3 -> 5`.
9. Final result: `2 -> 1 -> 4 -> 3 -> 5`, matching the expected output — groups `[1,2]` and `[3,4]` reversed, trailing `[5]` left as-is.

---

## 2. Rotate a Linked List (to the right by K places)

**Problem Statement:**
Given the head of a singly linked list and an integer `k`, rotate the list to the right by `k` places. This means the last `k` nodes are moved to the front of the list, preserving their relative order.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5, k = 2`
- Output: `4 -> 5 -> 1 -> 2 -> 3`
- Explanation: The last 2 nodes `[4, 5]` are moved to the front, followed by the remaining nodes `[1, 2, 3]` in their original order.

**Brute Force Approach:**
Store all node values into an array (or `List<int>`). Compute the effective rotation `k % n`. Build the rotated sequence by taking the last `k` elements followed by the first `n - k` elements, then construct a new linked list from that sequence.

**Logic (Steps):**
1. Handle trivial cases: empty list or single node returns `head` as-is.
2. Traverse the list once into `values`, recording `n = values.Count`.
3. Normalize `k = k % n`; if `k == 0` no rotation is needed, return `head`.
4. Build `rotated` by taking `values.GetRange(n - k, k)` (last `k` elements) followed by `values.GetRange(0, n - k)` (the rest).
5. Construct a brand-new list from `rotated` using a `dummy`/`curr` pair and return `dummy.next`.

```csharp
public ListNode RotateRightBruteForce(ListNode head, int k) {
    if (head == null || head.next == null) return head;

    List<int> values = new List<int>();
    ListNode temp = head;
    while (temp != null) {
        values.Add(temp.val);
        temp = temp.next;
    }

    int n = values.Count;
    k = k % n;
    if (k == 0) return head;

    List<int> rotated = new List<int>();
    rotated.AddRange(values.GetRange(n - k, k));       // last k elements
    rotated.AddRange(values.GetRange(0, n - k));       // remaining elements

    ListNode dummy = new ListNode(-1);
    ListNode curr = dummy;
    foreach (int v in rotated) {
        curr.next = new ListNode(v);
        curr = curr.next;
    }
    return dummy.next;
}
```
Time Complexity: O(N) to read into the array plus O(N) to rebuild the list = O(N).
Space Complexity: O(N) for the auxiliary array/list and the newly created nodes.

**Walkthrough:** On `1 -> 2 -> 3 -> 4 -> 5, k = 2`: `values = [1,2,3,4,5]`, `n = 5`, `k = 2 % 5 = 2`. `rotated = values.GetRange(3, 2) + values.GetRange(0, 3) = [4,5] + [1,2,3] = [4,5,1,2,3]`. Rebuilding a new list from these values gives `4 -> 5 -> 1 -> 2 -> 3`, matching the expected output.

---

**Optimized Approach:**
Do it in-place with O(1) extra space using the tail-to-head linking trick. First find the length `n` and the tail node in one pass. Connect `tail.next = head` to form a circular list. Compute the effective rotation `k = k % n`; the new tail is at position `n - k - 1` from the original head (0-indexed), and the node after it becomes the new head. Break the circle there.

**Logic (Steps):**
1. Handle trivial cases: empty/single-node list or `k == 0` returns `head` unchanged.
2. Traverse from `head` to find `length` and the last node `tail` in one pass.
3. Normalize `k = k % length`; if it becomes `0`, no rotation needed.
4. Link `tail.next = head`, turning the list into a circle.
5. Walk `stepsToNewTail = length - k` steps from `head` to locate `newTail`, the node that will become the new last node.
6. Set `newHead = newTail.next`, then break the circle with `newTail.next = null`, and return `newHead`.

```csharp
public ListNode RotateRight(ListNode head, int k) {
    if (head == null || head.next == null || k == 0) return head;

    // Step 1: find length and the last node
    int length = 1;
    ListNode tail = head;
    while (tail.next != null) {
        tail = tail.next;
        length++;
    }

    // Step 2: normalize k
    k = k % length;
    if (k == 0) return head;

    // Step 3: make it circular
    tail.next = head;

    // Step 4: find the new tail = (length - k - 1)th node from head
    int stepsToNewTail = length - k;
    ListNode newTail = head;
    for (int i = 1; i < stepsToNewTail; i++) {
        newTail = newTail.next;
    }

    // Step 5: new head is right after the new tail; break the circle
    ListNode newHead = newTail.next;
    newTail.next = null;

    return newHead;
}
```
Time Complexity: O(N) — one pass to find length/tail, one partial pass (up to N steps) to find the new tail.
Space Complexity: O(1) — only a few pointers are used; no extra data structures.

**Walkthrough:** Dry run on `1 -> 2 -> 3 -> 4 -> 5, k = 2`:
1. Traverse to find `length = 5` and `tail = node(5)`.
2. `k = 2 % 5 = 2` (no change needed, already less than length).
3. Link `tail.next = head` → circular list: `1 -> 2 -> 3 -> 4 -> 5 -> 1 -> 2 -> ...`.
4. `stepsToNewTail = 5 - 2 = 3`. Starting at `newTail = head = 1`, move `3 - 1 = 2` steps: `1 -> 2 -> 3`. So `newTail = node(3)`.
5. `newHead = newTail.next = node(4)`. Break the circle: `newTail.next = null`, so node `3` now points to nothing.
6. Resulting list starting at `newHead`: `4 -> 5 -> 1 -> 2 -> 3`, matching the expected output.

---

## 3. Flatten a Multilevel/Multi-child Linked List

**Problem Statement:**
Given a linked list where each node has a `next` pointer and a `child` (or `down`) pointer, and each node's own `next` chain plus every `child` sub-list is individually sorted, flatten the entire structure into a single sorted linked list using only the `next`/`down` pointers (merging all levels together in sorted order).

For this problem we model each node as having `next` and `child`, where `child` points to another sorted linked list (using the same node type), similar to the classic "flatten a multilevel linked list" / "flatten a linked list with a bottom pointer" problem.

```csharp
public class FlattenNode {
    public int val;
    public FlattenNode next;
    public FlattenNode child; // also referred to as "down" — a sorted sub-list
    public FlattenNode(int val) { this.val = val; this.next = null; this.child = null; }
}
```

**Example:**
- Input: main list `5 -> 10 -> 19 -> 28`, where
  - `5`'s child list: `5 -> 7 -> 8 -> 30`
  - `10`'s child list: `10 -> 20`
  - `19`'s child list: `19 -> 22 -> 50`
  - `28`'s child list: `28 -> 35 -> 40 -> 50`
- Output: `5 -> 7 -> 8 -> 10 -> 19 -> 20 -> 22 -> 28 -> 30 -> 35 -> 40 -> 50 -> 50`
- Explanation: Every child sub-list (and the main list itself) is already individually sorted. Flattening merges all of them into one fully sorted list connected purely via `child`/`next` pointers, moving top-to-bottom, left-to-right.

**Brute Force Approach:**
Traverse the whole structure (main list, then each child list) collecting every node's value into a single array, sort the array, then rebuild a brand-new flattened list (using the `child` pointer as the single-level `next`) from the sorted values.

**Logic (Steps):**
1. Walk the main list with `mainCurr`; for each main node, walk its child chain with `childCurr`, adding every visited `val` to a flat `values` list.
2. Sort `values` in ascending order.
3. Build a new chain of `FlattenNode`s from the sorted values, linking them purely via the `child` pointer (`curr.child = new FlattenNode(v)`).
4. Return `dummy.child`, the head of the newly built sorted chain.

```csharp
public FlattenNode FlattenBruteForce(FlattenNode head) {
    List<int> values = new List<int>();

    FlattenNode mainCurr = head;
    while (mainCurr != null) {
        FlattenNode childCurr = mainCurr;
        while (childCurr != null) {
            values.Add(childCurr.val);
            childCurr = childCurr.child;
        }
        mainCurr = mainCurr.next;
    }

    values.Sort();

    FlattenNode dummy = new FlattenNode(-1);
    FlattenNode curr = dummy;
    foreach (int v in values) {
        curr.child = new FlattenNode(v);
        curr = curr.child;
    }
    return dummy.child;
}
```
Time Complexity: O(N log N) where N is the total number of nodes, dominated by the sort; collecting values is O(N).
Space Complexity: O(N) for the values array and the newly allocated flattened list.

**Walkthrough:** On the example input, every node's value (main list `5,10,19,28` plus every child chain value) is collected into `values`, then sorted to `[5,7,8,10,19,20,22,28,30,35,40,50,50]`. Rebuilding a `child`-linked chain from this sorted list yields `5 -> 7 -> 8 -> 10 -> 19 -> 20 -> 22 -> 28 -> 30 -> 35 -> 40 -> 50 -> 50`, matching the expected output.

---

**Optimized Approach:**
Use the fact each list is already sorted. Recursively flatten from the last main-list node backward (or process left to right, repeatedly merging), merging each node's child list with the already-flattened result of everything to its right, using the standard "merge two sorted lists" routine (problem 4) each time.

**Logic (Steps):**
1. Base case: if `head` is `null` or `head.next` is `null`, the list is already "flat" (it's just its own child chain, or empty) — return `head`.
2. Recursively flatten the rest of the main list first: `head.next = Flatten(head.next)`, so the tail end is processed before the current node.
3. Merge `head`'s own child chain with the already-flattened rest (`head.next`) using `MergeTwoSortedFlattenLists`, which compares `val`s and links nodes via `child` exactly like the standard merge in problem 4 but along the `child` pointer.
4. Reassign `head = MergeTwoSortedFlattenLists(head, head.next)` so `head` now points at the merged chain's true head, then clear any stray `next` pointers in the merged result so only `child` defines the final chain.
5. Return `head`.

```csharp
public FlattenNode Flatten(FlattenNode head) {
    if (head == null || head.next == null) return head;

    // recursively flatten the rest of the main list first
    head.next = Flatten(head.next);

    // merge current node's own child-chain with the flattened rest
    head = MergeTwoSortedFlattenLists(head, head.next);

    return head;
}

private FlattenNode MergeTwoSortedFlattenLists(FlattenNode a, FlattenNode b) {
    FlattenNode dummy = new FlattenNode(-1);
    FlattenNode tail = dummy;

    while (a != null && b != null) {
        if (a.val <= b.val) {
            tail.child = a;
            a = a.child;
        } else {
            tail.child = b;
            b = b.child;
        }
        tail = tail.child;
    }
    tail.child = (a != null) ? a : b;

    // once merged into a single "child" chain, clear stray next pointers
    FlattenNode result = dummy.child;
    FlattenNode curr = result;
    while (curr != null) {
        curr.next = null;
        curr = curr.child;
    }
    return result;
}
```
Time Complexity: O(N) overall — although it looks like repeated merges, each node is compared/moved a constant number of times across the whole recursion because every node is consumed into exactly one merge step per level of the main list, giving a standard merge-based total of O(N) (similar to merging a list of already partially-sorted runs where total work is bounded by total nodes, not N log(number of lists), because merging proceeds pairwise from the tail).
Space Complexity: O(N) recursion stack depth in the worst case (main list length), O(1) extra space per merge step besides pointers.

**Walkthrough:** Conceptually this is repeated pairwise merging on the example input: recursion first reaches node `28`, whose own child chain `28 -> 35 -> 40 -> 50` is already "flat" (base case). Unwinding to `19`, its child chain `19 -> 22 -> 50` is merged with `28`'s flattened chain, producing `19 -> 22 -> 28 -> 30(from 28's child)...` — more precisely `MergeTwoSortedFlattenLists` interleaves both sorted chains via `child`. Continuing to unwind at `10` (child chain `10 -> 20`) and finally `5` (child chain `5 -> 7 -> 8 -> 30`), each node's own child chain is merged with everything already flattened to its right, using the exact two-pointer merge from problem 4 (`MergeTwoSortedFlattenLists`). By the time recursion unwinds fully back to `head = 5`, the result is `5 -> 7 -> 8 -> 10 -> 19 -> 20 -> 22 -> 28 -> 30 -> 35 -> 40 -> 50 -> 50`, matching the expected output.

---

## 4. Merge Two Sorted Linked Lists

**Problem Statement:**
Given the heads of two sorted (ascending) singly linked lists, merge them into a single sorted linked list by splicing together the nodes of the two lists, and return the head of the merged list.

**Example:**
- Input: `list1 = 1 -> 3 -> 5`, `list2 = 2 -> 4 -> 6`
- Output: `1 -> 2 -> 3 -> 4 -> 5 -> 6`
- Explanation: At each step, the smaller of the two current nodes is chosen and appended to the result; once one list is exhausted, the remainder of the other list is appended as-is.

**Brute Force Approach:**
Copy all values from both lists into a single array, sort the combined array, then build a brand-new linked list from the sorted values.

**Logic (Steps):**
1. Traverse `list1` fully, appending each `val` to `values`.
2. Traverse `list2` fully, appending each `val` to the same `values` list.
3. Sort `values` ascending.
4. Build a brand-new list from the sorted `values` using a `dummy`/`curr` pair.
5. Return `dummy.next`.

```csharp
public ListNode MergeTwoListsBruteForce(ListNode list1, ListNode list2) {
    List<int> values = new List<int>();

    ListNode temp = list1;
    while (temp != null) {
        values.Add(temp.val);
        temp = temp.next;
    }
    temp = list2;
    while (temp != null) {
        values.Add(temp.val);
        temp = temp.next;
    }

    values.Sort();

    ListNode dummy = new ListNode(-1);
    ListNode curr = dummy;
    foreach (int v in values) {
        curr.next = new ListNode(v);
        curr = curr.next;
    }
    return dummy.next;
}
```
Time Complexity: O((M + N) log(M + N)) due to sorting the combined array of M + N elements.
Space Complexity: O(M + N) for the values array and the newly created merged list.

**Walkthrough:** On `list1 = 1 -> 3 -> 5`, `list2 = 2 -> 4 -> 6`: `values = [1,3,5,2,4,6]`, sorted to `[1,2,3,4,5,6]`. Rebuilding a new list from these values gives `1 -> 2 -> 3 -> 4 -> 5 -> 6`, matching the expected output.

---

**Optimized Approach:**
Standard two-pointer merge exploiting the fact both lists are already sorted — no extra array or sorting needed. Walk both lists simultaneously, at each step re-linking the smaller node's `next` pointer directly into the result, reusing existing nodes in place.

**Logic (Steps):**
1. Create a `dummy` node and a `tail` pointer starting at `dummy`.
2. While both `list1` and `list2` are non-null, compare their `val`s; link `tail.next` to whichever node is smaller (or `list1`'s node on a tie), advance that list's pointer, then advance `tail`.
3. When one list is exhausted, attach the remaining (already-sorted) tail of the other list directly via `tail.next`.
4. Return `dummy.next`, the head of the merged list.

```csharp
public ListNode MergeTwoLists(ListNode list1, ListNode list2) {
    ListNode dummy = new ListNode(-1);
    ListNode tail = dummy;

    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            tail.next = list1;
            list1 = list1.next;
        } else {
            tail.next = list2;
            list2 = list2.next;
        }
        tail = tail.next;
    }

    // attach the remainder of whichever list still has nodes
    tail.next = (list1 != null) ? list1 : list2;

    return dummy.next;
}
```
Time Complexity: O(M + N) — each node from both lists is visited exactly once.
Space Complexity: O(1) extra space — existing nodes are re-linked in place; only a dummy node and a couple of pointers are used (excluding the O(1) space for the pointers themselves, no new nodes are allocated).

**Walkthrough:** Dry run on `list1 = 1 -> 3 -> 5`, `list2 = 2 -> 4 -> 6`:
1. `dummy -> null`, `tail = dummy`. Compare `1` vs `2`: `1 <= 2`, so `tail.next = node(1)`; advance `list1` to `3`; `tail = node(1)`.
2. Compare `3` vs `2`: `2 < 3`, so `tail.next = node(2)`; advance `list2` to `4`; `tail = node(2)`.
3. Compare `3` vs `4`: `3 <= 4`, so `tail.next = node(3)`; advance `list1` to `5`; `tail = node(3)`.
4. Compare `5` vs `4`: `4 < 5`, so `tail.next = node(4)`; advance `list2` to `6`; `tail = node(4)`.
5. Compare `5` vs `6`: `5 <= 6`, so `tail.next = node(5)`; advance `list1` to `null`; `tail = node(5)`.
6. `list1` is now `null`, loop ends. `tail.next = list2` (which is `node(6)`), attaching the remainder.
7. Result starting at `dummy.next`: `1 -> 2 -> 3 -> 4 -> 5 -> 6`, matching the expected output.

---

## 5. Clone a Linked List with Random Pointers

**Problem Statement:**
Given a linked list where each node has an additional `random` pointer that can point to any node in the list (or `null`), construct a deep copy of the list. The cloned list should be completely independent of the original — every `next` and `random` pointer in the clone must point to nodes within the clone, not the original.

**Example:**
- Input: `head = [ (val=7, random=null), (val=13, random -> node0), (val=11, random -> node4), (val=10, random -> node2), (val=1, random -> node0) ]` (0-indexed nodes in list order: `7 -> 13 -> 11 -> 10 -> 1`, with `13.random = 7`, `11.random = 1`, `10.random = 11`, `1.random = 7`)
- Output: A new list `7' -> 13' -> 11' -> 10' -> 1'` where `13'.random = 7'`, `11'.random = 1'`, `10'.random = 11'`, `1'.random = 7'` — same structure but every node is a distinct new object.
- Explanation: The clone mirrors the exact `next` chain and `random` targets of the original, but no pointer in the clone ever references an original node.

**Brute Force Approach:**
First pass: create a new node for each original node (copying only `val`, `next = null`, `random = null`) and store the mapping `original node -> its index` (or directly store copies in an array/list) alongside a hashmap from original node reference to the corresponding new node. Second pass: for each original node, set the clone's `next` and `random` by looking up the mapped clone via the hashmap. (This "brute force" is effectively the same idea as the optimized HashMap approach below but implemented with an explicit index array first — shown here using two full passes plus an auxiliary array for clarity as the more basic version before consolidating into the cleaner optimized form.)

**Logic (Steps):**
1. Traverse `head` once, collecting every original node reference into `originalNodes` (an ordered list).
2. Create a parallel `clonedNodes` list, one plain clone (`new Node(n.val)`) per original node, in the same order.
3. Build `indexMap`, a `Dictionary<Node, int>` mapping each original node to its position `i` in `originalNodes`.
4. For each index `i`, wire `clonedNodes[i].next` and `clonedNodes[i].random` by looking up the index of `originalNodes[i].next` / `.random` in `indexMap` and pointing at the corresponding entry in `clonedNodes`.
5. Return `clonedNodes[0]`, the head of the cloned list.

```csharp
public Node CopyRandomListBruteForce(Node head) {
    if (head == null) return null;

    List<Node> originalNodes = new List<Node>();
    Node temp = head;
    while (temp != null) {
        originalNodes.Add(temp);
        temp = temp.next;
    }

    // create plain clones first, store in a parallel list
    List<Node> clonedNodes = new List<Node>();
    foreach (Node n in originalNodes) {
        clonedNodes.Add(new Node(n.val));
    }

    // build original-node -> index map
    Dictionary<Node, int> indexMap = new Dictionary<Node, int>();
    for (int i = 0; i < originalNodes.Count; i++) {
        indexMap[originalNodes[i]] = i;
    }

    // wire next and random using indices
    for (int i = 0; i < originalNodes.Count; i++) {
        if (originalNodes[i].next != null) {
            clonedNodes[i].next = clonedNodes[indexMap[originalNodes[i].next]];
        }
        if (originalNodes[i].random != null) {
            clonedNodes[i].random = clonedNodes[indexMap[originalNodes[i].random]];
        }
    }

    return clonedNodes[0];
}
```
Time Complexity: O(N) — a constant number of passes over the N nodes.
Space Complexity: O(N) for the original-node list, the cloned-node list, and the index hashmap.

**Walkthrough:** On `7 -> 13 -> 11 -> 10 -> 1` (with `13.random=7`, `11.random=1`, `10.random=11`, `1.random=7`): `originalNodes = [7,13,11,10,1]`, `clonedNodes = [7',13',11',10',1']`, `indexMap = {7:0, 13:1, 11:2, 10:3, 1:4}`. Wiring by index: `clonedNodes[0].next = clonedNodes[1] (13')`, ... down to `clonedNodes[4].next = null`; randoms wire the same way (e.g. `13.random = 7` → index `0` → `clonedNodes[1].random = clonedNodes[0] = 7'`). Returning `clonedNodes[0]` gives `7' -> 13' -> 11' -> 10' -> 1'` with matching random pointers, an independent deep copy as expected.

---

**Optimized Approach:**
The clean O(N) HashMap-based node-mapping technique: use a `Dictionary<Node, Node>` mapping each original node directly to its clone. First pass creates all clone nodes (with only `val` set) and populates the map. Second pass walks the original list again and uses the map to wire each clone's `next` and `random` pointers directly from the original node's `next`/`random`, via one dictionary lookup each — no separate index bookkeeping needed.

**Logic (Steps):**
1. Pass 1: walk the original list with `curr`, and for each node create `map[curr] = new Node(curr.val)` — a bare clone with no `next`/`random` set yet.
2. Pass 2: walk the original list again with `curr`; set `map[curr].next = map[curr.next]` if `curr.next` exists (else `null`), and `map[curr].random = map[curr.random]` if `curr.random` exists (else `null`) — both are simple dictionary lookups since every original node already has a clone in `map`.
3. Return `map[head]`, the clone corresponding to the original head.

```csharp
public Node CopyRandomList(Node head) {
    if (head == null) return null;

    Dictionary<Node, Node> map = new Dictionary<Node, Node>();

    // Pass 1: create clone nodes (val only) and map original -> clone
    Node curr = head;
    while (curr != null) {
        map[curr] = new Node(curr.val);
        curr = curr.next;
    }

    // Pass 2: wire next and random pointers using the map
    curr = head;
    while (curr != null) {
        map[curr].next = (curr.next != null) ? map[curr.next] : null;
        map[curr].random = (curr.random != null) ? map[curr.random] : null;
        curr = curr.next;
    }

    return map[head];
}
```
Time Complexity: O(N) — exactly two linear passes over the original list, each dictionary operation is O(1) average.
Space Complexity: O(N) for the hashmap storing one entry per node (plus O(N) for the N newly allocated clone nodes, which is unavoidable output space).

**Bonus O(1) extra-space improvement (interleaving trick):** instead of a hashmap, interleave each clone directly after its original node (`orig1 -> clone1 -> orig2 -> clone2 -> ...`) in a first pass. In a second pass, set each `clone.random = orig.random.next` (since `orig.random`'s clone is right after it) using only the interleaved structure itself. In a third pass, unweave the two lists back into separate original and cloned lists by restoring `orig.next` and setting `clone.next` correctly. This avoids the O(N) hashmap, using only O(1) extra pointers (excluding the O(N) output list itself).

**Walkthrough:** Dry run the HashMap technique on `7 -> 13 -> 11 -> 10 -> 1` with `13.random = 7`, `11.random = 1`, `10.random = 11`, `1.random = 7`:

*Pass 1 (build the map):*
- Visit `7`: create clone `7'`, `map[7] = 7'`.
- Visit `13`: create clone `13'`, `map[13] = 13'`.
- Visit `11`: create clone `11'`, `map[11] = 11'`.
- Visit `10`: create clone `10'`, `map[10] = 10'`.
- Visit `1`: create clone `1'`, `map[1] = 1'`.
- Map now: `{7: 7', 13: 13', 11: 11', 10: 10', 1: 1'}`.

*Pass 2 (wire next and random using the map):*
- At `7`: `map[7].next = map[13] = 13'` (since `7.next = 13`); `7.random = null`, so `map[7].random = null`. Result: `7'.next = 13'`, `7'.random = null`.
- At `13`: `map[13].next = map[11] = 11'`; `13.random = 7`, so `map[13].random = map[7] = 7'`. Result: `13'.next = 11'`, `13'.random = 7'`.
- At `11`: `map[11].next = map[10] = 10'`; `11.random = 1`, so `map[11].random = map[1] = 1'`. Result: `11'.next = 10'`, `11'.random = 1'`.
- At `10`: `map[10].next = map[1] = 1'`; `10.random = 11`, so `map[10].random = map[11] = 11'`. Result: `10'.next = 1'`, `10'.random = 11'`.
- At `1`: `1.next = null`, so `map[1].next = null`; `1.random = 7`, so `map[1].random = map[7] = 7'`. Result: `1'.next = null`, `1'.random = 7'`.

*Final cloned list:* `7' -> 13' -> 11' -> 10' -> 1'`, with `13'.random = 7'`, `11'.random = 1'`, `10'.random = 11'`, `1'.random = 7'` — an exact structural mirror of the original, but every node and every pointer belongs entirely to the new clone, satisfying the deep-copy requirement.
