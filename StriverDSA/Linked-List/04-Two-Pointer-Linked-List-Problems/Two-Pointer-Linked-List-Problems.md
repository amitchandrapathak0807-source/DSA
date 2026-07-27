# Linked List — Two Pointer Linked List Problems

Standard singly linked list node used throughout this document:

```csharp
public class ListNode {
    public int val;
    public ListNode next;
    public ListNode(int val) { this.val = val; this.next = null; }
}
```

---

## 1. Delete the Middle Node of a Linked List

### 1. Delete the Middle Node of a Linked List

**Problem Statement:** Given the head of a singly linked list, delete the middle node of the linked list and return the head of the modified list. The middle node of a linked list of size `n` is the `⌊n / 2⌋`-th node (0-indexed).

**Example:**
- Input: `1 -> 3 -> 4 -> 7 -> 1 -> 2 -> 6`
- Output: `1 -> 3 -> 4 -> 1 -> 2 -> 6`
- Explanation: The list has 7 nodes, so the middle index is `7 / 2 = 3` (0-indexed), which is the node with value `7`. Removing it links node `4` directly to node `1`.

**Brute Force Approach:** First traverse the list once to count the total number of nodes `n`. Then traverse again up to node `n/2 - 1` (the node just before the middle) and unlink the middle node by pointing its `next` reference past it.

**Logic (Steps):**
1. Handle the trivial edge case: if the list is empty or has only one node, there's nothing meaningful to keep, so return `null`.
2. Traverse the entire list once with `temp`, incrementing `count` for each node, to find the total length `n`.
3. Compute `middleIndex = count / 2` (0-indexed middle position).
4. Traverse again from `head` using `prev`, moving `middleIndex - 1` times, so `prev` lands on the node just before the middle node.
5. Unlink the middle node with `prev.next = prev.next.next`.
6. Return the (unchanged) `head`.

```csharp
public ListNode DeleteMiddleBrute(ListNode head) {
    if (head == null || head.next == null) return null;

    // Count total nodes
    int count = 0;
    ListNode temp = head;
    while (temp != null) {
        count++;
        temp = temp.next;
    }

    int middleIndex = count / 2;

    // Traverse to node just before the middle
    ListNode prev = head;
    for (int i = 0; i < middleIndex - 1; i++) {
        prev = prev.next;
    }

    // Unlink the middle node
    prev.next = prev.next.next;

    return head;
}
```

Time Complexity: O(n) — one pass to count nodes, another partial pass to reach the middle.
Space Complexity: O(1) — no extra data structures used.

**Walkthrough:** Using `1 -> 3 -> 4 -> 7 -> 1 -> 2 -> 6` (7 nodes):
- First pass counts `count = 7`, so `middleIndex = 7 / 2 = 3`.
- Second pass moves `prev` from `head` (`1`) forward `middleIndex - 1 = 2` times: `prev = 3`, then `prev = 4`.
- `prev` (value `4`) is the node just before the middle node `7`. Unlink: `prev.next = prev.next.next` makes `4.next = 1` (skipping over `7`).
- Result: `1 -> 3 -> 4 -> 1 -> 2 -> 6`, matching the expected Output.

---

**Optimized Approach:** Use the slow and fast pointer technique. Move `fast` two steps and `slow` one step at a time, but start `fast` two steps ahead of `slow` (i.e., `fast = head.next.next`, `slow = head`). When `fast` reaches the end, `slow` is positioned exactly at the node just before the middle, so it can be unlinked directly in a single pass.

**Logic (Steps):**
1. Handle the trivial edge case: if the list is empty or has one node, return `null`.
2. Initialize `slow = head` and `fast = head.next.next` (fast starts two nodes ahead).
3. Loop while `fast != null && fast.next != null`: advance `slow` by one node and `fast` by two nodes each iteration.
4. When the loop ends, `slow` sits exactly one node before the true middle node.
5. Unlink the middle node with `slow.next = slow.next.next`.
6. Return `head`.

