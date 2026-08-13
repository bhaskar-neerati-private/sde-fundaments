# 27. Same Tree

**LeetCode:** [#100 - Same Tree](https://leetcode.com/problems/same-tree/) · **Topic:** [Trees](../topics/07-trees.md) · **Difficulty:** Easy

## Problem statement

Given the roots of two binary trees `p` and `q`, return `true` if they are **structurally identical** and every corresponding node has the **same value**.

**Example:**
```
p:   1        q:   1
   /   \          /   \
  2     3        2     3
Output: true
```

## Applicable approaches

- **Recursive DFS (compare corresponding nodes simultaneously)** — the natural, expected approach.
- **Iterative DFS/BFS (compare using a stack/queue of node pairs)**.

## Approach 1: Recursive DFS

### Intuition

Two trees are the same exactly when: both current nodes are `None` (both empty here — a match, since "nothing" equals "nothing"), or both are non-`None` **and** have equal values **and** their left subtrees are the same **and** their right subtrees are the same. This is a direct recursive definition comparing both trees "in lockstep," and it's a slightly different flavor of recursion from Maximum Depth — instead of one tree feeding one recursive call, we're walking *two* trees simultaneously, one pair of corresponding nodes at a time.

### Algorithm

1. If both `p` and `q` are `None`, they match at this position — return `True`.
2. If exactly one of them is `None` (not both), they differ in structure here — return `False`.
3. If both exist but `p.val != q.val`, return `False`.
4. Otherwise, return `True` only if **both** the left subtrees match (recursively) **and** the right subtrees match (recursively).

### Python code
```python
def isSameTree(p, q):
    if not p and not q:
        return True
    if not p or not q:
        return False
    if p.val != q.val:
        return False

    return isSameTree(p.left, q.left) and isSameTree(p.right, q.right)
```

### Line-by-line explanation

- `if not p and not q: return True` — both empty at this position — structurally matching (a `None` here in both trees is a valid, matching base case).
- `if not p or not q: return False` — **exactly one** is `None` (we already ruled out "both `None`" above, so reaching this line means at most one is `None`) — a structural mismatch: one tree has a node here, the other doesn't, which can never be "the same."
- `if p.val != q.val: return False` — both nodes exist, but hold different values — a value mismatch at this exact position.
- `return isSameTree(p.left, q.left) and isSameTree(p.right, q.right)` — both current nodes match; the trees are the same overall only if *both* corresponding subtrees also match. Python's `and` short-circuits, so if the left subtrees don't match, the right subtrees are never even checked — a real efficiency detail, not just a correctness one, since it means the function can terminate early on the first detected mismatch anywhere in the traversal.

### Dry run

`p = [1,2,3]`, `q = [1,2,3]` (identical trees)

- `isSameTree(1, 1)`: both exist, `1==1`, recurse.
  - `isSameTree(2, 2)`: both exist, `2==2`, recurse into their (both `None`) children → `isSameTree(None,None)=True` and `isSameTree(None,None)=True` → returns `True`.
  - `isSameTree(3, 3)`: similarly returns `True`.
  - `True and True` → `True`.
- Final: `True` ✅

**A mismatched dry run:** `p = [1,2]`, `q = [1,None,2]` (same *values* overall, but `2` is a *left* child in `p` and a *right* child in `q` — this is specifically chosen to show that value-matching alone isn't sufficient, structure matters too)
- `isSameTree(1,1)`: values match, recurse.
  - `isSameTree(p.left=2, q.left=None)`: one is `None`, the other isn't → `False`.
- Overall: `False` ✅ (correctly detects the structural difference, even though "the same values exist somewhere in both trees" — position matters, not just the multiset of values, unlike some Arrays & Hashing problems where order genuinely didn't matter).

### Time & space complexity

- **Time: O(min(n, m))** where n, m are the sizes of the two trees — in the worst case (the trees actually are identical), we visit every node of both; if a mismatch is found early, we stop early (short-circuiting via `and`), so the *worst* case is bounded by the smaller tree's traversal completing (or a mismatch being found, whichever comes first).
- **Space: O(min(h_p, h_q))** for the recursion call stack, bounded by the shallower tree's height (since the recursion effectively stops as soon as either tree runs out of a corresponding node to compare against).

---

## Approach 2: Iterative (Stack of Node Pairs)

### Intuition

Same comparison logic, but managed explicitly instead of via recursion — push **pairs** of corresponding nodes onto a stack, and check each pair as it's popped, mirroring the recursive version's "compare, then recurse on each side" structure but with an explicit work list instead of function calls.

### Python code
```python
def isSameTree(p, q):
    stack = [(p, q)]

    while stack:
        node1, node2 = stack.pop()

        if not node1 and not node2:
            continue
        if not node1 or not node2:
            return False
        if node1.val != node2.val:
            return False

        stack.append((node1.left, node2.left))
        stack.append((node1.right, node2.right))

    return True
```

### Line-by-line explanation

- `stack = [(p, q)]` — start with the pair of roots to compare.
- `node1, node2 = stack.pop()` — take the next pair to check.
- `if not node1 and not node2: continue` — both empty at this position — fine, nothing more to check here, move to the next pair on the stack (this doesn't end the loop, just skips further work for this particular pair).
- `if not node1 or not node2: return False` — mismatch (one exists, the other doesn't) — immediate, final answer.
- `if node1.val != node2.val: return False` — value mismatch — immediate, final answer.
- Push both pairs of children (left-with-left, right-with-right) for future comparison — this mirrors the recursive version's two recursive calls, just deferred onto the stack instead of made immediately.
- `return True` — if we exhaust the entire stack without ever returning `False`, every corresponding pair matched — the loop only terminates this way once every pair has been checked and passed.

*(A BFS version using a queue instead of a stack works identically — the order pairs are compared in doesn't affect correctness here, same as with Invert Binary Tree's DFS/BFS interchangeability.)*

### Time & space complexity

- **Time: O(min(n, m))**.
- **Space: O(min(n, m))** worst case for the explicit stack (in the worst case where the trees are identical, roughly half the tree's nodes could be sitting in the stack at once, waiting to be processed).

---

## Common mistakes & misconceptions

1. **Checking only values, not structure.** As the mismatched dry run above shows, two trees can contain exactly the same values but arranged differently — a solution that only compares, say, in-order traversals' *value sequences* (rather than recursively comparing actual structure) would incorrectly call some structurally-different trees "the same." This specific pitfall matters because in-order traversal of a *non-BST* tree carries no special ordering guarantee, so this mistake isn't even a reliable shortcut for BSTs either without extra care.
2. **Forgetting the `not p and not q` case entirely and jumping straight to `not p or not q`.** Without the "both None" check first, `not p or not q` would incorrectly return `False` whenever *both* are `None` (since `not None` is `True`, and `True or True` is `True`... wait — actually this specific ordering issue would incorrectly trigger `False` when both are None, since `not p or not q` evaluates to `True` when both are `None`) — the two checks must be separate and in the right order, precisely to distinguish "both empty" (a match) from "exactly one empty" (a mismatch).
3. **Not short-circuiting the two recursive calls with `and`.** Using two separate `if` statements instead of a single `and`-chained return works too, but the `and` version makes the short-circuit behavior (skip checking the right subtree if the left already failed) explicit and is the more idiomatic, commonly expected form.
4. **Assuming this problem needs to consider trees "the same" if they're mirror images of each other.** That's a genuinely different problem (Symmetric Tree, not in this list) — Same Tree requires identical left/right structure, not a left-right-swapped match.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursive DFS | O(min(n,m)) | O(min(h_p,h_q)) | Cleanest, most idiomatic solution. |
| Iterative (stack of pairs) | O(min(n,m)) | O(min(n,m)) | Same idea without recursion. |

**Key takeaway:** "compare two trees simultaneously" problems (Same Tree, and later Subtree of Another Tree) all follow this same shape: recurse on **corresponding pairs** of nodes from both trees at once, checking structure (both `None`, or both exist) and value equality at each step before recursing deeper — and always verify with a concrete example where values match but structure doesn't, since that's the specific case a naive value-only comparison gets wrong.
