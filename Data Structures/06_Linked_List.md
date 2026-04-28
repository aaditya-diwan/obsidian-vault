# SECTION 6 — LINKED LIST (Problems 56–65)

> **Core insight:** Linked list problems reduce to pointer manipulation. Master the "dummy node" and "two-pointer (slow/fast)" techniques.

---

## Problem 56 — Reverse Linked List

**Difficulty:** Easy
**Real-world intuition:** Undo operations in text editors, reversing a command queue, reflection operations in graphics.

---

### Theory

- **Core concept:** Redirect each node's `next` pointer to the previous node.
- **Pattern:** Three-pointer iteration (prev, curr, next).
- **When to use:** Any list reversal or reversal-in-parts.
- **Common mistakes:**
  - Losing `curr.next` before reassigning it.
  - Not returning `prev` (which becomes the new head).

---

### Iterative Java Implementation

```java
public class ReverseLinkedList {
    public ListNode reverseList(ListNode head) {
        ListNode prev = null, curr = head;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }
        return prev;
    }
}
```

### Recursive Java Implementation

```java
public ListNode reverseListRecursive(ListNode head) {
    if (head == null || head.next == null) return head;
    ListNode newHead = reverseListRecursive(head.next);
    head.next.next = head;
    head.next = null;
    return newHead;
}
```

- **Time:** O(n), **Space:** O(1) iterative / O(n) recursive.

---

### Variations

1. **Reverse Linked List II (range):** Reverse only nodes from position m to n.
2. **Reverse Nodes in k-Group:** Reverse every k consecutive nodes.
3. **Palindrome Linked List:** Reverse second half, compare.

---

## Problem 57 — Detect Cycle in Linked List

**Difficulty:** Easy
**Real-world intuition:** Detecting infinite loops in workflow engines, circular dependencies in build systems.

---

### Theory

- **Core concept:** Floyd's cycle detection — slow pointer moves 1 step, fast moves 2. If they meet, there's a cycle.
- **Pattern:** Slow-fast pointers.
- **When to use:** Cycle detection in any sequence.
- **Common mistakes:**
  - Not checking `fast != null && fast.next != null` before advancing (NullPointerException).

---

### Java Implementation

```java
public class DetectCycle {
    public boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }
        return false;
    }

    // Follow-up: find cycle start node
    public ListNode detectCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) {
                slow = head;
                while (slow != fast) { slow = slow.next; fast = fast.next; }
                return slow;
            }
        }
        return null;
    }
}
```

---

## Problem 58 — Merge Two Sorted Lists

**Difficulty:** Easy
**Real-world intuition:** Merging sorted log files, combining sorted search result pages, merge step in merge sort.

---

### Theory

- **Pattern:** Dummy head node to simplify edge cases + greedy merge.
- **When to use:** Merging sorted structures.
- **Common mistakes:**
  - Not attaching the remaining non-null list at the end.
  - Trying to merge without a dummy node (complex null checks).

---

### Java Implementation

```java
public class MergeTwoSortedLists {
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;

        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
            else { curr.next = l2; l2 = l2.next; }
            curr = curr.next;
        }
        curr.next = (l1 != null) ? l1 : l2;
        return dummy.next;
    }
}
```

---

### Variations

1. **Merge K Sorted Lists:** Use a min-heap (Problem 79).
2. **Sort List:** Merge sort using this merge function.

---

## Problem 59 — Remove Nth Node From End

**Difficulty:** Medium
**Real-world intuition:** Maintaining a rolling history of N events and removing the oldest — used in LRU cache and session management.

---

### Theory

- **Core concept:** Two pointers with a gap of n. When fast reaches the end, slow is at the node before the target.
- **Pattern:** Two-pointer with fixed gap.
- **When to use:** "Nth from end" without knowing the length.
- **Common mistakes:**
  - Not using a dummy head (edge case: removing the first node).
  - Gap of n vs n+1 — fast pointer should be n+1 steps ahead so slow lands one before target.

---

### Java Implementation