```csharp
public ListNode DeleteMiddleOptimal(ListNode head) {
    if (head == null || head.next == null) return null;

    ListNode slow = head;
    ListNode fast = head.next.next;

    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }

    // slow now points to the node just before the middle node
    slow.next = slow.next.next;

    return head;
}
```

Time Complexity: O(n), Space Complexity: O(1).

**Walkthrough:** Using `1 -> 3 -> 4 -> 7 -> 1 -> 2 -> 6`:
- Initial: `slow = 1` (head), `fast = 4` (head.next.next).
- Loop check: `fast (4) != null` and `fast.next (7) != null` → move: `slow = 3`, `fast = 1` (the second `1`, index 4).
- Loop check: `fast (1) != null` and `fast.next (2) != null` → move: `slow = 4`, `fast = 6`.
- Loop check: `fast (6) != null` but `fast.next == null` → stop.
- `slow` (value `4`) is the node just before the middle (`7`). Unlink: `slow.next = slow.next.next` makes `4.next = 1`.
- Result: `1 -> 3 -> 4 -> 1 -> 2 -> 6`, matching the expected Output — achieved in a single pass since `fast` started two nodes ahead of `slow`.

---

## 2. Find the Intersection Point of Two Linked Lists (Y-shaped list)

### 2. Find the Intersection Point of Two Linked Lists (Y-shaped list)

**Problem Statement:** Given the heads of two singly linked lists, `headA` and `headB`, that may or may not intersect at some node (forming a Y-shape by sharing a common tail), find and return the node at which the two lists intersect. If the two lists have no intersection, return `null`. The intersection is determined by reference (node identity), not by value.

**Example:**
- Input: List A: `4 -> 1 -> 8 -> 4 -> 5`, List B: `5 -> 6 -> 1 -> 8 -> 4 -> 5`, where both lists share the tail starting at node with value `8` (so `8 -> 4 -> 5` is common).
- Output: Node with value `8` (the first common node).
- Explanation: List A has nodes `4, 1` before merging into the shared segment `8, 4, 5`. List B has nodes `5, 6, 1` before merging into the same shared segment. The intersection point is the node `8` because that is the first node reference shared by both lists.

**Brute Force Approach:** For every node in list A, traverse the entirety of list B and compare node references. If a match is found, that is the intersection point.

**Logic (Steps):**
1. Start `nodeA` at `headA`.
2. For each `nodeA`, start a fresh `nodeB` at `headB` and walk it to the end, comparing `nodeA == nodeB` (reference equality) at each step.
3. If a match is found, return that node immediately — it's the intersection point.
4. If `nodeB` runs out without a match, advance `nodeA` to the next node and repeat.
5. If `nodeA` runs out entirely with no match found, return `null`.

```csharp
public ListNode GetIntersectionNodeBrute(ListNode headA, ListNode headB) {
    ListNode nodeA = headA;

    while (nodeA != null) {
        ListNode nodeB = headB;
        while (nodeB != null) {
            if (nodeA == nodeB) {
                return nodeA;
            }
            nodeB = nodeB.next;
        }
        nodeA = nodeA.next;
    }

    return null;
}
```

Time Complexity: O(m * n), where `m` and `n` are the lengths of list A and list B.
Space Complexity: O(1).

**Walkthrough:** Using List A: `4 -> 1 -> 8 -> 4 -> 5`, List B: `5 -> 6 -> 1 -> 8 -> 4 -> 5` (sharing tail `8 -> 4 -> 5`):
- `nodeA = 4` (first node of A). Compare against every node of B (`5, 6, 1, 8, 4, 5`) — no reference match found.
- `nodeA = 1` (second node of A). Compare against every node of B again — no match.
- `nodeA = 8` (third node of A, the shared node). Compare against B's nodes: `5` no, `6` no, `1` no, `8` — reference matches (same node object as A's `8`)!
- Return this node (`8`) immediately, matching the expected Output.

---

