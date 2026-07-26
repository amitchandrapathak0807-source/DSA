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

**Optimized Approach:**
Do it in-place using O(1) extra space. For each group: first check whether at least `k` nodes remain by walking ahead with a helper (`GetKthNode`); if fewer than `k` nodes remain, stop and leave them untouched. Otherwise, reverse the `k` nodes in place using the classic iterative three-pointer reversal (`prev`, `curr`, `next`), then recursively/iteratively connect the reversed group's tail to the result of processing the rest of the list.

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

**Explanation:**
Dry run on `1 -> 2 -> 3 -> 4 -> 5, k = 2`:
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

**Optimized Approach:**
Do it in-place with O(1) extra space using the tail-to-head linking trick. First find the length `n` and the tail node in one pass. Connect `tail.next = head` to form a circular list. Compute the effective rotation `k = k % n`; the new tail is at position `n - k - 1` from the original head (0-indexed), and the node after it becomes the new head. Break the circle there.

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

**Explanation:**
Dry run on `1 -> 2 -> 3 -> 4 -> 5, k = 2`:
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

**Optimized Approach:**
Use the fact each list is already sorted. Recursively flatten from the last main-list node backward (or process left to right, repeatedly merging), merging each node's child list with the already-flattened result of everything to its right, using the standard "merge two sorted lists" routine (problem 4) each time.

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

**Explanation:**
Conceptually this is repeated pairwise merging: start from the rightmost main-list node (already "flattened" since it's just its own child chain), then merge the second-to-last node's child chain into that result, and so on moving left, using the exact two-pointer merge from problem 4 (`MergeTwoSortedFlattenLists`), each merge combining two sorted chains into one sorted chain via the `child` pointer. By the time recursion unwinds back to `head`, every child chain has been merged in, producing one fully sorted list reachable via `child` pointers only.

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

**Optimized Approach:**
Standard two-pointer merge exploiting the fact both lists are already sorted — no extra array or sorting needed. Walk both lists simultaneously, at each step re-linking the smaller node's `next` pointer directly into the result, reusing existing nodes in place.

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

**Explanation:**
Dry run on `list1 = 1 -> 3 -> 5`, `list2 = 2 -> 4 -> 6`:
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

**Optimized Approach:**
The clean O(N) HashMap-based node-mapping technique: use a `Dictionary<Node, Node>` mapping each original node directly to its clone. First pass creates all clone nodes (with only `val` set) and populates the map. Second pass walks the original list again and uses the map to wire each clone's `next` and `random` pointers directly from the original node's `next`/`random`, via one dictionary lookup each — no separate index bookkeeping needed.

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

**Explanation:**
Dry run the HashMap technique on `7 -> 13 -> 11 -> 10 -> 1` with `13.random = 7`, `11.random = 1`, `10.random = 11`, `1.random = 7`:

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
