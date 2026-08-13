# 43. Clone Graph

**LeetCode:** [#133 - Clone Graph](https://leetcode.com/problems/clone-graph/) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

Given a reference to a node in a **connected** undirected graph, return a **deep copy** (clone) of the entire graph — every node and every edge must be duplicated as new objects, not just referencing the originals.

```python
class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []
```

## Applicable approaches

- **DFS with a hash map (original node → cloned node)** — the most common approach.
- **BFS with a hash map** — equally valid alternative.

## Approach 1: DFS with a Hash Map

### Intuition

The tricky part of cloning a graph (as opposed to a tree) is that it can have **cycles** — if we naively clone a node, then clone its neighbors, then their neighbors, and so on, we could loop forever revisiting the same nodes, exactly the infinite-loop risk the topic overview's `visited` set is meant to prevent. But here we need more than just "don't revisit" — we need to actually **remember which clone corresponds to which original**, so that when a cycle brings us back to a node we've already started cloning, we correctly reconnect to the *same* clone instead of creating a duplicate. The fix: keep a hash map from **original node → already-created clone**, so that the moment we're asked to clone a node we've already started cloning, we simply return the **existing** clone instead of creating a new one or recursing again.

### Algorithm

1. Maintain a hash map `clones`: original node → its clone.
2. `dfs(node)`:
   - If `node` is already in `clones`, return the existing clone immediately (this is what prevents infinite loops on cycles).
   - Otherwise, create a new clone node (same value, empty neighbor list for now), and **immediately** store it in `clones` **before** recursing into neighbors (critical ordering — see explanation below).
   - For each of `node`'s original neighbors, recursively clone them (`dfs(neighbor)`), and append the result to the new clone's neighbor list.
   - Return the clone.

### Python code
```python
class Node:
    def __init__(self, val=0, neighbors=None):
        self.val = val
        self.neighbors = neighbors if neighbors is not None else []

def cloneGraph(node):
    if not node:
        return None

    clones = {}

    def dfs(original):
        if original in clones:
            return clones[original]

        clone = Node(original.val)
        clones[original] = clone  # store BEFORE recursing into neighbors

        for neighbor in original.neighbors:
            clone.neighbors.append(dfs(neighbor))

        return clone

    return dfs(node)
```

### Line-by-line explanation

- `clones = {}` — maps each original node (using the node object itself as the key, relying on default identity-based hashing, same as Linked List Cycle's `seen` set) to its already-created clone.
- `dfs(original)`: `if original in clones: return clones[original]` — **this is the cycle-safety check**: if we've already started (or finished) cloning this node, don't create a duplicate clone or recurse again — just hand back the one we already have.
- `clone = Node(original.val); clones[original] = clone` — **critically, we register the new clone in the map immediately, before recursing into its neighbors.** This ordering matters enormously: if node A and node B are neighbors of each other (a very common, even guaranteed-by-cycles situation), cloning A will recurse into cloning B, which will in turn try to clone A again as *B's* neighbor — if A's clone weren't already registered in the map at that point, this would recurse infinitely. Because we registered A's clone *before* recursing into its neighbors, that second attempt to clone A immediately finds it in `clones` and returns it directly, breaking the cycle.
- `for neighbor in original.neighbors: clone.neighbors.append(dfs(neighbor))` — recursively clone each neighbor (which might already exist in the map, or might need to be created now), and build up the new clone's own neighbor list to mirror the original's structure.
- `return clone`.

### Dry run

Graph: `1 -- 2`, `1 -- 3`, `2 -- 3` (a triangle, so cycles exist between all three nodes).

- `dfs(1)`: not in `clones`. Create `clone_1`, register `clones = {1: clone_1}`.
  - Loop over `1`'s neighbors `[2, 3]`:
    - `dfs(2)`: not in `clones`. Create `clone_2`, register `clones = {1: clone_1, 2: clone_2}`.
      - Loop over `2`'s neighbors `[1, 3]`:
        - `dfs(1)`: **`1` IS in `clones`** (registered in the very first call, before we ever got here) → return `clone_1` directly, no infinite recursion. `clone_2.neighbors = [clone_1]`.
        - `dfs(3)`: not in `clones`. Create `clone_3`, register `clones = {1:clone_1, 2:clone_2, 3:clone_3}`.
          - Loop over `3`'s neighbors `[1, 2]`:
            - `dfs(1)`: in `clones` → return `clone_1`.
            - `dfs(2)`: **in `clones`** (registered earlier in this same call chain, even though it's not "fully finished" yet — its own neighbor list is still being built) → return `clone_2`.
          - `clone_3.neighbors = [clone_1, clone_2]`. Return `clone_3`.
      - `clone_2.neighbors = [clone_1, clone_3]`. Return `clone_2`.
    - `clone_1.neighbors` gets `clone_2` appended.
    - `dfs(3)`: **already in `clones`** (fully built by now) → return `clone_3` directly.
  - `clone_1.neighbors = [clone_2, clone_3]`. Return `clone_1`.

Final cloned graph correctly mirrors the original triangle structure, using entirely new `Node` objects throughout, with no infinite recursion despite the cycles ✅ — and specifically note that node `2`'s clone was returned successfully at `dfs(2)` inside `dfs(3)`'s loop even though `clone_2.neighbors` wasn't yet fully populated at that moment — that's fine, because by the time anything actually *reads* `clone_2.neighbors`, the full recursion has completed and it's correctly filled in.

### Time & space complexity

- **Time: O(V + E)** where V = number of nodes, E = number of edges — each node is "created" exactly once, and each edge is traversed exactly once (from each direction, in an undirected graph, so effectively O(2E) = O(E)).
- **Space: O(V)** for the `clones` map, plus O(V) recursion depth in the worst case (a long chain-like graph).

---

## Approach 2: BFS with a Hash Map

### Intuition

The exact same "map original → clone, register before fully processing neighbors" idea, but using an explicit queue instead of recursion — the DFS-vs-BFS interchangeability from the Trees/Graphs topics, applied here to a "build a new structure while traversing" problem instead of a pure traversal.

### Python code
```python
from collections import deque

def cloneGraph(node):
    if not node:
        return None

    clones = {node: Node(node.val)}
    queue = deque([node])

    while queue:
        current = queue.popleft()
        for neighbor in current.neighbors:
            if neighbor not in clones:
                clones[neighbor] = Node(neighbor.val)
                queue.append(neighbor)
            clones[current].neighbors.append(clones[neighbor])

    return clones[node]
```

### Line-by-line explanation

- `clones = {node: Node(node.val)}` — seed the map with the starting node's clone already created, mirroring the DFS version's "register before recursing" discipline, just applied at the very start instead of at the top of each recursive call.
- `queue = deque([node])` — process nodes level by level (order doesn't actually matter for correctness here, same as Invert Binary Tree — BFS and DFS are interchangeable when the task is "visit and process every node," not order-dependent).
- For each neighbor of the current node: if we haven't created its clone yet, do so now and enqueue it for later processing of *its* neighbors.
- `clones[current].neighbors.append(clones[neighbor])` — regardless of whether the neighbor's clone was just created or already existed, link it into the current node's clone's neighbor list — this correctly rebuilds every edge, whether the neighbor was "new" this iteration or already registered from an earlier one.

### Time & space complexity

- **Time: O(V + E)**, **Space: O(V)** — same overall complexity as the DFS version.

---

## Common mistakes & misconceptions

1. **Registering the clone in the map *after* recursing into (or enqueueing) its neighbors, instead of before.** This is the single most consequential ordering detail in this whole problem — as the dry run shows, cyclic graphs (which are extremely common, even guaranteed by "undirected" edges creating a trivial A↔B cycle) will cause infinite recursion or repeated re-cloning without this ordering.
2. **Using a plain `visited` set instead of a `clones` map.** A `visited` set alone would tell you "don't process this node again," but it wouldn't tell you *which clone* to reconnect to — you specifically need the map's value (the clone reference), not just a boolean "have I seen this" flag, since the whole point is to *build* a new structure, not just traverse the old one.
3. **Assuming the graph could be disconnected and adding an outer loop "just in case."** The problem specifically guarantees the graph is **connected**, so a single DFS/BFS from the given starting node is guaranteed to reach every node — unlike Number of Islands, no outer "try every unvisited node" loop is needed or expected here.
4. **Confusing node *values* with node *identity* when checking the map.** The map keys on the actual node object references (relying on default identity-based hashing), not on `.val` — if two different original nodes happened to share the same value (which the problem's examples don't typically feature, but isn't explicitly forbidden), keying by value instead of identity would incorrectly merge them into a single clone.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DFS + hash map | O(V+E) | O(V) | Clean recursive solution; the "register before recursing" ordering is essential. |
| BFS + hash map | O(V+E) | O(V) | Same idea, avoids recursion depth concerns. |

**Key takeaway:** cloning (or transforming) any graph with possible cycles requires a **map from original to new**, and critically, that map entry must be created **before** recursing/processing into neighbors — this exact "register first, then recurse" ordering is what prevents infinite loops whenever a graph traversal needs to build a *new* structure that mirrors the original, not just visit nodes, and it's a genuinely different (stronger) requirement than the plain `visited` set used for traversal-only problems like Number of Islands.
