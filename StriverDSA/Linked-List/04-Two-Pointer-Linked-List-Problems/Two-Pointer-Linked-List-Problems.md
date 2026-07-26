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

**Optimized Approach:** Use the slow and fast pointer technique. Move `fast` two steps and `slow` one step at a time, but start `fast` two steps ahead of `slow` (i.e., `fast = head.next.next`, `slow = head`). When `fast` reaches the end, `slow` is positioned exactly at the node just before the middle, so it can be unlinked directly in a single pass.

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

**Explanation:** By starting `fast` two nodes ahead of `slow`, when `fast` finishes traversing the list, `slow` has advanced only half as many steps and lands precisely one node before the true middle. This lets us delete the middle node without a separate counting pass or needing to know `n` beforehand — a single traversal suffices.

---

## 2. Find the Intersection Point of Two Linked Lists (Y-shaped list)

### 2. Find the Intersection Point of Two Linked Lists (Y-shaped list)

**Problem Statement:** Given the heads of two singly linked lists, `headA` and `headB`, that may or may not intersect at some node (forming a Y-shape by sharing a common tail), find and return the node at which the two lists intersect. If the two lists have no intersection, return `null`. The intersection is determined by reference (node identity), not by value.

**Example:**
- Input: List A: `4 -> 1 -> 8 -> 4 -> 5`, List B: `5 -> 6 -> 1 -> 8 -> 4 -> 5`, where both lists share the tail starting at node with value `8` (so `8 -> 4 -> 5` is common).
- Output: Node with value `8` (the first common node).
- Explanation: List A has nodes `4, 1` before merging into the shared segment `8, 4, 5`. List B has nodes `5, 6, 1` before merging into the same shared segment. The intersection point is the node `8` because that is the first node reference shared by both lists.

**Brute Force Approach:** For every node in list A, traverse the entirety of list B and compare node references. If a match is found, that is the intersection point.

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

**Optimized Approach (Length Difference):** Compute the lengths of both lists. Advance the pointer of the longer list by the length difference so both pointers have an equal number of remaining nodes to traverse. Then move both pointers together one step at a time; the node at which they become equal is the intersection point.

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

**Optimized Approach ("Switch Heads" Trick):** Use two pointers, one starting at `headA` and the other at `headB`. Advance both one step at a time. When a pointer reaches the end of its list (`null`), redirect it to the head of the *other* list. Continue this until the two pointers point to the same node — that node is the intersection (or both become `null` simultaneously if there is no intersection).

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

**Explanation:** Dry run of the "switch heads" trick on the example, where list A (`4 -> 1 -> 8 -> 4 -> 5`) has length 5 and list B (`5 -> 6 -> 1 -> 8 -> 4 -> 5`) has length 6, with the common tail `8 -> 4 -> 5` (3 nodes):

- `ptrA` starts at `headA` (value `4`), `ptrB` starts at `headB` (value `5`).
- Step by step, `ptrA` traverses A's 5 nodes (`4, 1, 8, 4, 5`), then hits `null` and switches to `headB`.
- Meanwhile `ptrB` traverses B's 6 nodes (`5, 6, 1, 8, 4, 5`), then hits `null` and switches to `headA`.
- `ptrA` total path length until reaching the intersection the second time: `lenA (5) + lenB_beforeIntersection (3)` = 8 steps.
- `ptrB` total path length until reaching the intersection: `lenB (6) + lenA_beforeIntersection (2)` = 8 steps.
- Both pointers have now traveled exactly `lenA + lenB = 11` combined "unique" steps and, because they switch lists on reaching `null`, they arrive at the shared node `8` at the exact same step count.
- Since both pointers traverse a total distance of `lenA + lenB` before meeting, and the extra distance difference (`|lenA - lenB|`) gets absorbed by the switch, they are perfectly synchronized to meet exactly at the first common node, `8`. If the lists never intersect, both pointers become `null` at the same time (after `lenA + lenB` steps), and the loop exits returning `null`.

This trick elegantly avoids the need to explicitly compute lengths — the "switch" implicitly equalizes the distance traveled by both pointers.

---

## 3. Remove the Nth Node from the End of a Linked List

### 3. Remove the Nth Node from the End of a Linked List

**Problem Statement:** Given the head of a singly linked list and an integer `n`, remove the `n`-th node from the end of the list and return the head of the resulting list.

**Example:**
- Input: `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`
- Output: `1 -> 2 -> 3 -> 5`
- Explanation: Counting from the end, the 2nd node from the end is `4` (order from end: `5` is 1st, `4` is 2nd). Removing it links `3` directly to `5`.

**Brute Force Approach:** Traverse the list once to compute its length `L`. The node to remove is the `(L - n + 1)`-th node from the beginning (1-indexed). Traverse again to the node just before it and unlink it. Handle the edge case where the head itself must be removed (when `n == L`).

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

**Optimized Approach:** Use a dummy node pointing to `head` to simplify edge cases (like removing the head). Use two pointers, `fast` and `slow`, both starting at the dummy node. Move `fast` ahead by `n` steps first. Then move both `fast` and `slow` together until `fast` reaches the last node. At that point, `slow` is positioned just before the node to be removed.

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

**Explanation:** Dry run of the two-pointer N-gap technique on `1 -> 2 -> 3 -> 4 -> 5`, `n = 2`:

- `dummy -> 1 -> 2 -> 3 -> 4 -> 5`. `fast = dummy`, `slow = dummy`.
- Move `fast` ahead by `n = 2` steps: `fast` goes from `dummy` to `1` to `2`. Now `fast` is at node `2`, `slow` is still at `dummy`.
- Now advance both together while `fast.next != null`:
  - `fast.next` is `3` (not null) → move: `fast = 3`, `slow = 1`.
  - `fast.next` is `4` (not null) → move: `fast = 4`, `slow = 2`.
  - `fast.next` is `5` (not null) → move: `fast = 5`, `slow = 3`.
  - `fast.next` is `null` → stop.
