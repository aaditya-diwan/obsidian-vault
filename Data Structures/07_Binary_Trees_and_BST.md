# SECTION 7 — BINARY TREES & BST (Problems 66–75)

> **Core insight:** Most tree problems are DFS (recursion) or BFS (level-order). Learn to think in terms of what each node returns upward.

---

## Problem 66 — Maximum Depth of Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Calculating the height of a file system directory tree, depth of organizational hierarchy.

---

### Theory

- **Core concept:** Max depth = 1 + max(depth(left), depth(right)).
- **Pattern:** Post-order DFS (compute children first, then use results at current node).
- **When to use:** Any "height / depth" computation.

---

### Java Implementation

```java
public class MaxDepth {
    public int maxDepth(TreeNode root) {
        if (root == null) return 0;
        return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
    }
}
```

- **Time:** O(n), **Space:** O(h) recursion stack.

---

### Variations

1. **Minimum Depth:** Watch out — min depth must reach a leaf, not just a null child.
2. **Balanced Binary Tree:** Check `|depth(left) - depth(right)| <= 1` at every node.
3. **Diameter of Binary Tree:** Problem 75 below.

---

## Problem 67 — Invert Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Mirror transformation in graphics, reversing decision trees in AI.

---

### Java Implementation

```java
public class InvertTree {
    public TreeNode invertTree(TreeNode root) {
        if (root == null) return null;
        TreeNode temp = root.left;
        root.left = invertTree(root.right);
        root.right = invertTree(temp);
        return root;
    }
}
```

---

## Problem 68 — Binary Tree Level Order Traversal

**Difficulty:** Medium
**Real-world intuition:** BFS in social networks (degrees of separation), layer-by-layer processing of hierarchical data.

---

### Theory

- **Core concept:** BFS using a queue; process all nodes at current depth before moving to next level.
- **Pattern:** BFS with level-size tracking.
- **When to use:** Any "level by level" or "layer" tree problem.
- **Common mistakes:**
  - Using `queue.size()` in the loop condition instead of capturing it before the inner loop (size changes during inner loop).

---

### Java Implementation

```java
import java.util.*;

public class LevelOrderTraversal {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int size = queue.size();  // capture before inner loop
            List<Integer> level = new ArrayList<>();
            for (int i = 0; i < size; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);
                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }
            result.add(level);
        }

        return result;
    }
}
```

---

### Variations

1. **Zigzag Level Order:** Alternate direction each level.
2. **Right Side View:** Take last element of each level.
3. **Average of Levels:** Average each level list.

---

## Problem 69 — Validate BST

**Difficulty:** Medium
**Real-world intuition:** Database B-tree integrity checks, validating sorted data structures.

---

### Theory

- **Core concept:** Each node must be strictly within a valid range. Left subtree: max = current node value. Right subtree: min = current node value.
- **Pattern:** DFS with min/max bounds passed down.
- **Common mistakes:**
  - Only checking `root.left.val < root.val` (not a recursive guarantee — a deeper node may violate BST property).
  - Using `Integer.MIN/MAX_VALUE` as default bounds (fails for INT_MIN or INT_MAX as node values — use `null` or `Long`).

---

### Java Implementation

```java
public class ValidateBST {
    public boolean isValidBST(TreeNode root) {
        return validate(root, null, null);
    }

    private boolean validate(TreeNode node, Integer min, Integer max) {
        if (node == null) return true;
        if (min != null && node.val <= min) return false;
        if (max != null && node.val >= max) return false;
        return validate(node.left, min, node.val) && validate(node.right, node.val, max);
    }
}
```

---

### Dry Run

**Input:**
```
    5
   / \
  1   4
     / \
    3   6
```

- validate(5, null, null): valid range OK.
- validate(1, null, 5): OK.
- validate(4, 5, null): 4 <= 5 → FALSE.

**Output:** `false` ✓

---

## Problem 70 — Lowest Common Ancestor of a Binary Tree

**Difficulty:** Medium
**Real-world intuition:** Finding the common ancestor in an organizational chart, the base class in OOP hierarchy, or the merge point in version control.

---

### Theory