**Optimized Approach (Length Difference):** Compute the lengths of both lists. Advance the pointer of the longer list by the length difference so both pointers have an equal number of remaining nodes to traverse. Then move both pointers together one step at a time; the node at which they become equal is the intersection point.

**Logic (Steps):**
1. Compute `lenA` and `lenB`, the lengths of list A and list B, via `GetLength`.
2. Set `ptrA = headA` and `ptrB = headB`.
3. Whichever list is longer, advance its pointer by the difference `|lenA - lenB|` steps so both pointers now have the same number of remaining nodes to the end.
4. Move `ptrA` and `ptrB` forward together, one step at a time, until `ptrA == ptrB` (reference equality) — this is either the intersection node or `null`.
5. Return `ptrA`.

```csharp
public ListNode GetIntersectionNodeLengthDiff(ListNode headA, ListNode headB) {
    int lenA = GetLength(headA);
    int lenB = GetLength(headB);

    ListNode ptrA = headA;
    ListNode ptrB = headB;

    if (lenA > lenB) {
        for (int i = 0; i < lenA - lenB; i++) ptrA = ptrA.next;
    } else {
        for (int i = 0; i < lenB - lenA; i++) ptrB = ptrB.next;
    }

    while (ptrA != ptrB) {
        ptrA = ptrA.next;
        ptrB = ptrB.next;
    }

    return ptrA; // either the intersection node or null
}

private int GetLength(ListNode head) {
    int len = 0;
    while (head != null) {
        len++;
        head = head.next;
    }
    return len;
}
```

**Walkthrough:** Using List A: `4 -> 1 -> 8 -> 4 -> 5` (`lenA = 5`), List B: `5 -> 6 -> 1 -> 8 -> 4 -> 5` (`lenB = 6`), shared tail `8 -> 4 -> 5`:
- `lenB (6) > lenA (5)`, so advance `ptrB` by `lenB - lenA = 1` step: `ptrB` moves from `5` (head) to `6`.
- Now both pointers have 5 remaining nodes ahead. Compare `ptrA (4) != ptrB (6)` → move both: `ptrA = 1`, `ptrB = 1`.
- Compare `ptrA (1) != ptrB (1)` — these are different node objects with the same value, so not equal by reference → move both: `ptrA = 8`, `ptrB = 8`.
- Compare `ptrA == ptrB` (both reference the shared node `8`) → loop stops.
- Return `ptrA`, the node `8`, matching the expected Output.

---

**Optimized Approach ("Switch Heads" Trick):** Use two pointers, one starting at `headA` and the other at `headB`. Advance both one step at a time. When a pointer reaches the end of its list (`null`), redirect it to the head of the *other* list. Continue this until the two pointers point to the same node — that node is the intersection (or both become `null` simultaneously if there is no intersection).

**Logic (Steps):**
1. Handle the trivial edge case: if either `headA` or `headB` is `null`, there can be no intersection, so return `null`.
2. Initialize `ptrA = headA` and `ptrB = headB`.
3. Loop while `ptrA != ptrB`: advance each pointer by one node; when a pointer hits `null`, redirect it to the head of the *other* list instead of stopping.
4. Because both pointers together traverse `lenA + lenB` nodes before meeting, they become synchronized and land on the same node — the intersection — or both reach `null` at the same time if there is none.
5. Return `ptrA` (the intersection node, or `null`).

```csharp
public ListNode GetIntersectionNodeOptimal(ListNode headA, ListNode headB) {
    if (headA == null || headB == null) return null;

    ListNode ptrA = headA;
    ListNode ptrB = headB;

    while (ptrA != ptrB) {
        ptrA = (ptrA == null) ? headB : ptrA.next;
        ptrB = (ptrB == null) ? headA : ptrB.next;
    }

    return ptrA; // intersection node, or null if lists don't intersect
}
```

Time Complexity: O(n + m), Space Complexity: O(1).