- Now `slow` is at node `3`, and `slow.next` is node `4`, which is exactly the 2nd node from the end.
- Unlink it: `slow.next = slow.next.next` makes `3.next = 5`.
- Result: `dummy -> 1 -> 2 -> 3 -> 5`, return `dummy.next` → `1 -> 2 -> 3 -> 5`.

The fixed gap of `n` nodes maintained between `fast` and `slow` ensures that when `fast` reaches the last node, `slow` is exactly `n` nodes behind it — precisely the node before the one to be removed.

---

## 4. Add 1 to a Number Represented as a Linked List

### 4. Add 1 to a Number Represented as a Linked List

**Problem Statement:** Given a non-negative integer represented as a singly linked list of digits (the most significant digit is at the head of the list), add 1 to the number and return the head of the resulting list. Assume no leading zeros in the input, except when the number itself is `0`.

**Example:**
- Input: `1 -> 2 -> 9`  (represents 129)
- Output: `1 -> 3 -> 0`  (129 + 1 = 130)
- Explanation: Adding 1 to 129 gives 130. Since the last digit `9` overflows to `0` with a carry, and that carry propagates to `2`, making it `3`. The first digit `1` is unaffected.

**Brute Force Approach:** Traverse the list to build the decimal number represented by the digits (be careful with very large numbers — using `long`/`BigInteger` in real scenarios), add 1 to it, then convert the resulting number back into a new linked list.

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

**Optimized Approach (Reverse-Based Carry Propagation):** Reverse the linked list so the least significant digit comes first. Traverse it, adding 1 to the first digit and propagating any carry to subsequent digits. If a final carry remains after processing all digits, prepend a new node with value `1`. Reverse the list back to restore the original digit order.

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

**Optimized Approach (Recursive Carry Propagation):** Recurse to the end of the list first, then add the carry while unwinding the recursion stack, propagating it backward through the digits without physically reversing the list.

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

**Explanation:** Using the reverse-based approach on `1 -> 2 -> 9`:

- Reverse the list: `9 -> 2 -> 1`.
- Start with `carry = 1`. At node `9`: `sum = 9 + 1 = 10`, so `node.val = 0`, `carry = 1`.
- At node `2`: `sum = 2 + 1 = 3`, so `node.val = 3`, `carry = 0`. Since `carry` is now `0`, the loop stops (node `1` is left untouched).
- List is now `0 -> 3 -> 1` (in reversed order).
- Reverse back: `1 -> 3 -> 0`, which is the correct result representing `130`.

Reversing lets us process digits from least significant to most significant using simple forward traversal, which mirrors how addition-with-carry naturally works on paper (right to left), then we reverse again to restore the original most-significant-first order.

---

## 5. Add Two Numbers Represented as Linked Lists

### 5. Add Two Numbers Represented as Linked Lists

**Problem Statement:** Given two numbers represented as linked lists where each node contains a single digit and the digits are stored such that the least significant digit is at the head of each list, add the two numbers and return the sum as a new linked list, in the same least-significant-digit-first order.

**Example:**
- Input: two numbers `2 -> 4 -> 3` (representing 342) and `5 -> 6 -> 4` (representing 465)
- Output: `7 -> 0 -> 8` (representing 807)
- Explanation: 342 + 465 = 807. Digit by digit from the least significant end: `2 + 5 = 7` (no carry), `4 + 6 = 10` → digit `0`, carry `1`, `3 + 4 + carry(1) = 8` (no carry). Result read from head: `7 -> 0 -> 8`.

**Brute Force Approach:** Convert each linked list into its represented decimal number (reading digits from head, which are least-significant-first, so reverse or accumulate accordingly), add the two numbers, then convert the resulting sum back into a new linked list in least-significant-first order.

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

**Optimized Approach (Dummy Node + Carry Simulation):** Use a dummy head node to simplify list construction. Traverse both lists simultaneously, adding corresponding digits along with any carry from the previous step, creating a new node for each resulting digit. Continue until both lists are exhausted and there is no carry left.

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

**Explanation:** Dry run of the carry simulation on `l1 = 2 -> 4 -> 3` (342) and `l2 = 5 -> 6 -> 4` (465):

- Initialize `dummy -> null`, `current = dummy`, `carry = 0`.
- Iteration 1: `val1 = 2`, `val2 = 5`. `sum = 2 + 5 + 0 = 7`. `carry = 0`, `digit = 7`. Append node `7`. List so far: `dummy -> 7`. Advance `l1` to `4`, `l2` to `6`.
- Iteration 2: `val1 = 4`, `val2 = 6`. `sum = 4 + 6 + 0 = 10`. `carry = 1`, `digit = 0`. Append node `0`. List so far: `dummy -> 7 -> 0`. Advance `l1` to `3`, `l2` to `4`.
- Iteration 3: `val1 = 3`, `val2 = 4`. `sum = 3 + 4 + 1 = 8`. `carry = 0`, `digit = 8`. Append node `8`. List so far: `dummy -> 7 -> 0 -> 8`. Advance `l1` to `null`, `l2` to `null`.
- Loop condition check: `l1 == null`, `l2 == null`, `carry == 0` → loop terminates.
- Return `dummy.next` → `7 -> 0 -> 8`, representing 807, which matches `342 + 465 = 807`.

The carry variable naturally accumulates overflow from each digit-wise addition and is folded into the next position's sum, exactly mimicking manual column addition, while the dummy node avoids special-casing the creation of the first result node.
