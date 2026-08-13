# 25. Invert Binary Tree

**LeetCode:** [#226 - Invert Binary Tree](https://leetcode.com/problems/invert-binary-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Easy

## Problem statement

Given the root of a binary tree, invert it (swap every node's left and right children, all the way down), and return the root.

**Example:**
```
Input:      4                Output:      4
          /   \                         /   \
         2     7                       7     2
        / \   / \                    / \   / \
       1   3 6   9                  9   6 3   1
```

## Applicable approaches

- **DFS (recursive)** — the most natural, common approach.
- **DFS (iterative, using a stack)** — same idea, no recursion.
- **BFS (iterative, using a queue)** — level-by-level, also works.

**Why this problem is a good one for comparing DFS and BFS side by side:** there's no meaningful "brute force vs. optimal" distinction here — all three traversal strategies achieve the same O(n) time, since every node must be visited exactly once regardless of order, and the work done per node (a swap) is identical regardless of which order you arrive at it in. The interesting choice is *which traversal shape* to use, precisely because this problem doesn't care about visit order at all — a rare case where you can freely pick DFS or BFS purely based on preference or constraints (like avoiding recursion depth risk), not correctness.

## Approach 1: DFS (Recursive)

### Intuition

"Invert this tree" decomposes naturally into: "swap this node's two children, then invert each of those children's subtrees too." Because a subtree is itself a complete, smaller tree (the self-similarity from the topic overview), this is a direct recursive definition — solving the problem for the whole tree is "do a small fixed amount of work at this node, then solve the identical smaller problem on each child."

### Algorithm

1. Base case: if the current node is `None`, there's nothing to invert — return `None`.
2. Swap the node's `.left` and `.right` children.
3. Recursively invert the (now-swapped) left subtree, and recursively invert the right subtree.
4. Return the node.

### Python code
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def invertTree(root):
    if not root:
        return None

    root.left, root.right = root.right, root.left
    invertTree(root.left)
    invertTree(root.right)

    return root
```

### Line-by-line explanation

- `if not root: return None` — base case: an empty subtree is already "inverted" (nothing to do) — without this, the recursion would try to access `.left`/`.right` on `None` and crash.
- `root.left, root.right = root.right, root.left` — Python's tuple-swap syntax; swaps the two child pointers in one line, no temporary variable needed (Python evaluates the right-hand side fully before assigning, so this is safe even though both sides reference the same object).
- `invertTree(root.left)` — recursively invert what is *now* the left subtree (which, after the swap, is the *original* right subtree) — it doesn't matter which physical child we recurse into first, since we're processing both regardless; order between these two lines is irrelevant to correctness.
- `invertTree(root.right)` — same for the other side.
- `return root` — return the (now inverted) node, so the caller (a parent's recursive call, or the top-level caller) gets a usable reference back.

### Dry run

```
    4
  /   \
 2     7
```
Call `invertTree(4)`:
- Swap: `4.left = 7`, `4.right = 2`.
- Recurse `invertTree(7)` (originally the right child, now the left): `7` has no children, swap does nothing, returns `7`.
- Recurse `invertTree(2)` (originally the left child, now the right): same, returns `2`.
- Return `4`, now with `left=7, right=2`.

Result:
```
    4
  /   \
 7     2
```
✅ matches expected pattern (children swapped at every level, since the swap happens at every recursive call, not just the root).

### Time & space complexity

- **Time: O(n)** — every node visited once, O(1) work (one swap) per node.
- **Space: O(h)** where h = tree height, for the recursion call stack — O(log n) for a balanced tree, O(n) worst case for a completely skewed (linked-list-like) tree, since the recursion depth equals the height, and a skewed tree's height equals its node count.

---

## Approach 2: DFS (Iterative, using a Stack)

### Intuition

The exact same "swap, then process both children" logic, but manually managed with an explicit stack instead of relying on the call stack — useful specifically when recursion depth might be a concern (e.g. an extremely unbalanced/deep tree that could exceed Python's default recursion limit, which the iterative version never can, since it doesn't use function-call recursion at all).

### Python code
```python
def invertTree(root):
    if not root:
        return None

    stack = [root]
    while stack:
        node = stack.pop()
        node.left, node.right = node.right, node.left
        if node.left:
            stack.append(node.left)
        if node.right:
            stack.append(node.right)

    return root
```

### Line-by-line explanation

- `stack = [root]` — start with just the root to process.
- `node = stack.pop()` — take the most recently added node (LIFO), matching the topic overview's explanation of why a stack produces depth-first order.
- `node.left, node.right = node.right, node.left` — swap this node's children, same as before.
- `if node.left: stack.append(node.left)` / same for `.right` — push the (already-swapped) children onto the stack for later processing. The `if` checks avoid pushing `None` onto the stack, which would cause an error when later popped and its attributes accessed.

### Time & space complexity

- **Time: O(n)** — same reasoning as the recursive version.
- **Space: O(n)** worst case for the explicit stack — in a very unbalanced tree, the stack can hold a number of nodes proportional to the tree's width or depth depending on shape; treated as O(n) worst case to be safe, though it typically tracks closer to the tree's actual structure.

---

## Approach 3: BFS (Iterative, using a Queue)

### Intuition

Same core operation (swap each node's children), but visiting nodes level by level instead of depth-first — this concretely demonstrates the topic overview's point that when a problem doesn't care about visit *order* (as this one doesn't), BFS and DFS are freely interchangeable, differing only in the underlying data structure (queue vs. stack) and therefore the *order* nodes get processed in, not the *correctness* of the result.

### Python code
```python
from collections import deque

def invertTree(root):
    if not root:
        return None

    queue = deque([root])
    while queue:
        node = queue.popleft()
        node.left, node.right = node.right, node.left
        if node.left:
            queue.append(node.left)
        if node.right:
            queue.append(node.right)

    return root
```

### Line-by-line explanation

- Identical structure to the iterative DFS version, with exactly one difference: `queue.popleft()` instead of `stack.pop()` — taking from the **front** (FIFO) instead of the **end** (LIFO) of the collection. This single difference is what turns a depth-first exploration into a breadth-first (level-by-level) one — the swap logic itself doesn't change at all, which is the concrete proof that order doesn't matter for this specific problem.
- `deque` (double-ended queue, from Python's `collections` module) is used instead of a plain list because `list.pop(0)` (removing from the front) is O(n) — per the Arrays & Hashing topic's explanation of why removing from the front of an array-backed structure requires shifting everything — while `deque.popleft()` is O(1), since a deque is internally structured to support efficient operations at both ends.

### Time & space complexity

- **Time: O(n)**.
- **Space: O(n)** worst case (a wide tree, like a complete binary tree, could have up to roughly n/2 nodes in the queue at the widest level — the queue's peak size is bounded by the tree's maximum width, not its height, which is a genuinely different bound from the stack version's height-bounded worst case, though both are O(n) in the absolute worst case).

---

## Common mistakes & misconceptions

1. **Recursing before swapping, instead of after.** If you write `invertTree(root.left); invertTree(root.right); root.left, root.right = root.right, root.left`, this actually still works correctly (order between "swap" and "recurse into what's about to become the new children" doesn't matter, as long as you're consistent about which pointer you recurse into) — but it's worth being deliberate: recursing into `root.left` and `root.right` *before* swapping means you're recursing into the subtrees using their *original* left/right labels, which are about to be swapped anyway. Either order is correct here, but understanding *why* is worth more than memorizing one specific ordering.
2. **Forgetting the `None` check before pushing onto a stack/queue in the iterative versions.** Without `if node.left:` / `if node.right:`, `None` gets pushed, and the next iteration's `node.left, node.right = ...` crashes trying to access attributes on `None`.
3. **Assuming this problem requires a specific traversal order "because it's a tree problem."** As shown by having three equally-valid solutions here, some tree problems genuinely don't care about order — always ask whether a problem's correctness depends on *when* you process a node relative to its neighbors, or only on *whether* you process it at all.
4. **Creating an entirely new tree instead of modifying in place.** The problem asks to invert and return the (same) tree — building a brand new tree with swapped values would work but wastes O(n) extra memory for no benefit, unlike the in-place swap shown above which costs O(1) extra per node.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DFS (recursive) | O(n) | O(h) call stack | Simplest to write and read. |
| DFS (iterative, stack) | O(n) | O(n) worst case | Avoids recursion depth limits. |
| BFS (iterative, queue) | O(n) | O(n) worst case | Same idea, level-order instead. |

**Key takeaway:** this problem is a great illustration that DFS and BFS are **interchangeable tools for "visit every node and do something"** when the order of visiting doesn't matter for correctness — the real decision between them (in problems where order *does* matter, like most other problems in this topic) comes down to whether you need "go deep first" (stack) or "go wide, level by level" (queue) behavior, a decision this specific problem happens to not require making.