**Walkthrough:** Dry run on list A (`4 -> 1 -> 8 -> 4 -> 5`, length 5) and list B (`5 -> 6 -> 1 -> 8 -> 4 -> 5`, length 6), sharing tail `8 -> 4 -> 5`:
- `ptrA` starts at `headA` (`4`), `ptrB` starts at `headB` (`5`).
- `ptrA` traverses A's 5 nodes (`4, 1, 8, 4, 5`), then hits `null` and switches to `headB`; `ptrB` meanwhile traverses B's 6 nodes (`5, 6, 1, 8, 4, 5`), then hits `null` and switches to `headA`.
- `ptrA`'s total path to the intersection: `lenA (5) + lenB_beforeIntersection (3)` = 8 steps. `ptrB`'s total path: `lenB (6) + lenA_beforeIntersection (2)` = 8 steps — identical step counts.
- Because each pointer's total distance sums to `lenA + lenB`, the switch absorbs the length difference and both pointers arrive at the shared node `8` simultaneously.
- Loop exits with `ptrA == ptrB` at node `8`. Return `ptrA`, matching the expected Output. (If the lists never intersected, both pointers would become `null` at the same time after `lenA + lenB` steps, and `null` would be returned.)

---

## 3. Remove the Nth Node from the End of a Linked List

### 3. Remove the Nth Node from the End of a Linked List

