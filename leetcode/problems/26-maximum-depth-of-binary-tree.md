# 26. Maximum Depth of Binary Tree

**LeetCode:** [#104 - Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Easy

## Problem statement

Given the root of a binary tree, return its **maximum depth** (the number of nodes along the longest path from the root down to the farthest leaf).

**Example:**
```
Input:    3
        /   \
       9     20
            /   \
           15    7
Output: 3
```

## Applicable approaches

- **Recursive DFS** — the most natural approach.
- **Iterative DFS (stack, tracking depth per node)**.
- **BFS (queue, counting levels)**.

## Approach 1: Recursive DFS

### Intuition

The depth of a tree rooted at `node` has a direct recursive definition: it's `1` (for the node itself) plus whichever of its two children's subtrees is deeper — because the longest path through this node must go through whichever child leads to the deeper subtree. This is the topic overview's "recursive DFS returning a value" pattern in its purest form: a node's answer is computed entirely from its children's answers, with no other information needed.

### Algorithm

1. Base case: an empty tree (`None`) has depth 0.
2. Recursively compute the depth of the left subtree and the right subtree.
3. Return `1 + max(left_depth, right_depth)` (the `+1` accounts for the current node itself).

### Python code
```python
def maxDepth(root):
    if not root:
        return 0
    left_depth = maxDepth(root.left)
    right_depth = maxDepth(root.right)
    return 1 + max(left_depth, right_depth)
```

### Line-by-line explanation

- `if not root: return 0` — base case: no node here means no depth to add — this also correctly handles a completely empty tree (`root` itself is `None`), returning depth 0.
- `left_depth = maxDepth(root.left)` — recursively find how deep the left side goes; this call itself bottoms out at `None` children eventually, unwinding back up with real values.
- `right_depth = maxDepth(root.right)` — same for the right side.
- `return 1 + max(left_depth, right_depth)` — the current node adds 1 to whichever side is deeper (the longest path through this node necessarily goes through its deeper child, since going through the shallower child could never produce a longer path).

### Dry run

```
    3
   / \
  9   20
     /  \
    15   7
```
- `maxDepth(9)`: no children → `1 + max(0,0) = 1`.
- `maxDepth(15)`: no children → 1. `maxDepth(7)`: no children → 1.
- `maxDepth(20)`: `1 + max(1, 1) = 2`.
- `maxDepth(3)`: `1 + max(1, 2) = 3`.

Final: `3` ✅

### Time & space complexity

- **Time: O(n)** — every node visited exactly once, O(1) work per node (one comparison, one addition).
- **Space: O(h)** — recursion call stack, h = tree height; O(log n) for a balanced tree, O(n) for a completely skewed one.

---

## Approach 2: Iterative DFS (Stack, Tracking Depth)

### Intuition

Use an explicit stack (per the Stack topic's "simulate recursion" pattern), but store **each node's depth alongside it**, so we always know how deep we are without needing the call stack to implicitly track it via recursion frames.

### Python code
```python
def maxDepth(root):
    if not root:
        return 0

    stack = [(root, 1)]
    best = 0

    while stack:
        node, depth = stack.pop()
        best = max(best, depth)
        if node.left:
            stack.append((node.left, depth + 1))
        if node.right:
            stack.append((node.right, depth + 1))

    return best
```

### Line-by-line explanation

- `stack = [(root, 1)]` — each stack entry pairs a node with its depth (root starts at depth 1, matching the "1 + ..." convention from the recursive version).
- `best = max(best, depth)` — track the deepest depth seen among all visited nodes; since every leaf eventually gets popped and its depth checked, the deepest leaf's depth is guaranteed to be captured.
- Pushing children with `depth + 1` — each child is one level deeper than its parent, carried explicitly instead of implicitly via recursion depth.

### Time & space complexity

- **Time: O(n)**, **Space: O(n)** worst case for the stack (in the worst case, a skewed tree puts every node on the stack one at a time as it's discovered, though never more than the tree's height at once for a stack-based DFS — still bounded by O(n) to be safe).

---

## Approach 3: BFS (Queue, Counting Levels)

### Intuition

Process the tree level by level; the total number of levels processed by the time the queue empties is, by definition, the maximum depth — this reframes "find the deepest node" as "count how many levels exist," which BFS answers naturally since it processes one complete level per outer iteration.

### Python code
```python
from collections import deque

def maxDepth(root):
    if not root:
        return 0

    queue = deque([root])
    depth = 0

    while queue:
        depth += 1
        for _ in range(len(queue)):  # process exactly this level's nodes
            node = queue.popleft()
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

    return depth
```

### Line-by-line explanation

- `for _ in range(len(queue)):` — **the key trick, worth understanding precisely**: `len(queue)` is captured *once*, before the inner loop starts adding next-level nodes, so this inner loop processes **exactly** the nodes that were already in the queue at the start of this level — even though the queue itself keeps growing (as next-level nodes get appended) during the very loop that's iterating over the old count.
- `depth += 1` — once per level processed, since we're counting levels, not nodes.
- After the inner loop finishes, the queue contains exactly the next level's nodes (everything added during this pass), ready for the next outer iteration to process as its own fixed-size batch.

### Dry run (BFS)

Same tree as before. `queue=[3]`, depth=0.
- Outer iter 1: `depth=1`. Process the 1 node in queue (`3`): push `9`, `20`. Queue now `[9,20]`.
- Outer iter 2: `depth=2`. Process 2 nodes (`9`, `20`): `9` has no children; `20` pushes `15`,`7`. Queue now `[15,7]`.
- Outer iter 3: `depth=3`. Process 2 nodes (`15`,`7`): neither has children. Queue now `[]`.
- Loop ends (`queue` empty). Return `depth = 3` ✅

### Time & space complexity

- **Time: O(n)**, **Space: O(n)** worst case (the widest level of the queue — bounded by the tree's maximum width, a different bound than the DFS stack's height-bound, though both are O(n) in the absolute worst case).

---

## Common mistakes & misconceptions

1. **Forgetting the `+1` in the recursive version, or misplacing it.** `return max(left_depth, right_depth)` (without the `+1`) would compute the depth of the *deeper child*, not the depth including the current node — off by exactly one at every level, compounding into a significantly wrong answer for deep trees.
2. **Using `len(queue)` incorrectly in the BFS version — e.g. calling it inside the inner loop instead of capturing it once before the loop starts.** If you wrote `for _ in range(len(queue))` in a way that re-evaluates `len(queue)` on every inner iteration (which the `range(len(queue))` form above does *not* do — `range()`'s argument is evaluated once, up front), the loop would keep including newly-appended next-level nodes in the *current* level's count, incorrectly merging two levels together.
3. **Confusing "depth" and "height" conventions.** Some definitions have an empty tree at height `-1` and a single node at height `0`; this problem's convention (matching the code above) has an empty tree at depth `0` and a single node at depth `1`. Mixing conventions between what you remember from a class and what a specific problem asks for is a common source of off-by-one answers.
4. **Assuming DFS is strictly "better" than BFS here, or vice versa.** Both are O(n) time and O(n) space in the worst case — the choice is stylistic for this specific problem, though the BFS version's `len(queue)` snapshot trick is worth learning regardless, since it generalizes to many other level-based problems where DFS has no natural equivalent.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursive DFS | O(n) | O(h) | Shortest, most natural solution. |
| Iterative DFS (stack + depth) | O(n) | O(n) | Avoids recursion, tracks depth explicitly. |
| BFS (level counting) | O(n) | O(n) | Naturally counts levels; the "len(queue) snapshot" trick is broadly useful for any level-order problem. |

**Key takeaway:** the `for _ in range(len(queue))` trick for processing "exactly one level at a time" in BFS is one of the most reusable patterns in tree/graph problems — remember it, since it comes up constantly whenever a problem cares about level boundaries specifically, not just visiting order in general.