```java
public class RemoveNthFromEnd {
    public ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode fast = dummy, slow = dummy;

        for (int i = 0; i <= n; i++) fast = fast.next;

        while (fast != null) {
            slow = slow.next;
            fast = fast.next;
        }

        slow.next = slow.next.next;
        return dummy.next;
    }
}
```

---

### Dry Run

**Input:** `1->2->3->4->5`, n=2

- fast advances n+1=3 steps: fast=node(3), slow=dummy.
- Iterate: slow=1,fast=4 → slow=2,fast=5 → slow=3,fast=null.
- `slow.next = slow.next.next` → node(3).next = node(5). Removed node(4).

**Output:** `1->2->3->5` ✓

---

## Problem 60 — Add Two Numbers

**Difficulty:** Medium
**Real-world intuition:** Big integer arithmetic in cryptography and financial systems where numbers exceed 64-bit integers.

---

### Theory

- **Core concept:** Simulate digit-by-digit addition with carry, like elementary school math.
- **Pattern:** Simultaneous traversal + carry propagation.
- **Common mistakes:**
  - Forgetting the final carry (if sum overflows to one more digit).
  - Not handling lists of different lengths.

---

### Java Implementation

```java
public class AddTwoNumbers {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode curr = dummy;
        int carry = 0;

        while (l1 != null || l2 != null || carry != 0) {
            int sum = carry;
            if (l1 != null) { sum += l1.val; l1 = l1.next; }
            if (l2 != null) { sum += l2.val; l2 = l2.next; }
            carry = sum / 10;
            curr.next = new ListNode(sum % 10);
            curr = curr.next;
        }

        return dummy.next;
    }
}
```

---

## Problem 61 — Copy List with Random Pointer

**Difficulty:** Medium
**Real-world intuition:** Deep copying complex graph-like structures in serialization, game state cloning, object persistence.

---

### Theory

- **Core concept:** Two passes — first create all new nodes, then assign next and random pointers.
- **Pattern:** HashMap (original node → copy node) for O(1) random pointer lookup.
- **Common mistakes:**
  - Trying to do it in one pass (random pointer may point to a node not yet created).

---

### Java Implementation

```java
import java.util.HashMap;

public class CopyListRandomPointer {
    public Node copyRandomList(Node head) {
        if (head == null) return null;
        HashMap<Node, Node> map = new HashMap<>();

        Node curr = head;
        while (curr != null) { map.put(curr, new Node(curr.val)); curr = curr.next; }

        curr = head;
        while (curr != null) {
            map.get(curr).next = map.get(curr.next);
            map.get(curr).random = map.get(curr.random);
            curr = curr.next;
        }

        return map.get(head);
    }
}
```

- **Time:** O(n), **Space:** O(n)

**O(1) space variant:** Weave copies into original list, set randoms, then unweave.

---

## Problem 62 — LRU Cache

**Difficulty:** Medium
**Real-world intuition:** Operating system page replacement, browser history, CPU cache management — evict least recently used item when capacity is reached.

---

### Theory

