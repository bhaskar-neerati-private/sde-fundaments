# Topic 7: Trees

## Core concepts / data structures

### Tree (and Binary Tree specifically)

**What it is:** a tree is a data structure made of nodes, where each node can point to some number of "child" nodes, starting from one special node called the **root**, with no cycles (you can never walk from a node back to an ancestor of itself). A **binary tree** is a tree where every node has **at most two** children, conventionally called `left` and `right`.

**Simple explanation:** think of a family tree, but flipped: one ancestor (the root) at the top, with children branching downward. Unlike a linked list (one path forward at every step), a tree can branch in multiple directions at every node — this branching is the single conceptual thing that's genuinely new compared to the previous topic.

**Node structure in Python:**
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right
```

**Key vocabulary:** the **root** is the top node; a node with no children is a **leaf**; a node's **depth** is how many steps from the root to reach it; a tree's **height** is the depth of its deepest leaf; a **subtree** rooted at any node is that node plus everything beneath it (itself a complete, valid tree — this self-similarity is exactly what makes recursion the natural tool here, see below).

### Binary Search Tree (BST) — a special kind of binary tree

**What it is:** a binary tree with an extra rule: for **every** node, everything in its **left** subtree is smaller than the node's value, and everything in its **right** subtree is larger. This ordering property is what makes searching a BST fast, and it's a direct extension of the Binary Search topic's monotonicity requirement — a BST is essentially "a sorted array's ordering property, expressed as a tree shape instead of a flat sequence."

**Why it matters:** because of this ordering, you can search for a value the same way you'd binary search a sorted array — at each node, compare your target to the current value and go left or right accordingly, discarding the entire opposite subtree at each step (in a **balanced** BST — see below), for exactly the same "provably safe to eliminate half" reason binary search works on arrays.

### The two fundamental traversal strategies

**1. Depth-First Search (DFS)** — go as deep as possible down one branch before backtracking to explore others. Implemented either **recursively** (the natural, common way — because a tree's recursive self-similarity maps directly onto function calls) or **iteratively with an explicit stack** (see the Stack topic — this is the "explicit stack instead of recursion" idea mentioned there, made concrete).
```python
def dfs(node):
    if not node:
        return
    # "pre-order": process node here, before visiting children
    dfs(node.left)
    # "in-order": process node here, between children
    dfs(node.right)
    # "post-order": process node here, after visiting children
```
- **Pre-order** (process, then left, then right) — useful for copying/serializing a tree (parent info needed before children can be reconstructed).
- **In-order** (left, then process, then right) — for a **BST**, this visits nodes in **sorted ascending order**, a direct consequence of the BST ordering rule — extremely useful, and worth internalizing as a fact you can rely on, not re-derive each time.
- **Post-order** (left, then right, then process) — useful when a node's answer depends on its children's answers *first* (e.g. computing height, deleting a tree bottom-up) — this is the traversal order that matches "compute sub-answers before combining them."

**2. Breadth-First Search (BFS)** — visit all nodes at depth 0, then all nodes at depth 1, then depth 2, and so on — level by level. Implemented with a **queue** (First In, First Out), not a stack.
```python
from collections import deque

