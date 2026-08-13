# 33. Lowest Common Ancestor of a Binary Search Tree

**LeetCode:** [#235 - Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Medium

## Problem statement

Given a **binary search tree** and two nodes `p` and `q` that exist in it, find their **lowest common ancestor (LCA)** — the deepest node that has both `p` and `q` as descendants (a node can be considered a descendant of itself).

**Example:**
```
        6
      /   \
     2     8
    / \   / \
   0   4 7   9
Input: p = 2, q = 8
Output: 6
```

## Applicable approaches

- **General Tree LCA (works on any binary tree)** — recursive, doesn't use the BST property.
- **Optimal — BST-Specific (using the ordering property)** — much simpler and faster, exploiting that this is specifically a BST.

## Approach 1: General Binary Tree LCA (doesn't use BST property)

### Intuition

Without assuming any ordering, find the LCA by recursively searching both subtrees: if `p` and `q` are found in *different* subtrees of the current node, the current node **is** the LCA — it's the exact point where their paths from the root diverge, since neither `p` nor `q` alone is reachable from the other's subtree here. If both are found in the same subtree, the LCA must be deeper, inside that subtree, since a node closer to the actual LCA is "more specific" and still satisfies "has both as descendants."

### Python code
```python
def lowestCommonAncestor(root, p, q):
    if not root or root == p or root == q:
        return root

    left = lowestCommonAncestor(root.left, p, q)
    right = lowestCommonAncestor(root.right, p, q)

    if left and right:
        return root
    return left if left else right
```

### Time & space complexity

- **Time: O(n)** — in the worst case, visits every node, since without ordering information there's no way to prune the search — we genuinely don't know which subtree `p` or `q` is in until we've searched.
- **Space: O(h)** for the recursion.

*(This works correctly on the BST in this problem too, but it doesn't take advantage of the BST's special ordering property, which lets us do much better — see below.)*

---

## Approach 2: Optimal — BST-Specific (Using the Ordering Property)

### Intuition

In a BST, every node's value tells us which direction a target value must be, **without needing to search both subtrees** — this is the exact same principle the Binary Search topic relies on (a comparison provably eliminates half the remaining space). If both `p.val` and `q.val` are **less than** the current node's value, both must be in the **left** subtree (the BST ordering guarantees this — every value in the right subtree is larger, so neither target could be there) — so the LCA must be there too, and we can ignore the right subtree entirely, with certainty, not just as a heuristic. Symmetrically, if both are **greater**, go right. The moment they're **not both on the same side** (one is ≤ current node's value and the other is ≥, or one of them *is* the current node), we've found the exact point where their paths diverge — that's the LCA, because any node further down would no longer have *both* `p` and `q` as descendants.

### Algorithm

1. Start at `root`.
2. Compare `p.val` and `q.val` to `root.val`:
   - If both are less than `root.val`, move to `root.left` and repeat.
   - If both are greater than `root.val`, move to `root.right` and repeat.
   - Otherwise (they're on different sides, or one of them equals the current node), the current node is the LCA — return it.

### Python code
```python
def lowestCommonAncestor(root, p, q):
    current = root

    while current:
        if p.val < current.val and q.val < current.val:
            current = current.left
        elif p.val > current.val and q.val > current.val:
            current = current.right
        else:
            return current
```

### Line-by-line explanation

- `current = root` — start the search at the top.
- `while current:` — keep walking down the tree (this loop is guaranteed to terminate at the LCA before ever reaching `None`, given the problem's guarantee that both `p` and `q` exist in the tree — the loop will always find a diverging point or a matching node before running out of tree).
- `if p.val < current.val and q.val < current.val:` — both targets are smaller than the current node — the BST property guarantees both must live in the left subtree, so the LCA (if not this node) must be there too.
- `current = current.left` — narrow the search, with certainty, not a guess.
- `elif p.val > current.val and q.val > current.val:` — symmetric case, both targets are larger — go right.
- `else: return current` — this covers every remaining case: `p` and `q` are on opposite sides of `current` (one smaller, one larger), or `current` **is** `p` or `q` itself (with the other one being a descendant somewhere below) — in all of these cases, `current` is exactly the point where the two nodes' paths from the root first diverge (or `current` is one of the two nodes itself, which counts as its own ancestor per the problem statement) — i.e. the LCA.

### Dry run

```
        6
      /   \
     2     8
    / \   / \
   0   4 7   9
```
`p = 2`, `q = 8`

- `current = 6`. `p.val=2 < 6` and `q.val=8 < 6`? No (8 is not < 6). Both less than 6? No. Both greater than 6? `2>6`? No. So neither the "both less" nor "both greater" branch is taken → `else: return current` → return `6` ✅ (2 is in the left subtree, 8 is in the right subtree — they diverge right at the root).

**A second dry run:** `p = 0`, `q = 4` (both in the left subtree)
- `current = 6`. Both `0<6` and `4<6`? Yes → `current = 2`.
- `current = 2`. Both `0<2`? Yes, but `4<2`? No. Both greater than 2? `0>2`? No. Neither condition fully met → return `current = 2` ✅ (0 and 4 diverge at node 2, their direct parent).

### Time & space complexity

- **Time: O(h)** where h = tree height — O(log n) for a balanced BST, O(n) worst case for a completely skewed one. This is a real, provable improvement over the general tree LCA's O(n), because we only ever walk down **one path**, never exploring both subtrees at any point — every step is a certain elimination, not a search.
- **Space: O(1)** for the iterative version (no recursion stack needed at all) — a further improvement over the general approach's O(h) recursive space, since this version doesn't need to remember where it came from to backtrack.

---

## Common mistakes & misconceptions

1. **Using the general tree LCA algorithm on a BST "just to be safe."** This works correctly, but throws away the O(h)/O(1) improvement the BST's ordering property specifically enables — always check whether a tree is specifically a BST before defaulting to the more general (and here, strictly slower) algorithm.
2. **Getting the boundary condition wrong when one of `p`/`q` equals `current`.** The `else` branch correctly handles this (since neither "both less than" nor "both greater than" would be true if one of them equals `current.val`), but it's worth being able to explain *why* — if `p == current`, then `p.val < current.val` is false, so the "both less" branch is correctly skipped, and the function correctly returns `current` immediately, since `current` (being `p` itself) is trivially an ancestor of `p`.
3. **Assuming the recursive version's "compare against value ranges" approach (used for BST validation) is needed here too.** It isn't — this problem doesn't need to track an inherited valid range down the recursion, because at each step we're only asking "which single direction do *both* targets agree on," not "is this specific node's value itself valid" — a simpler, more local question.
4. **Forgetting this specific algorithm relies on `p` and `q` genuinely being present in the tree.** If either doesn't exist, the loop could walk all the way to `None` (if implemented with a `while current` loop that doesn't separately verify presence) or otherwise behave in an unspecified way — the problem's guarantee that both exist is what licenses skipping an existence check.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| General tree LCA (recursive) | O(n) | O(h) | Works on any binary tree, doesn't exploit BST ordering. |
| BST-specific (iterative) | O(h) | O(1) | The standard, expected, much faster solution for a BST specifically. |

**Key takeaway:** always check whether a problem's tree is specifically a **BST** (not just "a binary tree") — the ordering property very often allows a dramatically simpler and faster solution than the general-tree version of the same problem, exactly as it does here (O(h)/O(1) instead of O(n)/O(h)), because each comparison provably eliminates an entire subtree rather than just suggesting where to look.
