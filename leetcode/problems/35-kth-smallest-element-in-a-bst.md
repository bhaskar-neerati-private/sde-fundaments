# 35. Kth Smallest Element in a BST

**LeetCode:** [#230 - Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Medium

## Problem statement

Given the root of a binary search tree and an integer `k`, return the `k`-th **smallest** value in the tree (1-indexed).

**Example:**
```
      3
     / \
    1   4
     \
      2
Input: k = 1
Output: 1
```

## Applicable approaches

- **Brute Force — Full in-order traversal, collect all values, index into the sorted result.**
- **Optimal — In-order traversal with early stop (as soon as the k-th value is reached).**
- **Optimal (alternative) — Iterative in-order using an explicit stack, stopping early.**

## Approach 1: Brute Force — Collect All, Then Index

### Intuition

Since in-order traversal of a BST produces values in sorted order (established and relied upon in the Validate BST write-up), the most direct approach is to just collect the *entire* in-order sequence into a list, then return the `(k-1)`-th element (0-indexed). This is correct, but it always does the full O(n) traversal regardless of how small `k` is — even for `k=1`, it visits every single node before answering, which is the specific waste the optimal approach targets.

### Python code
```python
def kthSmallest(root, k):
    values = []

    def inorder(node):
        if not node:
            return
        inorder(node.left)
        values.append(node.val)
        inorder(node.right)

    inorder(root)
    return values[k - 1]
```

### Time & space complexity

- **Time: O(n)** — visits every node, even if `k` is small (e.g. `k=1` still visits the whole tree, since the traversal has no way to know it's already found the answer without checking).
- **Space: O(n)** for the values list, plus O(h) for recursion.

*(Correct, but wasteful if `k` is small relative to the tree size — the optimal approach below stops as soon as it has the answer.)*

---

## Approach 2: Optimal — In-order Traversal with Early Stop

### Intuition

We don't need the *entire* sorted sequence — we just need to know when we've visited exactly `k` nodes during an in-order walk, since the k-th node visited during a correct in-order traversal is, by definition, the k-th smallest value. So: do the in-order traversal, but keep a counter, and as soon as the counter reaches `k`, we've found our answer — stop immediately rather than continuing to explore the rest of the tree.

### Algorithm

1. Maintain a counter `count = 0` and a `result` variable.
2. Do an in-order traversal (left, self, right).
3. Every time a node is visited (the "self" step), increment `count`. If `count == k`, record this node's value as the answer, and stop further exploration.

### Python code
```python
def kthSmallest(root, k):
    count = 0
    result = None

    def inorder(node):
        nonlocal count, result
        if not node or result is not None:
            return  # already found the answer, or nothing here - stop exploring this branch

        inorder(node.left)

        if result is not None:  # found during the left subtree - don't process this node or go right
            return

        count += 1
        if count == k:
            result = node.val
            return

        inorder(node.right)

    inorder(root)
    return result
```

### Line-by-line explanation

- `count = 0`, `result = None` — track how many nodes visited so far, and the answer once found.
- `if not node or result is not None: return` — stop exploring if there's nothing here, **or** if we've already found the answer somewhere earlier in the traversal. **This second condition is what makes the early stop actually save work**, rather than just recording the answer but wastefully continuing to traverse anyway — without it, the function would still visit every node, just with a useless extra check at each one.
- `inorder(node.left)` — explore the left subtree first (smaller values, guaranteed by BST ordering to all be less than the current node).
- `if result is not None: return` — if the answer was found while exploring the left subtree, don't process the current node or its right subtree at all — propagate the "stop" signal back up immediately, preventing any further unnecessary work.
- `count += 1` — visiting this node counts as the next-smallest value found so far (this line is only reached if the left subtree didn't already find the answer).
- `if count == k: result = node.val; return` — found the k-th smallest value — record it and stop (don't recurse into the right subtree, since we're done).
- `inorder(node.right)` — only reached if we haven't found the answer yet — continue the in-order walk.

### Dry run

```
      3
     / \
    1   4
     \
      2
```
`k = 1`

In-order visiting order would normally be `1, 2, 3, 4`, but we stop as soon as `count == k`.

- `inorder(3)`: not None, result is None → proceed. `inorder(1)` (left of 3):
  - not None, result None → proceed. `inorder(None)` (left of 1) → returns immediately (no node).
  - back in `inorder(1)`: result still None → `count += 1` → `count = 1`. `count == k(1)`? **Yes** → `result = 1`. Return.
- Back in `inorder(3)`, after `inorder(node.left)` call returns: check `if result is not None: return` → **True** (`result = 1` now) → return immediately, **never processing node 3 itself or exploring node 4 at all**.

Final: `result = 1` ✅, and critically, nodes `3` and `4` (and the subtree under `2`) were never even visited — real, measurable savings when `k` is small relative to the tree size.

### Time & space complexity

- **Time: O(h + k)** — in the worst case (a completely left-skewed tree, or `k` close to `n`), this can be O(n), but in general it's bounded by needing to descend to the leftmost node (O(h)) plus visiting `k` nodes in sorted order — meaningfully better than a guaranteed full O(n) traversal when `k` is small.
- **Space: O(h)** for the recursion call stack.

---

## Approach 3: Optimal (alternative) — Iterative In-order with a Stack

### Intuition

The recursive early-stop version is a bit awkward because of the "check if already found, propagate stop" bookkeeping spread across multiple lines. An **iterative** in-order traversal using an explicit stack naturally supports stopping early with a simple, direct `return` the moment the k-th node is popped — no extra flags needed, because there's no recursive call stack to unwind through, just a `while` loop we can exit immediately.

### Algorithm

1. Use a stack, and a pointer `current` starting at `root`.
2. Standard iterative in-order pattern: push all left-children onto the stack first; when you can't go left anymore, pop, "visit" that node (count it), then move to its right child and repeat.
3. The moment the count reaches `k`, return that node's value immediately.

### Python code
```python
def kthSmallest(root, k):
    stack = []
    current = root
    count = 0

    while stack or current:
        while current:
            stack.append(current)
            current = current.left

        current = stack.pop()
        count += 1
        if count == k:
            return current.val

        current = current.right
```

### Line-by-line explanation

- `stack = []`, `current = root`, `count = 0` — standard iterative in-order traversal setup.
- `while stack or current:` — keep going as long as there's more to explore (either nodes waiting on the stack, or a current node to descend from).
- Inner `while current: stack.append(current); current = current.left` — descend as far left as possible, pushing every node along the way (these are all "not yet visited" ancestors waiting their turn, exactly mirroring the recursive version's implicit call-stack behavior, but made explicit).
- `current = stack.pop()` — once we can't go left anymore, pop the most recently pushed node — this is the next smallest unvisited value (in-order: this is exactly the correct next node to "visit," since everything smaller than it has already been visited via the left-descent).
- `count += 1; if count == k: return current.val` — visiting this node; check if it's the k-th one, and if so, return immediately — no further traversal happens at all, since `return` exits the function directly.
- `current = current.right` — move to this node's right subtree to continue the in-order walk (the outer loop will then push its left-descendants, if any, before visiting it).

### Time & space complexity

- **Time: O(h + k)** — same reasoning as the recursive version, but arguably cleaner to see *why* it's early-stopping, since we `return` directly rather than needing a flag to suppress further recursive work.
- **Space: O(h)** for the explicit stack.

---

## Common mistakes & misconceptions

1. **Forgetting the `result is not None` check at the top of the recursive version.** Without it, the function keeps recursing into every remaining node even after finding the answer, silently degrading back to O(n) — the early stop only actually saves work if every level of the recursion checks and respects the "already found" signal.
2. **Checking `count == k` before incrementing `count`, or incrementing in the wrong place relative to the left/right recursive calls.** The increment must happen exactly at the "visit this node" point — between the left and right recursive calls — matching in-order's defined visit order; incrementing before the left call or after the right call would count nodes in the wrong sequence.
3. **Using pre-order or post-order traversal by mistake.** Since the code structure (recurse left, do work, recurse right) looks almost identical to pre-order/post-order variants (which just move the "do work" line), it's easy to accidentally place the counting logic in the wrong position relative to the two recursive calls — always double-check the "self" step is genuinely *between* the two recursive calls, not before or after both.
4. **Assuming k is always valid without the problem's guarantee.** This solution assumes `1 <= k <= n` (the number of nodes) — if `k` exceeds the tree's size, `result` would remain `None` (recursive version) or the loop would exit without returning (iterative version, implicitly returning `None`) — worth being aware this isn't explicitly handled as an error case, relying on the problem's stated constraints.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Collect all + index | O(n) | O(n) | Simple, but always does full O(n) work regardless of k. |
| Recursive in-order, early stop | O(h + k) | O(h) | Faster when k is small; slightly awkward "stop propagation" bookkeeping. |
| Iterative in-order (stack), early stop | O(h + k) | O(h) | Same speed, cleaner early-return logic since there's no recursive unwind to manage. |

**Key takeaway:** "in-order traversal of a BST gives sorted order" is one of the single most useful facts in this whole topic — any "k-th smallest/largest," "closest value," or "sorted output" question about a BST should immediately bring this to mind, and stopping the traversal early (rather than always computing everything) is a worthwhile, genuine optimization once you know exactly what you're looking for and can prove it's safe to stop.