**Problem Statement:** Given the head of a singly linked list and an integer `n`, remove the `n`-th node from the end of the list and return the head of the resulting list.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`
- Output: `1 -> 2 -> 3 -> 5`
- Explanation: Counting from the end, the 2nd node from the end is `4` (order from end: `5` is 1st, `4` is 2nd). Removing it links `3` directly to `5`.

**Brute Force Approach:** Traverse the list once to compute its length `L`. The node to remove is the `(L - n + 1)`-th node from the beginning (1-indexed). Traverse again to the node just before it and unlink it. Handle the edge case where the head itself must be removed (when `n == L`).

**Logic (Steps):**
1. Traverse the list once with `temp` to compute its total length `L`.
2. If `n == L`, the node to remove is the head itself — return `head.next` directly.
3. Otherwise, compute `positionFromStart = L - n`, the 1-indexed position of the node just before the target.
4. Traverse again from `head` using `prev`, moving `positionFromStart - 1` times, so `prev` lands just before the node to remove.
5. Unlink the target node with `prev.next = prev.next.next`.
6. Return `head`.

```csharp
public ListNode RemoveNthFromEndBrute(ListNode head, int n) {
    int length = 0;
    ListNode temp = head;
    while (temp != null) {
        length++;
        temp = temp.next;
    }

    // If the node to remove is the head itself
    if (n == length) {
        return head.next;
    }

    int positionFromStart = length - n; // 1-indexed position of node just before target
    ListNode prev = head;
    for (int i = 1; i < positionFromStart; i++) {
        prev = prev.next;
    }

    prev.next = prev.next.next;

    return head;
}
```

Time Complexity: O(L), where `L` is the length of the list (two passes, but linear overall).
Space Complexity: O(1).

**Walkthrough:** Using `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`:
- First pass: `length = 5`.
- `n (2) != length (5)`, so `positionFromStart = 5 - 2 = 3`.
- Second pass: move `prev` from `head` (`1`) forward `positionFromStart - 1 = 2` times: `prev = 2`, then `prev = 3`.
- `prev` (value `3`) is just before the target node `4`. Unlink: `prev.next = prev.next.next` makes `3.next = 5`.
- Result: `1 -> 2 -> 3 -> 5`, matching the expected Output.

---

**Optimized Approach:** Use a dummy node pointing to `head` to simplify edge cases (like removing the head). Use two pointers, `fast` and `slow`, both starting at the dummy node. Move `fast` ahead by `n` steps first. Then move both `fast` and `slow` together until `fast` reaches the last node. At that point, `slow` is positioned just before the node to be removed.

**Logic (Steps):**
1. Create a `dummy` node pointing to `head` to uniformly handle the case where the head itself is removed.
2. Initialize `fast = dummy` and `slow = dummy`.
3. Advance `fast` by `n` steps, creating a gap of `n` nodes between `fast` and `slow`.
4. Move `fast` and `slow` forward together, one step at a time, until `fast.next == null` (fast is on the last node).
5. At this point `slow` sits exactly `n` nodes behind `fast`, i.e., just before the node to remove. Unlink it with `slow.next = slow.next.next`.
6. Return `dummy.next` as the new head.

```csharp
public ListNode RemoveNthFromEndOptimal(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;

    ListNode fast = dummy;
    ListNode slow = dummy;

    // Move fast n steps ahead
    for (int i = 0; i < n; i++) {
        fast = fast.next;
    }

    // Move both until fast reaches the last node
    while (fast.next != null) {
        fast = fast.next;
        slow = slow.next;
    }

    // slow is now just before the node to remove
    slow.next = slow.next.next;

    return dummy.next;
}
```

Time Complexity: O(L), Space Complexity: O(1).

**Walkthrough:** Dry run of the two-pointer N-gap technique on `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`:
- `dummy -> 1 -> 2 -> 3 -> 4 -> 5`. `fast = dummy`, `slow = dummy`.
- Move `fast` ahead by `n = 2` steps: `fast` goes from `dummy` to `1` to `2`. Now `fast` is at node `2`, `slow` is still at `dummy`.
- Advance both together while `fast.next != null`: `fast.next = 3` → move (`fast = 3, slow = 1`); `fast.next = 4` → move (`fast = 4, slow = 2`); `fast.next = 5` → move (`fast = 5, slow = 3`); `fast.next = null` → stop.
- `slow` is at node `3`, and `slow.next` is node `4`, exactly the 2nd node from the end. Unlink it: `slow.next = slow.next.next` makes `3.next = 5`.
- Result: `dummy -> 1 -> 2 -> 3 -> 5`, return `dummy.next` → `1 -> 2 -> 3 -> 5`, matching the expected Output.

---

## 4. Add 1 to a Number Represented as a Linked List

### 4. Add 1 to a Number Represented as a Linked List

**Problem Statement:** Given a non-negative integer represented as a singly linked list of digits (the most significant digit is at the head of the list), add 1 to the number and return the head of the resulting list. Assume no leading zeros in the input, except when the number itself is `0`.

**Example:**
- Input: `1 -> 2 -> 9`  (represents 129)
- Output: `1 -> 3 -> 0`  (129 + 1 = 130)
- Explanation: Adding 1 to 129 gives 130. Since the last digit `9` overflows to `0` with a carry, and that carry propagates to `2`, making it `3`. The first digit `1` is unaffected.

**Brute Force Approach:** Traverse the list to build the decimal number represented by the digits (be careful with very large numbers — using `long`/`BigInteger` in real scenarios), add 1 to it, then convert the resulting number back into a new linked list.

**Logic (Steps):**
1. Traverse the list, appending each node's `val` to a `StringBuilder` to build the full digit string (most-significant-first, matching the list order).
2. Parse that string into a `BigInteger` so arbitrarily large numbers are handled safely.
3. Add `1` to the parsed number.
4. Convert the result back to a string, then build a brand-new linked list (via a `dummy` node) with one node per resulting digit.
5. Return `dummy.next`.

```csharp
public ListNode AddOneBrute(ListNode head) {
    // Build the number as a string
    System.Text.StringBuilder sb = new System.Text.StringBuilder();
    ListNode temp = head;
    while (temp != null) {
        sb.Append(temp.val);
        temp = temp.next;
    }

    // Convert to a big integer using System.Numerics for safety with large numbers
    System.Numerics.BigInteger number = System.Numerics.BigInteger.Parse(sb.ToString());
    number += 1;

    string resultStr = number.ToString();

    // Build a new linked list from the resulting digits
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    foreach (char c in resultStr) {
        current.next = new ListNode(c - '0');
        current = current.next;
    }

    return dummy.next;
}
```

Time Complexity: O(n), where `n` is the number of digits (string/BigInteger operations are linear in digit count).
Space Complexity: O(n) — for the string representation and the new list.

**Walkthrough:** Using `1 -> 2 -> 9`:
- Traverse the list, appending digits: `sb = "129"`.
- Parse as `BigInteger`: `number = 129`. Add 1: `number = 130`.
- `resultStr = "130"`.
- Build a new list from `'1', '3', '0'`: `dummy -> 1 -> 3 -> 0`.
- Return `dummy.next` → `1 -> 3 -> 0`, matching the expected Output.

---

**Optimized Approach (Reverse-Based Carry Propagation):** Reverse the linked list so the least significant digit comes first. Traverse it, adding 1 to the first digit and propagating any carry to subsequent digits. If a final carry remains after processing all digits, prepend a new node with value `1`. Reverse the list back to restore the original digit order.

**Logic (Steps):**
1. Reverse the list with the `Reverse` helper so the least significant digit is now first (`reversedHead`).
2. Set `current = reversedHead` and `carry = 1` (since we're adding exactly 1).
3. While `current != null` and `carry > 0`: compute `sum = current.val + carry`, set `current.val = sum % 10`, update `carry = sum / 10`, track `prev = current`, and advance `current = current.next`.
4. If a `carry` remains after the loop (meaning every digit overflowed, e.g. `999 + 1`), append a new node with that `carry` value after `prev`.
5. Reverse the list again to restore the original most-significant-first digit order, and return it.

```csharp
public ListNode AddOneOptimal(ListNode head) {
    // Step 1: Reverse the list
    ListNode reversedHead = Reverse(head);

    // Step 2: Traverse and add 1 with carry propagation
    ListNode current = reversedHead;
    int carry = 1; // we are adding 1

    ListNode prev = null;
    while (current != null && carry > 0) {
        int sum = current.val + carry;
        current.val = sum % 10;
        carry = sum / 10;
        prev = current;
        current = current.next;
    }

    // Step 3: If carry remains after the last digit, append a new node (in reversed order)
    if (carry > 0 && prev != null) {
        prev.next = new ListNode(carry);
    }

    // Step 4: Reverse back to restore original order
    return Reverse(reversedHead);
}