- **Core concept:** If both p and q are in different subtrees of a node, that node is the LCA. If one is the ancestor of the other, it is the LCA.
- **Pattern:** Post-order DFS — check left and right subtrees, then decide at current node.
- **Return logic:** If both subtrees return non-null, current node is LCA.

---

### Java Implementation

```java
public class LowestCommonAncestor {
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) return root;
        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);
        if (left != null && right != null) return root;
        return left != null ? left : right;
    }
}
```

- **Time:** O(n), **Space:** O(h)

**LCA in BST:** If both values < root, go left; if both > root, go right; else root is LCA.

---

## Problem 71 — Binary Tree Maximum Path Sum

**Difficulty:** Hard
**Real-world intuition:** Finding the highest-value route in a logistics network modeled as a tree. Maximizing pipeline throughput.

---

### Theory

- **Core concept:** A path can go through any node as the "top" (turning point). The contribution of a subtree is `max(0, maxPathFromChild)` — never take a negative branch.
- **Pattern:** Post-order DFS; each call returns the best single-branch extension (for parent to use), while globally tracking the best path-through-node.
- **Common mistakes:**
  - Confusing "path through root" with "path anywhere in tree" — use a global variable.
  - Not clamping negative subtree contributions to 0.

---

### Java Implementation

```java
public class BinaryTreeMaxPathSum {
    private int maxSum = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        dfs(root);
        return maxSum;
    }

    private int dfs(TreeNode node) {
        if (node == null) return 0;
        int left = Math.max(0, dfs(node.left));
        int right = Math.max(0, dfs(node.right));
        maxSum = Math.max(maxSum, node.val + left + right);  // path through this node
        return node.val + Math.max(left, right);              // best single branch for parent
    }
}
```

---

### Dry Run

**Input:**
```
   -10
   /  \
  9   20
     /  \
    15   7
```

- dfs(9): left=0, right=0. maxSum=max(MIN,9)=9. return 9.
- dfs(15): return 15. dfs(7): return 7.
- dfs(20): left=15, right=7. maxSum=max(9,20+15+7)=42. return 20+15=35.
- dfs(-10): left=9→max(0,9)=9, right=35. maxSum=max(42,-10+9+35)=max(42,34)=42. return -10+35=25.

**Output:** `42` ✓

---

## Problem 72 — Serialize and Deserialize Binary Tree

**Difficulty:** Hard
**Real-world intuition:** Persisting tree structures to disk or sending over a network — databases, distributed systems, JSON/XML tree storage.

---

### Theory

- **Core concept:** Pre-order traversal for serialization (root first, then left, then right). Use `null` markers to distinguish structure.
- **Pattern:** Recursive pre-order with null markers.
- **When to use:** Any tree persistence/reconstruction problem.

---

### Java Implementation

```java
import java.util.Arrays;
import java.util.LinkedList;
import java.util.Queue;

public class SerializeDeserialize {
    public String serialize(TreeNode root) {
        if (root == null) return "null";
        return root.val + "," + serialize(root.left) + "," + serialize(root.right);
    }

    public TreeNode deserialize(String data) {
        Queue<String> queue = new LinkedList<>(Arrays.asList(data.split(",")));
        return buildTree(queue);
    }

    private TreeNode buildTree(Queue<String> queue) {
        String val = queue.poll();
        if ("null".equals(val)) return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = buildTree(queue);
        node.right = buildTree(queue);
        return node;
    }
}
```

---

## Problem 73 — Kth Smallest Element in BST

**Difficulty:** Medium
**Real-world intuition:** Finding the kth percentile entry in a sorted database index, the kth cheapest product in an ordered catalog.

---

### Theory

- **Core concept:** In-order traversal of BST produces elements in sorted order.
- **Pattern:** In-order DFS with counter.
- **Optimization:** Morris traversal for O(1) space.

---

### Java Implementation

```java
public class KthSmallestBST {
    private int count = 0, result = 0;

    public int kthSmallest(TreeNode root, int k) {
        inorder(root, k);
        return result;
    }

    private void inorder(TreeNode node, int k) {
        if (node == null) return;
        inorder(node.left, k);
        if (++count == k) { result = node.val; return; }
        inorder(node.right, k);
    }
}
```

