# 30. Construct Binary Tree from Preorder and Inorder Traversal

**LeetCode:** [#105 - Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Medium

## Problem statement

Given two integer arrays `preorder` and `inorder`, representing the preorder and inorder traversal of the **same** binary tree (with all unique values), reconstruct and return that tree.

**Reminder:** preorder visits **root, then left subtree, then right subtree**. Inorder visits **left subtree, then root, then right subtree**.

**Example:**
```
preorder = [3,9,20,15,7]
inorder  = [9,3,15,20,7]
Output: the tree
     3
   /   \
  9     20
       /  \
      15   7
```

## Applicable approaches

- **Basic Recursive Approach (O(n) search per call → O(n²) overall).**
- **Optimal — Recursive with a Hash Map for O(1) index lookups (→ O(n) overall).**

## Approach 1: Basic Recursive Approach

### Intuition

The key structural fact that makes this problem solvable at all: **the first element of `preorder` is always the root** of the (sub)tree, because preorder visits root first, before either subtree. Once we know the root's value, we can find that same value's position in `inorder` — and because inorder visits left-subtree-then-root-then-right-subtree, **everything to the left of the root in `inorder` belongs to the left subtree**, and **everything to the right belongs to the right subtree**. This single fact tells us exactly how to split both arrays and recurse: we've turned "reconstruct a tree" into "find where the root is in inorder, then reconstruct two smaller trees" — a genuine recursive decomposition, similar in spirit to how a tree's own self-similarity enables recursion generally.

### Algorithm

1. If `preorder` is empty, return `None` (no tree here).
2. The root's value is `preorder[0]`.
3. Find that value's index in `inorder` — call it `mid`. Everything in `inorder` before `mid` is the left subtree's values; everything after is the right subtree's.
4. The left subtree's preorder values are the next `mid` elements of `preorder` (after the root); the right subtree's preorder values are the rest.
5. Recursively build the left subtree and right subtree from their respective preorder/inorder slices, and attach them to a new node holding the root's value.

### Python code
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

def buildTree(preorder, inorder):
    if not preorder:
        return None

    root_val = preorder[0]
    root = TreeNode(root_val)

    mid = inorder.index(root_val)  # O(n) search

    root.left = buildTree(preorder[1:mid + 1], inorder[:mid])
    root.right = buildTree(preorder[mid + 1:], inorder[mid + 1:])

    return root
```

### Line-by-line explanation

- `if not preorder: return None` — base case: no elements means no subtree here.
- `root_val = preorder[0]` — preorder always lists the root first, by definition of the traversal order.
- `mid = inorder.index(root_val)` — find where the root sits in `inorder`; everything before it (`inorder[:mid]`) is the left subtree's inorder sequence, everything after (`inorder[mid+1:]`) is the right subtree's — a direct consequence of what inorder traversal means.
- `preorder[1:mid + 1]` — the left subtree has exactly `mid` nodes (matching the `mid` elements before the root in `inorder`), so its preorder values are the next `mid` elements of `preorder` right after the root — because preorder visits an entire subtree's nodes contiguously before moving to the next subtree.
- `preorder[mid + 1:]` — everything remaining in `preorder` belongs to the right subtree.
- Recursively build both subtrees and attach them.

### Why this is O(n²), precisely

Two separate costs compound here: `inorder.index()` is an O(n) linear search *every single call*, and slicing arrays (`preorder[1:mid+1]`, etc.) itself costs O(k) to create a new array of size k *every single call*. Summed across all n recursive calls (one per node), both costs total O(n²) — this is worth being able to state precisely, not just "it's slow," since the fix (Approach 2) directly targets both of these specific costs.

### Time & space complexity

- **Time: O(n²)** worst case — as explained above.
- **Space: O(n²)** worst case, due to the repeated array slicing creating new arrays at every recursive call, in addition to O(n) or O(h) for the recursion call stack itself.

---

## Approach 2: Optimal — Hash Map for O(1) Index Lookups + Index-Based Recursion (No Slicing)

### Intuition

The basic approach's two inefficiencies have two independent fixes: (1) `inorder.index()` re-scans the array every time — precompute a hash map from value → index in `inorder` once at the start, turning every future lookup into O(1) (the same "trade space for a one-time cost, then get O(1) lookups forever after" idea from the Arrays & Hashing topic). (2) Slicing creates new array copies at every call — pass **index ranges** into the recursive calls instead of actual sliced arrays, so no new arrays are ever created; we just track *which portion* of the original arrays we're currently working within.

### Algorithm

1. Build a hash map `inorder_index`: value → its index in `inorder`, once, up front.
2. Use a helper function that takes index ranges (`in_left, in_right`) instead of actual sliced arrays.
3. Maintain a pointer into `preorder` (since preorder is always consumed strictly left-to-right as we build the tree, we can use a single running index instead of re-slicing — this is the same insight as "preorder visits a subtree's nodes contiguously," now exploited directly rather than just used to justify slicing).
4. At each call: take the current preorder pointer's value as the root, look up its inorder index via the hash map (O(1)), compute how many nodes belong to the left subtree from that index, recurse into the left range first (which correctly advances the shared preorder pointer), then the right range.

### Python code
```python
def buildTree(preorder, inorder):
    inorder_index = {val: i for i, val in enumerate(inorder)}
    self_pre_idx = [0]  # using a list as a mutable "pointer" shared across recursive calls

    def build(in_left, in_right):
        if in_left > in_right:
            return None

        root_val = preorder[self_pre_idx[0]]
        self_pre_idx[0] += 1
        root = TreeNode(root_val)

        mid = inorder_index[root_val]

        root.left = build(in_left, mid - 1)
        root.right = build(mid + 1, in_right)

        return root

    return build(0, len(inorder) - 1)
```

### Line-by-line explanation

- `inorder_index = {val: i for i, val in enumerate(inorder)}` — precompute every value's position in `inorder`, once, for O(1) lookups (this relies on the problem's guarantee that all values are unique — with duplicates, a value → single-index map wouldn't be well-defined).
- `self_pre_idx = [0]` — a single running pointer into `preorder`, shared across all recursive calls (using a one-element list as a simple way to have a "mutable integer" in Python, since plain integers can't be modified in place across nested function calls without the `nonlocal` keyword — this is a common Python idiom worth recognizing).
- `build(in_left, in_right)` — builds the subtree whose *inorder* values span this index range (`[in_left, in_right]`, inclusive); the corresponding preorder values are always found by consuming the shared pointer, since preorder always lists a subtree's root immediately followed by that entire subtree's sequence, contiguously.
- `if in_left > in_right: return None` — empty range, no subtree here (this replaces the old `if not preorder` check, now expressed as an index-range condition instead of an array-emptiness check).
- `root_val = preorder[self_pre_idx[0]]; self_pre_idx[0] += 1` — take the next unused preorder value (this is always the correct root for whatever subtree we're currently building, because of how preorder is structured) and advance the shared pointer, consuming it permanently.
- `mid = inorder_index[root_val]` — O(1) lookup instead of an O(n) scan — this is the direct fix for the first inefficiency identified above.
- `root.left = build(in_left, mid - 1)` — build the left subtree first (matching preorder's left-before-right structure, and correctly advancing the shared pointer through exactly the left subtree's nodes before we ever touch the right subtree) — **the order of these two lines matters**: `root.left` must be built before `root.right`, because the shared pointer's current position depends on exactly how many preorder elements the left subtree consumed.
- `root.right = build(mid + 1, in_right)` — build the right subtree using whatever preorder values remain (the shared pointer is now correctly positioned right after the entire left subtree was consumed).

### Dry run

`preorder = [3,9,20,15,7]`, `inorder = [9,3,15,20,7]`

`inorder_index = {9:0, 3:1, 15:2, 20:3, 7:4}`. `self_pre_idx = [0]`.

- `build(0, 4)`: `root_val = preorder[0] = 3`, pointer→1. `mid = inorder_index[3] = 1`.
  - `root.left = build(0, 0)`: `root_val = preorder[1] = 9`, pointer→2. `mid = inorder_index[9] = 0`. `build(0,-1)`→`None` (empty range, `0>-1`), `build(1,0)`→`None` (empty range, `1>0`). Returns node `9` (leaf).
  - `root.right = build(2, 4)`: `root_val = preorder[2] = 20`, pointer→3. `mid = inorder_index[20] = 3`.
    - `left = build(2, 2)`: `root_val = preorder[3] = 15`, pointer→4. `mid=2`. `build(2,1)`→None, `build(3,2)`→None. Returns leaf `15`.
    - `right = build(4, 4)`: `root_val = preorder[4] = 7`, pointer→5. Returns leaf `7`.
    - Returns node `20` with `left=15, right=7`.
  - Returns node `3` with `left=9, right=20(15,7)`.

Final tree matches the expected structure exactly ✅. Notice `self_pre_idx` was consumed strictly left-to-right, exactly matching preorder's own definition — never needing to search for "which part of `preorder` belongs to this subtree," only ever needing to know *how many* elements to consume next (determined by the index-range width), which is precisely what index-range recursion buys us over slicing.

### Time & space complexity

- **Time: O(n)** — each node is processed exactly once, with O(1) work per node (hash map lookup instead of a scan, index-range tracking instead of slicing) — both of the O(n²)-causing costs from Approach 1 are eliminated.
- **Space: O(n)** — the hash map, plus O(h) for the recursion call stack (O(n) worst case for a skewed tree).

---

## Common mistakes & misconceptions

1. **Building `root.right` before `root.left`.** Because both subtrees share the *same* running preorder pointer, the order of these two lines directly determines correctness — `root.left` must consume its portion of `preorder` first, or `root.right` would start reading from the wrong position (it would incorrectly consume values that actually belong to the left subtree).
2. **Forgetting the problem guarantees unique values, and relying on that guarantee without stating it.** The entire `inorder_index` hash map approach depends on each value mapping to exactly one index — with duplicate values, this breaks down entirely (a value could correspond to multiple positions, making "the" root ambiguous from a value lookup alone), and this problem's constraint of unique values is what makes the whole technique valid.
3. **Using `list.index()` inside the recursive calls out of habit, without noticing it reintroduces the O(n) search per call.** It's easy to "fix" the slicing (switch to index ranges) while forgetting to also fix the lookup — both fixes are needed together to achieve the full O(n) improvement; fixing only one leaves an O(n log n)-to-O(n²) hybrid, not the clean O(n) target.
4. **Off-by-one in the index-range boundaries** (`mid - 1` vs `mid`, `mid + 1` vs `mid`) — always verify against a concrete small example (like the dry run above) rather than trusting the formula from memory, since these are easy to get subtly wrong under time pressure.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Basic recursive (index + slicing) | O(n²) | O(n²) | Correct, but repeated searching and array copying add up. |
| Hash map + index-range recursion | O(n) | O(n) | The standard, expected optimal solution. |

**Key takeaway:** whenever a recursive solution repeatedly searches an array for a value (`list.index()`), consider precomputing a hash map once up front — the classic Arrays & Hashing "trade a one-time O(n) setup cost for O(1) lookups forever after" trade. And when a recursive solution repeatedly slices arrays to "pass smaller pieces down," consider passing index ranges instead — both changes are the same underlying idea: eliminate repeated, avoidable work by either remembering (hashing) or tracking a position (index ranges) instead of re-deriving it every time.