private ListNode Reverse(ListNode head) {
    ListNode prev = null;
    ListNode current = head;
    while (current != null) {
        ListNode nextNode = current.next;
        current.next = prev;
        prev = current;
        current = nextNode;
    }
    return prev;
}
```

**Walkthrough:** Using the reverse-based approach on `1 -> 2 -> 9`:
- Reverse the list: `9 -> 2 -> 1`.
- Start with `carry = 1`. At node `9`: `sum = 9 + 1 = 10`, so `node.val = 0`, `carry = 1`.
- At node `2`: `sum = 2 + 1 = 3`, so `node.val = 3`, `carry = 0`. Since `carry` is now `0`, the loop stops (node `1` is left untouched).
- List is now `0 -> 3 -> 1` (in reversed order). No leftover `carry`, so no node is prepended.
- Reverse back: `1 -> 3 -> 0`, matching the expected Output.

---

**Optimized Approach (Recursive Carry Propagation):** Recurse to the end of the list first, then add the carry while unwinding the recursion stack, propagating it backward through the digits without physically reversing the list.

**Logic (Steps):**
1. Call the recursive helper `AddOneHelper(head)`, which returns the leftover `carry` after processing the whole list.
2. Inside the helper: base case — when `node == null` (past the last digit), return `1` (this is where the "+1" is actually introduced).
3. Otherwise, recurse first (`AddOneHelper(node.next)`) to process all digits to the right before touching the current node — this is what lets the carry propagate from least significant to most significant digit as the recursion unwinds.
4. On the way back up, add the returned `carry` to `node.val`, set `node.val = sum % 10`, and return `sum / 10` as the new carry for the caller.
5. Back in `AddOneRecursive`, if a `carry` remains after the top-level call returns (all digits overflowed), prepend a brand-new head node with that carry value; otherwise return `head` unchanged.

```csharp
public ListNode AddOneRecursive(ListNode head) {
    int carry = AddOneHelper(head);
    if (carry > 0) {
        ListNode newHead = new ListNode(carry);
        newHead.next = head;
        return newHead;
    }
    return head;
}

