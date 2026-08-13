# 31. Binary Tree Maximum Path Sum

**LeetCode:** [#124 - Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Hard

## Problem statement

A **path** in a binary tree is any sequence of nodes connected by edges, where each node appears at most once, and the path does **not** need to pass through the root. Given the root of a binary tree, return the maximum possible **sum** of values along any path.

**Example:**
```
Input:    -10
         /    \
        9      20
              /   \
             15    7
Output: 42   (path: 15 -> 20 -> 7)
```

## Applicable approaches

- **Brute Force — For every node, try every path through it.** Inefficient, shown for intuition.
- **Optimal — Recursive DFS returning "best downward path from this node," with a separate global-best tracked across all nodes.**

## Approach 1 (Conceptual): Brute Force

### Intuition

For every node as a potential "peak" of a path (the highest point where the path might bend from going up one side to going down the other), you could compute the best path sum achievable using that node as the peak by separately finding the best downward path into its left subtree and the best downward path into its right subtree, adding them together with the node's own value. Doing this **naively** — recomputing "best downward path" from scratch for every single node as a starting point, without reusing work between overlapping subtree computations — leads to significant redundant computation, since a node deep in the tree gets its "best downward path" recomputed once for every one of its ancestors that also needs that information, rather than computed once and reused. This redundancy is exactly what the optimal solution below eliminates by computing each subtree's contribution exactly once, in a single bottom-up pass.

---

## Approach 2: Optimal — DFS Returning Best Downward Path, Tracking a Global Best

### Intuition

This is one of the trickier tree recursion patterns because the function needs to do **two different things at once**, and clearly separating them is the entire key to understanding the solution:

1. **Return**, to its caller (the parent's recursive call), the best sum achievable by a path that starts at *this* node and goes **straight down** into at most one child direction — because if the parent wants to extend a path upward through this node, that path can only continue through *one* of this node's two children (a path can't branch and still be a valid path, by the problem's own definition of "sequence of nodes connected by edges").
2. **Track**, as a side effect, the best possible path sum found *anywhere so far* — which, unlike what's returned, **is allowed to bend** at a node (using both its left and right downward paths simultaneously), because a "bending" path can't be extended any further upward by any ancestor (it already uses both of this node's children), so it only matters as a *final*, complete candidate answer — never as something a parent could build on top of.

We use a variable outside the recursive function (captured via closure with `nonlocal`) to track this second, "can bend" global best, separately from the function's actual return value.

**One more detail, worth deriving rather than memorizing:** if a subtree's best downward path sum is negative, including it in a path can only ever *hurt* the total — a path is always allowed to simply not extend into a low-value subtree and start fresh at the current node instead. So we clamp negative contributions to 0 before using them, which correctly models "choosing not to extend the path in that direction."

### Algorithm

1. Maintain `best = -infinity` (the global best path sum found anywhere, allowed to bend at a node).
2. Define a recursive helper `dfs(node)` that returns "the best sum of a straight downward path starting at `node`":
   - Base case: `None` contributes `0` (an empty path adds nothing).
   - Recursively get `left_gain = max(dfs(node.left), 0)` and `right_gain = max(dfs(node.right), 0)` — clamped to 0, since a negative contribution should just be excluded rather than dragging the sum down.
   - Update the global best: `best = max(best, node.val + left_gain + right_gain)` — this considers the path that **bends** at `node`, using both children's best downward paths simultaneously (this candidate can never be returned upward, since a path can't branch, but it's a valid *complete* path in its own right).
   - Return `node.val + max(left_gain, right_gain)` — what this node can offer *upward* to its own parent: itself, plus the better of its two single-direction downward extensions (not both, since a path continuing upward can only extend in one direction from this node).
3. Run `dfs(root)`, then return `best`.

### Python code
```python
def maxPathSum(root):
    best = float("-inf")

    def dfs(node):
        nonlocal best
        if not node:
            return 0

        left_gain = max(dfs(node.left), 0)
        right_gain = max(dfs(node.right), 0)

        best = max(best, node.val + left_gain + right_gain)

        return node.val + max(left_gain, right_gain)

    dfs(root)
    return best
```

### Line-by-line explanation

- `best = float("-inf")` — starts as negative infinity so that even an all-negative tree (where the "best" path is a single negative node — no positive path exists anywhere) still correctly updates `best` on the first comparison, rather than incorrectly defaulting to 0 (which would be wrong if every possible path sum is actually negative).
- `nonlocal best` — lets the inner `dfs` function modify the outer `best` variable; without this, Python would treat any assignment to `best` inside `dfs` as creating a *new* local variable, shadowing the outer one, and the outer `best` would never actually update.
- `if not node: return 0` — an empty subtree contributes nothing to any path sum, the correct base case for a "sum" computation (contrast with a "count" computation, which might use a different base case).
- `left_gain = max(dfs(node.left), 0)` — recursively get the best downward path from the left child, but never let it be negative — if it would drag the total down, it's better to simply not extend the path into that side at all (equivalent to contributing 0, i.e., "the path just doesn't go there").
- `right_gain = max(dfs(node.right), 0)` — same idea for the right side.
- `best = max(best, node.val + left_gain + right_gain)` — **this is the "bending path" candidate**: a path that comes up from the left subtree, through this node, and back down into the right subtree — a complete, valid path in isolation, but not one that could be extended further by a parent (since it already uses both directions from this node, and a parent could only attach to *one* end of it, which isn't how "extend upward through this node" works once both children are already used).
- `return node.val + max(left_gain, right_gain)` — **this is what gets passed up to the parent**: this node's value plus whichever single side offers more — because if the parent wants to extend a path through this node upward, the path can only continue through *one* of this node's children, not both, since a tree node only connects "up" through a single edge to its own parent, and using both children here would make this node a branch point, not a pass-through point.

### Dry run

```
    -10
   /    \
  9      20
        /  \
       15   7
```

- `dfs(9)`: leaf. `left_gain=max(dfs(None),0)=0`, `right_gain=0`. `best = max(-inf, 9+0+0) = 9`. Return `9 + max(0,0) = 9`.
- `dfs(15)`: leaf. Similarly, `best = max(9, 15) = 15`. Return `15`.
- `dfs(7)`: leaf. `best = max(15, 7) = 15` (7 < 15, no change). Return `7`.
- `dfs(20)`: `left_gain = max(dfs(15), 0) = max(15,0)=15`. `right_gain = max(dfs(7),0)=max(7,0)=7`. `best = max(15, 20+15+7) = max(15, 42) = 42`. Return `20 + max(15,7) = 20+15 = 35`.
- `dfs(-10)`: `left_gain = max(dfs(9),0) = max(9,0)=9`. `right_gain = max(dfs(20),0) = max(35,0)=35`. `best = max(42, -10+9+35) = max(42, 34) = 42` (unchanged — the "bending at root" candidate of 34 isn't better than the already-found 42). Return `-10 + max(9,35) = -10+35 = 25` (this return value is never used by anything, since root has no parent — it's computed but irrelevant to the final answer).

Final answer: `best = 42` ✅ (the path `15 -> 20 -> 7`, discovered as the "bending" candidate at node `20`, correctly excluding the very negative root `-10` entirely — the clamping of negative contributions to 0, combined with tracking `best` separately from the return value, is exactly what allows the true best path to be found even though it doesn't include the root at all).

### Time & space complexity

- **Time: O(n)** — each node visited exactly once, O(1) work per node (a couple of comparisons and additions).
- **Space: O(h)** for the recursion call stack, where h is the tree's height.

---

## Common mistakes & misconceptions

1. **Returning the "bending" value (`node.val + left_gain + right_gain`) instead of the "straight-down" value.** This is the single most common bug: if the function returns the bent-path sum up to its parent, the parent would then (incorrectly) treat that value as if it were a valid single-direction extension, potentially double-counting a branch or constructing an invalid "path" that isn't actually a connected sequence without repeats.
2. **Forgetting to clamp `left_gain`/`right_gain` to 0.** Without `max(dfs(...), 0)`, a negative subtree contribution would be allowed to *decrease* the current node's path sum, when the correct behavior is to simply not extend the path into a negative-sum subtree at all — the fix isn't "compare and pick," it's "clamp before using."
3. **Initializing `best` to 0 instead of negative infinity.** This silently produces a wrong answer specifically on all-negative trees (where every possible path sum is negative) — with `best` starting at 0, an all-negative tree would incorrectly report 0 as the answer, even though 0 was never an achievable path sum (every real path includes at least one node, and every node is negative).
4. **Believing the maximum path must pass through the root**, and only checking `dfs(root)`'s *return value* instead of tracking a separate global best. As the dry run above shows, the true maximum path (through node 20, not through the root at all) is captured only because `best` is checked at *every* node during the traversal, not just derived from the root's own final return value.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Naive brute force (recompute per node) | O(n²) worst case | O(h) to O(n) | Redundant recomputation across overlapping subtrees. |
| DFS with dual return/global-best | O(n) | O(h) | The standard, expected optimal solution. |

**Key takeaway:** this problem is the classic example of a recursive tree function needing to **return one thing (for the parent to use) while tracking a different thing (the actual final answer) as a side effect**, because "the best answer overall" and "the best thing I can offer my parent" are genuinely different questions once branching/bending is allowed. Recognizing when a recursion needs this "return vs. track separately" split is a skill that reappears in several harder tree and graph problems.
