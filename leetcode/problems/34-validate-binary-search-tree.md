# 34. Validate Binary Search Tree

**LeetCode:** [#98 - Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Medium

## Problem statement

Given the root of a binary tree, determine if it is a valid **binary search tree (BST)**: for every node, **all** values in its left subtree must be strictly less than the node's value, and **all** values in its right subtree must be strictly greater — not just its immediate children.

**Example (invalid, a common trap):**
```
     5
   /   \
  1     4
       / \
      3   6
Output: false
```
This is invalid even though `5 > 1` and `4`'s immediate children (`3` and `6`) look locally fine relative to `4` — the problem is that `3` is in `5`'s right subtree, but `3 < 5`, violating the *ancestor* constraint from node `5`, not just node `4`.

## Applicable approaches

- **Wrong (common bug) — Check only immediate parent/child relationship.** Shown to explain why it fails.
- **Optimal — Recursive with a Valid Range (min/max bounds) passed down.**
- **Optimal (alternative) — In-order Traversal, check strictly increasing.**

## Why the "just check immediate children" approach is wrong

A tempting-but-incorrect approach: recursively check `left.val < node.val < right.val` using only each node's *direct* children. This fails on exactly the example above — node `4` locally looks fine (`3 < 4 < 6`), but `3` is invalid because it violates the *root's* constraint (`3` must be `< 5` since it's `5`'s node value... wait, more precisely: `3` is in `5`'s right subtree, so it must be `> 5`, and it isn't). **Every node's valid range is constrained by ALL of its ancestors, not just its immediate parent** — this is the core insight the correct approaches below are built around, and it's worth being able to state exactly why the naive check misses it: a purely local check only ever compares a node to its *direct* parent/children, never to a *grandparent* or higher, so any violation that "skips a generation" (like this one) slips through undetected.

## Approach 1: Optimal — Recursive with a Valid Range

### Intuition

As we recurse down the tree, we can track the **valid range** `(low, high)` that the current node's value must fall within, based on all the decisions made by its ancestors so far — this directly fixes the "only checks immediate neighbors" flaw by explicitly carrying ancestor information forward through the recursion, instead of discarding it after each level. Every time we go **left**, the upper bound tightens to the parent's value (everything in a left subtree must be less than its parent). Every time we go **right**, the lower bound tightens to the parent's value. This correctly accumulates constraints from *every* ancestor along the path, not just the immediate parent, because each recursive call inherits the range from its own caller, which in turn inherited it from *its* caller, and so on back to the root.

### Algorithm

1. Define a helper `valid(node, low, high)` — `True` if the subtree rooted at `node` is a valid BST **and** every value in it falls strictly within `(low, high)`.
2. Base case: `None` is trivially valid (empty subtree, no violation possible).
3. If `node.val` isn't strictly between `low` and `high`, it violates some ancestor's constraint — return `False`.
4. Otherwise, recursively validate: `valid(node.left, low, node.val)` (tightening the upper bound to this node's value) **and** `valid(node.right, node.val, high)` (tightening the lower bound).

### Python code
```python
def isValidBST(root):
    def valid(node, low, high):
        if not node:
            return True
        if not (low < node.val < high):
            return False
        return valid(node.left, low, node.val) and valid(node.right, node.val, high)

    return valid(root, float("-inf"), float("inf"))
```

### Line-by-line explanation

- `valid(node, low, high)` — `node`'s value (and everything in its subtree) must stay strictly within `(low, high)`, where `low`/`high` encode *every* ancestor's constraint accumulated so far, not just the direct parent's.
- `if not node: return True` — an empty subtree can never violate anything.
- `if not (low < node.val < high): return False` — Python allows this chained comparison syntax; checks that `node.val` is strictly greater than `low` **and** strictly less than `high` in one expression. If it fails, some ancestor's constraint has been violated — this is exactly the check the naive "only immediate children" approach was missing, because `low`/`high` here can come from *any* ancestor, arbitrarily far up the tree.
- `valid(node.left, low, node.val)` — the left subtree keeps the same lower bound (`low`, inherited from ancestors), but its upper bound tightens to `node.val` (everything in the left subtree must be less than `node`).
- `valid(node.right, node.val, high)` — mirror: the right subtree's lower bound tightens to `node.val`, upper bound stays inherited.
- `return valid(root, float("-inf"), float("inf"))` — the root has no constraints from any ancestor, so it starts with an unbounded valid range.

### Dry run

```
     5
   /   \
  1     4
       / \
      3   6
```
`valid(5, -inf, inf)`: `-inf < 5 < inf` ✓. Recurse:
- `valid(1, -inf, 5)`: `-inf < 1 < 5` ✓. `1` is a leaf → `valid(None,...) and valid(None,...)` → `True`.
- `valid(4, 5, inf)`: **check** `5 < 4 < inf`? `5 < 4` is **False** → return `False` immediately.

Since the right-side call to `valid(4, 5, inf)` returns `False`, the `and` short-circuits and the whole thing returns `False` ✅ (correctly detects the violation — node `4` and everything under it must be `> 5`, since it's in the root's right subtree, but `4` itself already fails that).

*(Notice how the bound `5` — the root's value — correctly propagated all the way down to constrain node `4`'s subtree, which is exactly what the "only check immediate children" broken approach fails to do; that broken version would have separately checked `3 < 4 < 6` locally and found nothing wrong, missing the violation entirely.)*

### Time & space complexity

- **Time: O(n)** — every node visited once, O(1) work per node.
- **Space: O(h)** for the recursion call stack.

---

## Approach 2: Optimal (alternative) — In-order Traversal, Check Strictly Increasing

### Intuition

Recall from the Trees topic overview: an **in-order traversal of a valid BST always produces values in strictly increasing sorted order** — this is a direct consequence of the BST ordering rule, not a coincidence. So instead of tracking ranges explicitly, we can simply perform an in-order traversal and check that each value is strictly greater than the previous one visited — if we ever find a value that's *not* greater than the one before it, the tree isn't a valid BST. This approach sidesteps the "range" bookkeeping entirely by leaning on a single global fact about what correct in-order output must look like.

### Python code
```python
def isValidBST(root):
    prev = [float("-inf")]  # mutable "pointer" to track the previously visited value

    def inorder(node):
        if not node:
            return True
        if not inorder(node.left):
            return False
        if node.val <= prev[0]:
            return False
        prev[0] = node.val
        return inorder(node.right)

    return inorder(root)
```

### Line-by-line explanation

- `prev = [float("-inf")]` — tracks the most recently visited value during the in-order walk (using a one-element list as a simple mutable container, similar to the shared-pointer trick from Construct Binary Tree from Preorder and Inorder Traversal — could also use `nonlocal` with a plain variable instead).
- `inorder(node)`: `if not inorder(node.left): return False` — fully process the left subtree first (in-order: left, then self, then right); if it's already found to be invalid, propagate that failure immediately without bothering to check the current node or the right subtree.
- `if node.val <= prev[0]: return False` — the crucial check: the current node's value must be strictly greater than the last value we visited (which, since we're going in-order, was the largest value visited so far in a correctly-ordered scan) — if it isn't, the ordering property is broken somewhere.
- `prev[0] = node.val` — update the "last visited" tracker.
- `return inorder(node.right)` — continue the in-order walk into the right subtree.

### Dry run

Same tree as before. In-order visits: `1, 5, 3, 4, 6` (left of 5 first: just `1`; then `5` itself; then right subtree of 5 in-order: left of 4 is `3`, then `4`, then right of 4 is `6`).

- Visit `1`: `1 <= -inf`? No. `prev = 1`.
- Visit `5`: `5 <= 1`? No. `prev = 5`.
- Visit `3`: `3 <= 5`? **Yes** → return `False` immediately.

Correctly detects the violation: `3` appearing after `5` in in-order sequence, without being greater than it, proves the BST ordering is broken. ✅

### Time & space complexity

- **Time: O(n)**, **Space: O(h)** for the recursion (plus O(1) extra for the `prev` tracker).

---

## Common mistakes & misconceptions

1. **Checking only immediate children (`node.left.val < node.val < node.right.val`).** As demonstrated above, this doesn't account for ancestor constraints beyond the immediate parent — a genuinely incorrect approach that passes many simple test cases while failing on trees where a violation "skips a generation."
2. **Using `<=`/`>=` instead of strict `<`/`>` in the range check.** The problem requires *strictly* less/greater — a BST with duplicate values arranged ambiguously (e.g. `node.val == low` or `node.val == high`) should be invalid, and using non-strict comparisons would incorrectly accept such cases.
3. **Forgetting to initialize `prev` to negative infinity in the in-order approach.** Without a suitably small starting sentinel, the very first visited value could incorrectly fail the `<= prev[0]` check (or, if initialized to something like `0`, incorrectly reject a valid tree containing negative values) — negative infinity guarantees the first comparison always passes, exactly mirroring the range approach's `float("-inf")` root bound.
4. **Forgetting the left-subtree short-circuit in the in-order version.** `if not inorder(node.left): return False` must run *before* checking the current node — if you check the current node first and only later look at `inorder(node.left)`'s result, you could report a value comparison as valid before knowing the left subtree was already broken, though in practice this specific ordering mistake is more about wasted work than incorrectness, since the final `return inorder(node.right)` would still eventually propagate a `False` — but structuring it correctly (left-check first) keeps the short-circuit behavior clean and avoids doing unnecessary further traversal after a failure is already known.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Check only immediate children | - | - | **Incorrect** — doesn't account for ancestor constraints. |
| Recursive with valid range (min/max) | O(n) | O(h) | The standard, most commonly taught correct solution. |
| In-order traversal, check increasing | O(n) | O(h) | Equally valid, leverages the BST↔sorted-inorder relationship directly. |

**Key takeaway:** "is this a valid BST" is a classic trap for the "only check immediate parent/child" bug — always remember that BST validity is a property that must hold against **every ancestor**, not just the direct parent, which is why passing a shrinking valid range down the recursion (or equivalently, checking in-order output is strictly increasing) is necessary, not merely a stylistic choice.