private int AddOneHelper(ListNode node) {
    if (node == null) {
        return 1; // base case: add 1 once we've passed the last digit
    }

    int carry = AddOneHelper(node.next);
    int sum = node.val + carry;
    node.val = sum % 10;
    return sum / 10;
}
```

Time Complexity: O(n), Space Complexity: O(1) for the reverse-based iterative approach; O(n) for the recursive approach due to the recursion call stack.

**Walkthrough:** Using `1 -> 2 -> 9`, call `AddOneHelper(1)`:
- `AddOneHelper(1)` recurses into `AddOneHelper(2)`, which recurses into `AddOneHelper(9)`, which recurses into `AddOneHelper(null)`.
- `AddOneHelper(null)` hits the base case and returns `carry = 1`.
- Unwinding to node `9`: `sum = 9 + 1 = 10` → `node.val = 0`, returns `carry = 1`.
- Unwinding to node `2`: `sum = 2 + 1 = 3` → `node.val = 3`, returns `carry = 0`.
- Unwinding to node `1`: `sum = 1 + 0 = 1` → `node.val = 1`, returns `carry = 0`.
- Top-level `carry = 0`, so no new node is prepended; return `head` unchanged in structure but with updated values: `1 -> 3 -> 0`, matching the expected Output.

---

## 5. Add Two Numbers Represented as Linked Lists

### 5. Add Two Numbers Represented as Linked Lists

**Problem Statement:** Given two numbers represented as linked lists where each node contains a single digit and the digits are stored such that the least significant digit is at the head of each list, add the two numbers and return the sum as a new linked list, in the same least-significant-digit-first order.

**Example:**
- Input: two numbers `2 -> 4 -> 3` (representing 342) and `5 -> 6 -> 4` (representing 465)
- Output: `7 -> 0 -> 8` (representing 807)
- Explanation: 342 + 465 = 807. Digit by digit from the least significant end: `2 + 5 = 7` (no carry), `4 + 6 = 10` → digit `0`, carry `1`, `3 + 4 + carry(1) = 8` (no carry). Result read from head: `7 -> 0 -> 8`.

**Brute Force Approach:** Convert each linked list into its represented decimal number (reading digits from head, which are least-significant-first, so reverse or accumulate accordingly), add the two numbers, then convert the resulting sum back into a new linked list in least-significant-first order.

**Logic (Steps):**
1. Convert `l1` and `l2` into `BigInteger` values via `ListToNumber`, which walks each list and prepends each digit to a `StringBuilder` (so the head, being least-significant, ends up last in the string — producing a normal most-significant-first numeric string).
2. Add the two `BigInteger` values together to get `sum`.
3. Convert `sum` to a string (most-significant-first) and reverse its characters to get least-significant-first order.
4. Build a new linked list (via `dummy`) from the reversed digit characters, one node per digit.
5. Return `dummy.next`.

```csharp
public ListNode AddTwoNumbersBrute(ListNode l1, ListNode l2) {
    System.Numerics.BigInteger num1 = ListToNumber(l1);
    System.Numerics.BigInteger num2 = ListToNumber(l2);
    System.Numerics.BigInteger sum = num1 + num2;

    string sumStr = sum.ToString();
    // sumStr is most-significant-first; reverse it to build the list least-significant-first
    char[] digitsReversed = sumStr.ToCharArray();
    System.Array.Reverse(digitsReversed);

    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    foreach (char c in digitsReversed) {
        current.next = new ListNode(c - '0');
        current = current.next;
    }

    return dummy.next;
}

