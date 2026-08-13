# 32. Serialize and Deserialize Binary Tree

**LeetCode:** [#297 - Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Hard

## Problem statement

Design an algorithm to **serialize** a binary tree into a string, and **deserialize** that string back into the exact same tree structure. (There's no single "correct" string format — you design it — it just needs to round-trip correctly.)

## Applicable approaches

- **Preorder DFS with explicit null markers** — the standard, expected approach.
- **BFS (level-order) with explicit null markers** — an equally valid alternative.

## Approach 1: Preorder DFS with Null Markers

### Intuition

The core challenge, worth stating precisely: a plain list of values (like just the values from an inorder traversal alone) **isn't enough** to reconstruct a unique tree shape — you need to also know **where the empty (`None`) children are**, or you can't tell where one subtree ends and another begins (this is the direct tree-shaped analog of Encode and Decode Strings' lesson that a naive scheme loses essential boundary information). The fix: explicitly write down a marker (e.g. `"N"`) every time we encounter a `None` child during traversal, instead of just skipping it. With every `None` explicitly recorded, a **preorder** traversal (root, then left, then right) becomes enough information to reconstruct the tree unambiguously, because we always know exactly when a subtree "closes" — the moment we hit its null markers, which are now genuinely present in the data rather than implied by absence.

This also reuses the **length-prefixing family of ideas** from Encode and Decode Strings: we need an unambiguous way to separate tokens. Here, using a fixed delimiter like `,` between values is safe *specifically because* we're confident node values (typically just integers, per this problem's usual constraints) will never themselves contain that delimiter character — for full robustness against arbitrary string values you could combine this with length-prefixing, but the simple delimiter approach is standard and expected for this specific problem's usual (integer-valued) constraints.

### Algorithm

**Serialize (preorder DFS):**
1. If the current node is `None`, add a null marker (e.g. `"N"`) to the output and return.
2. Otherwise, add the node's value to the output, then recursively serialize the left subtree, then the right subtree.
3. Join everything with a delimiter (e.g. `,`) into one string.

**Deserialize:**
1. Split the string by the delimiter into a list of tokens.
2. Use an index pointer (or an iterator) to consume tokens one at a time, in the same preorder order they were written.
3. Recursively rebuild: read the next token — if it's the null marker, return `None`; otherwise, create a node with that value, then recursively build its left child (consuming the next tokens), then its right child.

### Python code
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val = val
        self.left = left
        self.right = right

class Codec:
    def serialize(self, root):
        result = []

        def dfs(node):
            if not node:
                result.append("N")
                return
            result.append(str(node.val))
            dfs(node.left)
            dfs(node.right)

        dfs(root)
        return ",".join(result)

    def deserialize(self, data):
        values = iter(data.split(","))

        def build():
            val = next(values)
            if val == "N":
                return None
            node = TreeNode(int(val))
            node.left = build()
            node.right = build()
            return node

        return build()
```

### Line-by-line explanation (serialize)

- `result = []` — collects tokens in preorder sequence.
- `dfs(node)`: `if not node: result.append("N"); return` — explicitly record every empty child — this is the key idea that makes reconstruction unambiguous, mirroring the "explicitly mark boundaries" lesson from Encode and Decode Strings.
- `result.append(str(node.val))` — record the node's value (converted to a string, since we'll join everything into one string).
- `dfs(node.left)` then `dfs(node.right)` — preorder order: root already recorded, now left subtree, then right subtree.
- `",".join(result)` — combine all tokens into the final serialized string.

### Line-by-line explanation (deserialize)

- `values = iter(data.split(","))` — split the string back into tokens, wrapped in an **iterator** so that `next(values)` always gives the *next unconsumed* token — this is what lets the recursive `build()` calls naturally "share" progress through the token stream without needing to pass an explicit index variable around by hand (a similar shared-position idea to the running preorder pointer used in Construct Binary Tree from Preorder and Inorder Traversal).
- `build()`: `val = next(values)` — consume the next token, in the same order it was written (preorder: root, then left subtree's tokens, then right subtree's tokens).
- `if val == "N": return None` — this token was a null marker — this subtree position is empty.
- `node = TreeNode(int(val))` — otherwise, it's a real value — create the node.
- `node.left = build()` — recursively consume and build the entire left subtree next (matching exactly how it was written, since we're consuming from the same shared iterator in the same order).
- `node.right = build()` — then the entire right subtree.
- `return node` — return the fully-built node (with its subtrees attached) to whichever call requested it.

### Dry run

Tree:
```
    1
   / \
  2   3
     / \
    4   5
```

**Serialize (preorder, with null markers):**
- `dfs(1)`: append "1". `dfs(2)`: append "2". `dfs(None)` (2's left): append "N". `dfs(None)` (2's right): append "N". Back to `dfs(3)`: append "3". `dfs(4)`: append "4", then "N","N" for its children. `dfs(5)`: append "5", then "N","N".

Result tokens in order: `["1","2","N","N","3","4","N","N","5","N","N"]` → joined: `"1,2,N,N,3,4,N,N,5,N,N"`

**Deserialize** `"1,2,N,N,3,4,N,N,5,N,N"`:
- `build()`: `val="1"` → node(1).
  - `node.left = build()`: `val="2"` → node(2).
    - `node.left = build()`: `val="N"` → `None`.
    - `node.right = build()`: `val="N"` → `None`.
    - returns node(2) with both children `None`.
  - `node.right = build()`: `val="3"` → node(3).
    - `node.left = build()`: `val="4"` → node(4), then consumes "N","N" for its children → node(4) is a leaf.
    - `node.right = build()`: `val="5"` → node(5), then consumes "N","N" → node(5) is a leaf.
    - returns node(3) with `left=4, right=5`.
  - returns node(1) with `left=2, right=3(4,5)`.

Final reconstructed tree exactly matches the original ✅.

### Time & space complexity

- **Time: O(n)** for both serialize and deserialize — each node (and each `None` child, of which there are also O(n)) is visited/processed exactly once.
- **Space: O(n)** for the output string/token list, plus O(h) for the recursion call stack.

---

## Approach 2: BFS (Level-Order) with Null Markers

### Intuition

The same "explicitly record nulls" idea works with BFS instead of DFS — process the tree level by level, and for every node dequeued (including recording "N" for null children), but **not** enqueuing those nulls for further processing, since a `None` has no children of its own to explore.

### Python code
```python
from collections import deque

class Codec:
    def serialize(self, root):
        if not root:
            return "N"

        result = []
        queue = deque([root])
        while queue:
            node = queue.popleft()
            if node:
                result.append(str(node.val))
                queue.append(node.left)
                queue.append(node.right)
            else:
                result.append("N")

        return ",".join(result)

    def deserialize(self, data):
        values = data.split(",")
        if values[0] == "N":
            return None

        root = TreeNode(int(values[0]))
        queue = deque([root])
        i = 1

        while queue:
            node = queue.popleft()
            if values[i] != "N":
                node.left = TreeNode(int(values[i]))
                queue.append(node.left)
            i += 1
            if values[i] != "N":
                node.right = TreeNode(int(values[i]))
                queue.append(node.right)
            i += 1

        return root
```

### Time & space complexity

- **Time: O(n)**, **Space: O(n)** — same overall complexity as the DFS version, just processed level-by-level instead of depth-first.

---

## Common mistakes & misconceptions

1. **Choosing a delimiter or null-marker that could collide with real values.** If node values could be arbitrary strings (not guaranteed integers), using `"N"` as a marker or `","` as a delimiter without escaping could break exactly the way a naive delimiter choice broke Encode and Decode Strings — always check the actual value constraints before assuming a simple marker/delimiter is safe.
2. **Trying to serialize/deserialize using only in-order traversal.** Unlike preorder, in-order traversal alone (even with null markers) doesn't preserve enough information to reconstruct a unique tree shape in all cases without additional structure — preorder (or postorder, with care) is specifically what's needed because it records the root *before* its children, letting deserialization know immediately "here's a root, now consume its subtrees" in a way in-order's "left, root, right" ordering doesn't directly support.
3. **Forgetting to convert values to strings during serialize (or back to integers during deserialize).** `result.append(node.val)` (without `str()`) would work for the append itself but fail at `",".join(result)`, since `join` requires all elements to already be strings — a common, easy-to-miss type error.
4. **In the BFS version, forgetting to still enqueue `None` markers' "positions" implicitly by incrementing the index correctly.** The BFS deserialize code above increments `i` twice per node (once for left, once for right) regardless of whether either child was `None` — getting this bookkeeping wrong (e.g. only incrementing when a real node is created) misaligns the reading position for every subsequent node.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Preorder DFS + null markers | O(n) | O(n) | Simplest to write and reason about; the most commonly expected answer. |
| BFS + null markers | O(n) | O(n) | Equally valid, slightly more bookkeeping with explicit indices. |

**Key takeaway:** the essential trick for serializing *any* tree structure (not just binary trees) is to **explicitly record where the structure ends** (here: null markers for empty children) — without that, a flat list of values alone can't be unambiguously turned back into a specific tree shape. This "explicitly mark the boundaries" idea is the same underlying lesson as Encode and Decode Strings' length-prefixing, applied to tree structure instead of string content — recognizing the two problems share this core idea is more valuable than memorizing either solution in isolation.
