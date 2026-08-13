# 29. Binary Tree Level Order Traversal

**LeetCode:** [#102 - Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Medium

## Problem statement

Given the root of a binary tree, return the values of its nodes as a list of lists, where each inner list contains the values of one **level**, from top to bottom, left to right within each level.

**Example:**
```
Input:    3
        /   \
       9     20
            /   \
           15    7
Output: [[3],[9,20],[15,7]]
```

## Applicable approaches

- **BFS with level-size tracking** — the standard, expected solution.
- **DFS with a depth parameter** — an alternative approach that also works well.

## Approach 1: BFS with Level-Size Tracking

### Intuition

This problem is asking for exactly what BFS naturally produces — node values grouped by level — so the algorithm here is almost a direct transcription of the topic overview's "why a queue for BFS" explanation into code. Using the `len(queue)` snapshot trick (from Maximum Depth of Binary Tree), we can process one full level at a time and collect each level's values into its own list, rather than just counting levels as that problem did.

### Algorithm

1. If `root` is `None`, return an empty list.
2. Initialize a queue with just `root`.
3. While the queue isn't empty:
   - Record `level_size = len(queue)` (how many nodes are in the current level).
   - Process exactly `level_size` nodes: pop each one, record its value into a `current_level` list, and enqueue its (non-`None`) children.
   - Append `current_level` to the result.
4. Return the result.

### Python code
```python
from collections import deque

def levelOrder(root):
    if not root:
        return []

    result = []
    queue = deque([root])

    while queue:
        level_size = len(queue)
        current_level = []

        for _ in range(level_size):
            node = queue.popleft()
            current_level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)

        result.append(current_level)

    return result
```

### Line-by-line explanation

- `if not root: return []` — edge case: empty tree has no levels, and no nodes to report.
- `queue = deque([root])` — start BFS from the root.
- `while queue:` — keep processing as long as there are more levels to go.
- `level_size = len(queue)` — **critical, exactly as in Maximum Depth's BFS version**: snapshot the current queue size *before* the inner loop, since we'll be appending next-level nodes to the same queue during this loop — this snapshot is what lets us know precisely when the current level ends and stop the inner loop there, rather than accidentally including next-level nodes.
- `current_level = []` — collects this level's values, freshly initialized each outer iteration.
- `for _ in range(level_size):` — process exactly the nodes that were in the queue at the start of this level (not any of the next-level nodes we're about to add during this same pass).
- `node = queue.popleft()` — FIFO removal, which maintains left-to-right order within the level, since children were pushed left-then-right when their parent was processed.
- `current_level.append(node.val)` — record this node's value.
- Enqueue non-`None` children for the *next* level's processing.
- `result.append(current_level)` — once the inner loop finishes (this level fully processed), save the completed level's values.

### Dry run

Tree: `3 -> (9, 20 -> (15, 7))`

- `queue = [3]`.
- Outer iter 1: `level_size = 1`. Process `3`: `current_level=[3]`, enqueue `9`, `20`. `result = [[3]]`. `queue = [9, 20]`.
- Outer iter 2: `level_size = 2`. Process `9`: `current_level=[9]`, no children. Process `20`: `current_level=[9,20]`, enqueue `15`, `7`. `result = [[3],[9,20]]`. `queue = [15, 7]`.
- Outer iter 3: `level_size = 2`. Process `15`: `current_level=[15]`, no children. Process `7`: `current_level=[15,7]`, no children. `result = [[3],[9,20],[15,7]]`. `queue = []`.
- Loop ends (`queue` empty).

Final: `[[3],[9,20],[15,7]]` ✅

### Time & space complexity

- **Time: O(n)** — every node processed exactly once, O(1) amortized work per node.
- **Space: O(n)** — both the queue (up to the widest level, O(n) worst case for a very wide tree) and the result list (holds every node's value once, unavoidable since it's the required output).

---

## Approach 2: DFS with a Depth Parameter

### Intuition

Even though this looks like a "must use BFS" problem, the level-grouped structure can also be produced with DFS, by passing the current depth down through the recursion and using it as an index into the result list — appending a new empty list whenever we reach a depth we haven't seen before. This is worth working through specifically because it demonstrates that "level-order" describes the *output structure*, not a hard requirement on *which traversal algorithm* produces it.

### Python code
```python
def levelOrder(root):
    result = []

    def dfs(node, depth):
        if not node:
            return
        if depth == len(result):
            result.append([])
        result[depth].append(node.val)
        dfs(node.left, depth + 1)
        dfs(node.right, depth + 1)

    dfs(root, 0)
    return result
```

### Line-by-line explanation

- `result = []` — will end up with one inner list per level, built up as new depths are first encountered during the DFS.
- `dfs(node, depth)` — `depth` tracks how many levels deep we currently are (0 = root's level).
- `if depth == len(result): result.append([])` — the first time we reach a given depth, `result` won't have an entry for it yet (its length exactly equals the number of levels created so far) — so we create a new empty list for this level, on demand.
- `result[depth].append(node.val)` — add this node's value to its level's list.
- Recurse into left, then right, each one level deeper (`depth + 1`).

### Why this still produces correctly left-to-right ordered levels despite being depth-first

**This is the subtle part worth understanding, not just accepting:** because DFS always recurses left before right at every node, the *first* time any given depth is reached during the traversal is always via the leftmost path to that depth — and every subsequent node appended to that same `result[depth]` list, across the whole recursion, arrives in an order that respects left-to-right sibling order, because a right subtree at any level is only ever explored after its corresponding left subtree has been fully explored (including all of that left subtree's contribution to deeper levels). So even though the *levels themselves* get filled in an interleaved order across the whole recursion (a deep node in the left subtree gets appended to a high-depth list before a shallow node in the right subtree gets appended to a low-depth list), *within* each individual level's list, the left-to-right order is preserved.

### Time & space complexity

- **Time: O(n)** — every node visited once.
- **Space: O(h)** for the recursion call stack, plus O(n) for the result — so O(n) overall, but the *extra* (non-output) space is O(h), which is better than BFS's O(n)-worst-case queue in a balanced tree (though the same O(n) worst case for a completely skewed tree, where h = n).

---

## Common mistakes & misconceptions

1. **Forgetting the `len(queue)` snapshot in the BFS version**, and instead checking something like `while queue and node.left/right exists` inline — without the snapshot, there's no clean way to know where one level ends and the next begins, since the queue is a continuously flowing structure, not naturally partitioned by level.
2. **Believing DFS "can't" produce level-order output because it's not literally breadth-first.** As shown above, it can — the output structure and the traversal algorithm are two different things, and conflating "level-order" with "must use BFS" is a common but incorrect assumption.
3. **In the DFS version, checking `if depth >= len(result)` instead of `if depth == len(result)`.** In correct usage, `depth` can only ever be `len(result)` (a new level, exactly one deeper than what exists) or less (an already-created level) — it can never jump ahead by more than one, since depth increases by exactly 1 per recursive call. Using `==` is precise and correct; `>=` would also work here but obscures this invariant.
4. **Appending directly to `result` instead of `result[depth]` in the DFS version.** This would flatten every node into one list, losing the level grouping entirely — a subtle bug if you're not careful about which list you're appending to.

## Summary

| Approach | Time | Space (extra) | Notes |
|---|---|---|---|
| BFS with level-size tracking | O(n) | O(n) (queue, worst case) | The natural, standard solution — directly mirrors what the problem asks for. |
| DFS with depth parameter | O(n) | O(h) (call stack) | Works too; a good way to confirm you understand that DFS and BFS aren't rigidly tied to "depth-based" vs. "level-based" problems. |

**Key takeaway:** "process level by level" is BFS's defining use case, and the `level_size = len(queue)` snapshot trick is the standard tool for it — but it's worth knowing DFS can also produce level-grouped output (via a depth parameter indexing into the result), which deepens your understanding of what each traversal strategy is actually doing versus what output shape a problem asks for.