private System.Numerics.BigInteger ListToNumber(ListNode head) {
    // head is least-significant-first, so build the digit string in reverse
    System.Text.StringBuilder sb = new System.Text.StringBuilder();
    while (head != null) {
        sb.Insert(0, head.val); // prepend to reverse order (most-significant-first string)
        head = head.next;
    }
    if (sb.Length == 0) return System.Numerics.BigInteger.Zero;
    return System.Numerics.BigInteger.Parse(sb.ToString());
}
```

Time Complexity: O(max(m, n)), where `m` and `n` are the digit counts of `l1` and `l2` (linear in list/number size).
Space Complexity: O(max(m, n)) — for building intermediate strings/numbers and the resulting list.

**Walkthrough:** Using `l1 = 2 -> 4 -> 3` (represents 342) and `l2 = 5 -> 6 -> 4` (represents 465):
- `ListToNumber(l1)` prepends `2`, then `4`, then `3` → string `"342"` → `num1 = 342`. Similarly `ListToNumber(l2)` gives `num2 = 465`.
- `sum = 342 + 465 = 807`.
- `sumStr = "807"`, reversed → `['7', '0', '8']`.
- Build the new list from these reversed digits: `dummy -> 7 -> 0 -> 8`.
- Return `dummy.next` → `7 -> 0 -> 8`, matching the expected Output.

---

**Optimized Approach (Dummy Node + Carry Simulation):** Use a dummy head node to simplify list construction. Traverse both lists simultaneously, adding corresponding digits along with any carry from the previous step, creating a new node for each resulting digit. Continue until both lists are exhausted and there is no carry left.

**Logic (Steps):**
1. Create a `dummy` node and a `current` pointer starting at `dummy`, plus `carry = 0`.
2. Loop while `l1 != null || l2 != null || carry != 0` (continue as long as there are digits left in either list or a carry still to flush out).
3. In each iteration, read `val1` from `l1` (or `0` if `l1` is exhausted) and `val2` from `l2` (or `0` if exhausted), compute `sum = val1 + val2 + carry`, then split it into `carry = sum / 10` and `digit = sum % 10`.
4. Append a new node holding `digit` to `current.next`, and advance `current`, `l1`, and `l2` (the latter two only if not already `null`).
5. Return `dummy.next` once the loop ends.

```csharp
public ListNode AddTwoNumbersOptimal(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    int carry = 0;

    while (l1 != null || l2 != null || carry != 0) {
        int val1 = (l1 != null) ? l1.val : 0;
        int val2 = (l2 != null) ? l2.val : 0;

        int sum = val1 + val2 + carry;
        carry = sum / 10;
        int digit = sum % 10;

        current.next = new ListNode(digit);
        current = current.next;

        if (l1 != null) l1 = l1.next;
        if (l2 != null) l2 = l2.next;
    }

    return dummy.next;
}
```

Time Complexity: O(n), Space Complexity: O(1) (excluding the output list, which is required regardless).

**Walkthrough:** Dry run of the carry simulation on `l1 = 2 -> 4 -> 3` (342) and `l2 = 5 -> 6 -> 4` (465):
- Initialize `dummy -> null`, `current = dummy`, `carry = 0`.
- Iteration 1: `val1 = 2`, `val2 = 5`. `sum = 2 + 5 + 0 = 7`. `carry = 0`, `digit = 7`. Append node `7`. List so far: `dummy -> 7`. Advance `l1` to `4`, `l2` to `6`.
- Iteration 2: `val1 = 4`, `val2 = 6`. `sum = 4 + 6 + 0 = 10`. `carry = 1`, `digit = 0`. Append node `0`. List so far: `dummy -> 7 -> 0`. Advance `l1` to `3`, `l2` to `4`.
- Iteration 3: `val1 = 3`, `val2 = 4`. `sum = 3 + 4 + 1 = 8`. `carry = 0`, `digit = 8`. Append node `8`. List so far: `dummy -> 7 -> 0 -> 8`. Advance `l1` to `null`, `l2` to `null`.
- Loop condition check: `l1 == null`, `l2 == null`, `carry == 0` → loop terminates.
- Return `dummy.next` → `7 -> 0 -> 8`, matching the expected Output (representing 807 = 342 + 465).