---

## Problem 74 — Construct Binary Tree from Preorder and Inorder

**Difficulty:** Medium
**Real-world intuition:** Reconstructing a tree from two complementary traversals — used in tree serialization standards.

---

### Theory

- **Core concept:** Preorder[0] is always the root. Find it in inorder — everything to its left is the left subtree, right is the right subtree.
- **Pattern:** Recursive divide-and-conquer with HashMap for O(1) inorder index lookup.
- **Common mistakes:**
  - Recomputing inorder index by scanning (O(n²)) instead of using HashMap (O(1)).

---

### Java Implementation

```java
import java.util.HashMap;

public class BuildTreePreorderInorder {
    private HashMap<Integer, Integer> inorderIndex = new HashMap<>();

    public TreeNode buildTree(int[] preorder, int[] inorder) {
        for (int i = 0; i < inorder.length; i++) inorderIndex.put(inorder[i], i);
        return build(preorder, 0, preorder.length - 1, 0, inorder.length - 1);
    }

    private TreeNode build(int[] pre, int preLeft, int preRight, int inLeft, int inRight) {
        if (preLeft > preRight) return null;
        int rootVal = pre[preLeft];
        int mid = inorderIndex.get(rootVal);
        int leftSize = mid - inLeft;

        TreeNode root = new TreeNode(rootVal);
        root.left = build(pre, preLeft + 1, preLeft + leftSize, inLeft, mid - 1);
        root.right = build(pre, preLeft + leftSize + 1, preRight, mid + 1, inRight);
        return root;
    }
}
```

- **Time:** O(n), **Space:** O(n)

---

## Problem 75 — Diameter of Binary Tree

**Difficulty:** Easy
**Real-world intuition:** Finding the longest network path in a tree-structured network topology.

---

### Theory

- **Core concept:** The diameter = max over all nodes of `depth(left) + depth(right)`.
- **Pattern:** Post-order DFS tracking global maximum.
- **Common mistakes:**
  - Returning the diameter from the recursion instead of the depth (breaks the recursion contract).

---

### Java Implementation

```java
public class DiameterBinaryTree {
    private int maxDiameter = 0;

    public int diameterOfBinaryTree(TreeNode root) {
        depth(root);
        return maxDiameter;
    }

    private int depth(TreeNode node) {
        if (node == null) return 0;
        int left = depth(node.left);
        int right = depth(node.right);
        maxDiameter = Math.max(maxDiameter, left + right);
        return 1 + Math.max(left, right);
    }
}
```

---

## Binary Trees & BST — Pattern Summary

| Pattern | When to use | Return from DFS |
|---------|-------------|-----------------|
| Post-order DFS | Compute something from subtrees | Result for parent to use |
| Pre-order DFS | Construct / serialize | N/A |
| In-order DFS | BST sorted traversal | N/A |
| BFS (level order) | Layer-by-layer processing | Per-level results |
| Global variable + DFS | Track max/min over all paths | Only depth/contribution |
| Min/max bounds (BST validation) | Validate BST property | true/false |

### How to Identify

- "Compute at each node using children" → post-order.
- "Level by level" → BFS.
- "BST sorted property" → in-order.
- "Path sum / diameter" → global variable + post-order DFS.
- "Reconstruct tree" → pre-order or divide-and-conquer with inorder index.

---

## Revision Checklist — After Problems 61–70

- [ ] Max depth, invert, LCA — can you code all three from memory?
- [ ] Validate BST: do you pass bounds, not just compare children?
- [ ] Level order: do you capture `queue.size()` before the inner loop?
- [ ] Max path sum: do you clamp negative subtree sums to 0?
- [ ] Construct from traversals: do you use a HashMap for inorder index?

### Mixed Practice

1. **Path Sum II:** Backtracking on tree to find all root-to-leaf paths with given sum.
2. **Flatten Binary Tree to Linked List:** Pre-order, right-thread the tree.
3. **Count Complete Tree Nodes:** Binary search on last level.

---

