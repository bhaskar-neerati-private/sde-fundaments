# 28. Subtree of Another Tree

**LeetCode:** [#572 - Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Easy

## Problem statement

Given the roots of two binary trees `root` and `subRoot`, return `true` if there's a node in `root` such that the subtree rooted at that node is **identical** (same structure and values) to `subRoot`.

**Example:**
```
root:        3            subRoot:   4
           /   \                    / \
          4     5                  1   2
        /   \
       1     2
Output: true
```

## Applicable approaches

- **For every node of `root`, check if the subtree rooted there matches `subRoot`, using Same Tree's comparison.** This is the standard/expected approach for this problem.
- **Alternative — Serialize both trees to strings, check substring containment.**

## Approach 1: For Every Node, Check "Same Tree"

### Intuition

"Is `subRoot` a subtree of `root`?" is really asking: "does there exist **any** node in `root` such that, if you cut the tree there and looked at just that node and everything below it, it would be identical to `subRoot`?" We already know how to check "are these two trees identical" (the previous problem, Same Tree). So this problem reduces to: walk through **every** node of `root`, and at each one, run the Same-Tree check between that node and `subRoot`. The moment any node passes, we're done.

### Algorithm

1. Define a helper `same(node1, node2)` that checks if two trees are identical — exactly the Same Tree solution.
2. Walk through `root` with DFS. At each node visited, check `same(current_node, subRoot)`. If it ever matches, return `True`.
3. If no node in `root` matches after checking all of them, return `False`.

### Python code
```python
def isSubtree(root, subRoot):
    def same(node1, node2):
        if not node1 and not node2:
            return True
        if not node1 or not node2:
            return False
        if node1.val != node2.val:
            return False
        return same(node1.left, node2.left) and same(node1.right, node2.right)

    def dfs(node):
        if not node:
            return False
        if same(node, subRoot):
            return True
        return dfs(node.left) or dfs(node.right)

    return dfs(root)
```

### Line-by-line explanation

- `same(node1, node2)` — identical to the Same Tree solution: `True` if both trees rooted at these two nodes are structurally and value-wise identical.
- `dfs(node)` — walks through **every** node of `root`, treating each one as a candidate "does the subtree starting here match `subRoot`?"
- `if not node: return False` — reached past a leaf (an empty subtree) without a match here — nothing more to check down this path.
- `if same(node, subRoot): return True` — check if the subtree rooted *at this specific node* matches `subRoot` entirely, using the full recursive comparison from the previous problem.
- `return dfs(node.left) or dfs(node.right)` — if this node didn't match, keep searching — check if a match exists somewhere in the left subtree, or (if not) somewhere in the right subtree. `or` short-circuits, so if a match is found in the left subtree, the right subtree isn't even searched — this is a genuine efficiency detail, not just style, since it means the function returns as soon as *any* valid match is found anywhere.
- `return dfs(root)` — kick off the search from the top.

### Dry run

`root: 3 -> (4 -> (1, 2), 5)`, `subRoot: 4 -> (1, 2)`

- `dfs(3)`: `same(3, 4)`? `3.val=3 != 4.val=4` → `False`. Not a match here, recurse: `dfs(4) or dfs(5)`.
  - `dfs(4)`: `same(4, 4)`? values match (4==4), recurse `same(1,1)` (both leaves, match) and `same(2,2)` (both leaves, match) → `same(4,4) = True`. So `dfs(4)` returns `True` immediately (short-circuits before ever reaching `dfs(5)`, since Python's `or` never evaluates its second operand once the first is truthy).
- Overall: `True` ✅

### Time & space complexity

- **Time: O(n · m)** where n = size of `root`, m = size of `subRoot` — in the worst case, we call `same()` (which costs up to O(m)) at up to n different nodes of `root`. This is worth stating precisely: it's *not* O(n + m), because in the worst case (e.g. `root` is a long chain of nodes with the same value as `subRoot`'s root, but never actually matching beyond the first node), we redo a partial O(m)-ish comparison at nearly every one of the n nodes.
- **Space: O(h_root + h_subRoot)** for the combined recursion depth of `dfs` and `same` at any point in time (bounded by the taller of the two trees' heights, roughly, since `dfs`'s own recursion depth and `same`'s recursion depth are both active simultaneously at the deepest point).

*(There exist more advanced O(n + m) solutions using string serialization with proper escaping, or hashing subtree structures — these are worth knowing exist for further study, but the direct "check every node" approach above is the standard, expected solution for this difficulty level and is what you should be able to produce confidently without extra tooling.)*

### A note on the tempting "faster" alternative

A tempting shortcut is to serialize both trees into strings (like in Encode and Decode Strings) and check if `subRoot`'s serialization is a **substring** of `root`'s serialization, which can achieve O(n + m) with an efficient substring search algorithm. This requires careful serialization (e.g. explicitly marking `None` children, and using unambiguous delimiters, exactly per the Encode and Decode Strings problem's lesson) to avoid false positives — e.g. without care, a tree containing the value `12` could false-match a search for a tree containing just `2`, if numbers aren't clearly delimited. This is worth being aware of as a further-optimized idea, but isn't necessary to reach for by default in an interview setting for this specific problem, and it's easy to introduce a subtle correctness bug (exactly the kind flagged in the Encode/Decode problem) if the serialization isn't done carefully.

## Common mistakes & misconceptions

1. **Only checking node values match somewhere, without the full structural comparison.** As with Same Tree, matching values alone (e.g. via a value-only traversal comparison) isn't sufficient — the *entire* subtree structure from the candidate node downward must match exactly, which is why `same()` needs to be the full recursive Same-Tree check, not a shortcut.
2. **Forgetting `dfs` needs to check node itself before recursing into children.** Some incorrect attempts only check children (`dfs(node.left) or dfs(node.right)`) without ever calling `same(node, subRoot)` on `node` itself first — this would miss valid matches at the very node being visited.
3. **Confusing this with Same Tree entirely** — believing you can just call `isSameTree(root, subRoot)` directly. That only checks if the *entire* `root` tree matches `subRoot` exactly; it says nothing about whether `subRoot` matches some smaller piece *within* `root`, which is what this problem actually asks.
4. **Assuming the naive O(n·m) solution is "wrong" or unacceptable because a faster one exists.** For this specific difficulty level and typical constraints, O(n·m) is a fully correct, expected, standard answer — knowing a faster alternative exists is good context, but don't feel obligated to reach for the more error-prone serialization approach unless specifically asked for something faster.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Check every node with Same Tree logic | O(n · m) | O(h_root + h_subRoot) | The standard, expected solution — clear and correct. |
| Serialize + substring search | O(n + m) | O(n + m) | Faster asymptotically, but more subtle to implement correctly (delimiter/escaping care needed). |

**Key takeaway:** many "does X exist somewhere within Y" tree problems reduce to "try a known sub-check (like Same Tree) at every possible position, using DFS to visit every position." Reusing a helper function you already know how to write (Same Tree) inside a broader traversal is a very common problem-solving move — look for it whenever a new problem's statement contains an easier, already-solved problem as a piece of its definition.