- **Core concept:** O(1) get and put. Use HashMap for O(1) lookup + doubly linked list for O(1) insertion/deletion.
- **Pattern:** HashMap + Doubly Linked List.
- **When to use:** Any "cache with eviction policy" problem.
- **Common mistakes:**
  - Using a singly linked list (can't delete in O(1) without prev pointer).
  - Forgetting to update the list order on every `get` (not just `put`).

---

### Java Implementation

```java
import java.util.HashMap;

public class LRUCache {
    private final int capacity;
    private final HashMap<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);  // dummy head (MRU side)
    private final Node tail = new Node(0, 0);  // dummy tail (LRU side)

    private static class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        remove(node);
        insertFront(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) remove(map.get(key));
        if (map.size() == capacity) {
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
        Node node = new Node(key, value);
        map.put(key, node);
        insertFront(node);
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }

    private void insertFront(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }
}
```

- **Time:** O(1) for get and put. **Space:** O(capacity).

---

## Problem 63 — Intersection of Two Linked Lists

**Difficulty:** Easy
**Real-world intuition:** Finding shared resources between two dependency trees, detecting forked code merging to the same commit.

---

### Theory

- **Core concept:** Two pointers that switch lists when they reach the end — both traverse exactly `m+n` nodes before meeting at the intersection (or null).
- **Pattern:** Dual list traversal.
- **Why it works:** Both pointers travel `len(A) + len(B)` total. If there's an intersection, they're in sync when they reach it.

---

### Java Implementation

```java
public class IntersectionLinkedLists {
    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode a = headA, b = headB;
        while (a != b) {
            a = (a == null) ? headB : a.next;
            b = (b == null) ? headA : b.next;
        }
        return a;
    }
}
```

---

## Problem 64 — Palindrome Linked List

**Difficulty:** Easy
**Real-world intuition:** Validating symmetric data structures in network protocol parsing, detecting symmetric sequences in bioinformatics.

---

### Theory

- **Core concept:** Find midpoint, reverse second half, compare with first half.
- **Pattern:** Slow-fast pointer for midpoint + list reversal.
- **Common mistakes:**
  - Off-by-one in finding midpoint (even vs odd length).
  - Not handling single-node list.

---

### Java Implementation

```java
public class PalindromeLinkedList {
    public boolean isPalindrome(ListNode head) {
        // Find midpoint
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Reverse second half
        ListNode prev = null, curr = slow;
        while (curr != null) {
            ListNode next = curr.next;
            curr.next = prev;
            prev = curr;
            curr = next;
        }

        // Compare
        ListNode left = head, right = prev;
        while (right != null) {
            if (left.val != right.val) return false;
            left = left.next;
            right = right.next;
        }
        return true;
    }
}
```

---

## Problem 65 — Reorder List

**Difficulty:** Medium
**Real-world intuition:** Interleaving two data streams — alternating between newest and oldest records in a timeline view.

---

### Theory

- **Core concept:** Merge first half with reversed second half alternately.
- **Pattern:** Find middle + reverse second half + merge two lists.
- **Steps:**
  1. Find midpoint (slow-fast).
  2. Reverse second half.
  3. Merge: take one from each alternately.

---

### Java Implementation

```java
public class ReorderList {
    public void reorderList(ListNode head) {
        if (head == null || head.next == null) return;

        // Find midpoint
        ListNode slow = head, fast = head;
        while (fast.next != null && fast.next.next != null) {
            slow = slow.next;
            fast = fast.next.next;
        }

        // Reverse second half
        ListNode second = slow.next;
        slow.next = null;
        ListNode prev = null;
        while (second != null) {
            ListNode next = second.next;
            second.next = prev;
            prev = second;
            second = next;
        }

        // Merge
        ListNode first = head;
        second = prev;
        while (second != null) {
            ListNode tmp1 = first.next, tmp2 = second.next;
            first.next = second;
            second.next = tmp1;
            first = tmp1;
            second = tmp2;
        }
    }
}
```

---

## Linked List — Pattern Summary

| Pattern | Trigger | Example |
|---------|---------|---------|
| Dummy head node | "edge case: remove head" | Remove Nth, Merge Lists |
| Slow-fast pointers | "cycle, midpoint, kth from end" | Detect Cycle, Palindrome |
| Three-pointer reversal | "reverse list or portion" | Reverse List |
| HashMap for node lookup | "copy with complex pointers" | Copy Random List |
| HashMap + DLL | "O(1) cache with ordering" | LRU Cache |
| Dual list traversal | "intersection of two lists" | Intersection |

### How to Identify

- "nth from end / midpoint / cycle" → slow-fast pointers.
- "In-place modification, no extra space" → pointer rewiring.
- "Copy complex structure" → HashMap first pass.
- "Cache / eviction" → LRU = HashMap + DLL.

---

## Revision Checklist — After Problems 51–60

- [ ] Can you reverse a list iteratively AND recursively?
- [ ] Do you use a dummy head for remove/merge operations?
- [ ] Can you find the cycle entry point (phase 2 of Floyd's)?
- [ ] LRU Cache: do you update on `get` as well as `put`?
- [ ] Can you code the palindrome check without drawing it?

### Mixed Practice

1. **Reverse Nodes in k-Group:** Recursive list reversal in chunks.
2. **Sort List:** Merge sort on a linked list.
3. **Swap Nodes in Pairs:** Fundamental pointer manipulation exercise.

---