def bfs(root):
    if not root:
        return
    queue = deque([root])
    while queue:
        node = queue.popleft()
        # process node here
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)
```

**Why a queue for BFS but a stack for DFS — the actual reason, not just the rule:** DFS wants to fully explore whatever it just discovered before moving to siblings — LIFO (the most recently discovered node is explored next) naturally produces this "go deep first" behavior, since the most recently pushed child is the very next thing popped. BFS wants to explore things in the order they were discovered, level by level — FIFO (the earliest discovered, not-yet-explored node goes next) naturally produces this "finish this level before starting the next" behavior, since older discoveries are always processed before newer ones. The data structure's ordering guarantee directly determines the traversal's shape — this is the same principle from the Stack topic overview, applied to trees.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Recursive DFS returning a value** | Most "compute something about the tree" problems (height, sum, is-balanced) — a node's answer is computed from its children's answers, recursively, exploiting the tree's self-similar structure. |
| **BFS / level-order traversal** | "Process level by level," "find the shortest path in an unweighted tree," "find the rightmost/leftmost node per level." |
| **In-order traversal for BSTs** | Anything involving "sorted order" in a BST — in-order traversal gives you sorted values for free, as a direct consequence of the BST invariant. |
| **Two recursive calls compared/combined** | "Are these two trees the same?", "is this tree symmetric?" — recurse on corresponding pairs of nodes from two (sub)trees simultaneously. |
| **Passing extra state down the recursion** (e.g. valid value range for BST validation) | When a node's correctness depends on constraints from its ancestors, not just its own children — a single node's local neighbors aren't enough information. |
| **"Global" answer tracked outside the recursion** | Problems like "maximum path sum" where the best answer might not pass through the root, requiring you to track a running best across the whole recursive exploration while still returning a *different* value up each recursive call for the parent to use. |

## Key terminology

- **Root** — the top-most node of the tree.
- **Leaf** — a node with no children.
- **Depth** (of a node) — number of edges from the root to that node.
- **Height** (of a tree, or a subtree) — the depth of its deepest leaf; a single node has height 0 (or sometimes defined as 1, depending on convention — be consistent within a problem, and state your convention if it matters).
- **Balanced tree** — a tree where, for every node, the heights of its left and right subtrees differ by at most 1 — this is what guarantees O(log n) height, and therefore O(log n) search time for a balanced BST (an *unbalanced* BST can degrade to a linked-list shape, with O(n) search — the ordering property alone doesn't guarantee speed, balance does too).
- **DFS (pre/in/post-order)** vs. **BFS (level-order)** — see above.
- **LCA (Lowest Common Ancestor)** — the deepest node that has both of two given nodes as descendants (or is one of them itself).

## Common beginner mistakes

1. **Forgetting the base case in recursion** (`if not node: return ...`). Every recursive tree function needs to handle the "empty subtree" case — a node with a missing child recurses into `None`, and without a base case, this crashes with an `AttributeError` (trying to access `.val` or `.left` on `None`). Return something sensible: `0`, `True`, `None`, or an empty list, depending on what's being computed.
2. **Confusing DFS traversal orders.** Mixing up pre/in/post-order, especially when the problem specifically needs one (e.g. in-order for sorted BST output) — since all three "look similar" in code (they differ only in *when* the "process node" line runs relative to the two recursive calls), it's easy to write the wrong one without noticing, since the code still runs without crashing, it just visits nodes in the wrong order.
3. **Validating a BST by only checking immediate children**, instead of the full valid range inherited from all ancestors. A common bug: checking `node.left.val < node.val < node.right.val` locally at every node, which misses cases where a deeper-left-then-right node violates an *ancestor's* constraint further up the tree, not just its immediate parent's.
4. **Using DFS (a stack) when BFS (a queue) is actually needed**, or vice versa. If a problem is about "levels" or "shortest path," that's a BFS signal (per the "why a queue" explanation above); if it's about "explore fully down one path first" or computing bottom-up subtree properties, that's DFS.
5. **Not handling `None` children correctly in BFS.** Forgetting to check `if node.left:` / `if node.right:` before adding to the queue causes `None` to be enqueued, which then crashes (or needs extra handling) when later popped and its `.val`/`.left`/`.right` accessed.
6. **Off-by-one in height/depth calculations**, especially around whether an empty tree has height `0` or `-1`, and whether a single node has height `0` or `1` — pick a convention and apply it consistently throughout one solution; mixing conventions mid-solution is a common, hard-to-spot bug source.

## Starter problems (easy, to warm up)

1. **Invert Binary Tree** (LeetCode #226) — a clean, simple recursive DFS warm-up. Also in your Blind 75 list.
2. **Maximum Depth of Binary Tree** (LeetCode #104) — also in your Blind 75 list; the classic "compute a value from children" DFS pattern.
3. **Same Tree** (LeetCode #100) — also in your Blind 75 list; a great "two trees compared simultaneously" warm-up.

## How this compares to Linked Lists

A tree node is essentially a linked list node with **two** `.next`-style pointers (`.left` and `.right`) instead of one — so all the "careful pointer manipulation, save references before overwriting" discipline from Linked Lists carries over directly and matters just as much here. What's genuinely new: **branching**. A linked list problem only ever has one path forward, so iteration is enough; a tree has multiple paths at every node, which is exactly why **recursion** becomes the natural default tool here — each recursive call naturally handles "explore this branch fully, then come back and explore the other," which is precisely what a linked list never needed to do.

## What carries over from here

DFS/BFS, learned here on trees, are the exact same techniques (with almost identical code) used on **Graphs** in the next-but-one topic — a tree is technically just a special kind of graph (one with no cycles and exactly one path between any two nodes), so everything about traversal order and the stack-vs-queue distinction transfers directly, with one addition (graphs need explicit cycle protection, since they don't structurally guarantee acyclic-ness the way a tree does). The recursive "compute an answer from children" pattern also directly prepares you for **Dynamic Programming** later, where "an answer built from smaller sub-answers" is the core idea, just usually expressed with memoized function calls or a table instead of tree nodes.
