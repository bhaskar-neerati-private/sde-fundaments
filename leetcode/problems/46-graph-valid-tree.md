# 46. Graph Valid Tree

**LeetCode:** [#261 - Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) (Premium) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

Given `n` nodes labeled `0` to `n-1` and a list of **undirected** `edges`, determine whether these edges form a **valid tree**.

## Applicable approaches

- **Two necessary-and-sufficient structural checks + DFS/BFS connectivity check.**
- **Union-Find** — an equally natural fit, arguably more direct for this specific problem.

## Approach 1: Edge-Count Check + DFS Connectivity Check

### Intuition

A valid tree, by definition (per the topic overview's "How this compares to Trees" section), must be **connected** (every node reachable from every other) and have **no cycles**. There's a beautifully simple structural fact that lets us check "no cycles" without any complicated cycle-detection logic: **a graph with `n` nodes is a tree if and only if it has exactly `n - 1` edges AND is connected.** Why exactly `n-1`? Because a tree is the *minimal* connected structure — start with 1 isolated node (0 edges, trivially connected to itself) and add nodes one at a time; each new node needs exactly one new edge to attach it to the existing structure, so after adding `n-1` more nodes, we've used exactly `n-1` edges. Any *fewer* edges than `n-1` and the graph can't possibly be connected (not enough edges to reach every node). Any *more* than `n-1` edges, given the graph is already connected with `n-1` edges' worth of "reach," necessarily creates a redundant connection — a second path between two already-connected nodes — which is precisely a cycle.

So the check becomes two simple, independent pieces:
1. `len(edges) == n - 1` (necessary edge count for a tree — cheap to check first, and immediately disqualifies many invalid inputs without even needing a traversal).
2. The graph is connected (a single DFS/BFS from any node reaches all `n` nodes) — because `n-1` edges *without* being connected would mean some component has "too many" edges relative to its own size (implying a cycle within that component) while another component has too few nodes reachable at all.

### Algorithm

1. If `len(edges) != n - 1`, return `False` immediately (can't be a tree — either too sparse to be connected, or has a cycle).
2. Build an adjacency list (undirected — add each edge to *both* endpoints' neighbor lists, per the topic overview's "undirected graph" reminder).
3. DFS/BFS from node `0`, tracking visited nodes.
4. Return `True` if and only if the traversal visits all `n` nodes.

### Python code
```python
def validTree(n, edges):
    if len(edges) != n - 1:
        return False

    graph = {i: [] for i in range(n)}
    for a, b in edges:
        graph[a].append(b)
        graph[b].append(a)

    visited = set()

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

    dfs(0)
    return len(visited) == n
```

### Line-by-line explanation

- `if len(edges) != n - 1: return False` — the cheap structural pre-check described above; note this single check *simultaneously* rules out both "too few edges to be connected" and "too many edges (implying a cycle)" — it's doing double duty, which is why it's such a powerful one-line filter.
- `graph[a].append(b); graph[b].append(a)` — undirected edge, added both directions, per the topic overview's explicit warning about this exact mistake.
- `dfs(0)` — arbitrary starting node; since we're only checking "is everything reachable from *some* single starting point," and a tree (if valid) is connected, starting anywhere is equivalent — no need for the "outer loop over every unvisited node" pattern used in Number of Islands, because we're not counting components, just confirming there's exactly one.
- `return len(visited) == n` — if the DFS from node 0 reached every node, the graph is connected; combined with the earlier `n-1` edge check, this fully confirms it's a valid tree. Note: **we don't need a separate cycle check here** — the `n-1` edge count check already rules out cycles *given* that we also verify connectivity, which is precisely why this problem doesn't need the `in_path`-style cycle detection from Course Schedule.

### Dry run

`n = 5`, `edges = [[0,1],[0,2],[0,3],[1,4]]` (4 edges, `n-1 = 4` ✓ passes the count check).

`graph = {0:[1,2,3], 1:[0,4], 2:[0], 3:[0], 4:[1]}`.

- `dfs(0)`: visit `0`. Neighbors `[1,2,3]`:
  - `dfs(1)`: visit `1`. Neighbors `[0,4]`: `0` already visited, skip. `dfs(4)`: visit `4`. Neighbors `[1]`: already visited, skip.
  - `dfs(2)`: visit `2`. No unvisited neighbors.
  - `dfs(3)`: visit `3`. No unvisited neighbors.
- `visited = {0,1,2,3,4}`, `len(visited) = 5 == n` ✓ → **`True`**, a valid tree.

Contrast with `edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]` (5 edges, but `n-1 = 4`) → the count check alone immediately returns `False` without needing any traversal — this input has a cycle among nodes 1, 2, 3.

### Time & space complexity

- **Time: O(n)** — the edge-count check is O(1) (given `len()` is O(1) for a list), and the DFS visits each of the `n` nodes and `n-1` edges at most once (recall `len(edges)` is already fixed at `n-1` by the time we even start the DFS, thanks to the pre-check).
- **Space: O(n)** for the adjacency list and `visited` set.

---

## Approach 2: Union-Find

### Intuition

An arguably even more direct fit: process edges one at a time, and **union** the two endpoints' groups. If, while doing this, we ever try to union two nodes that are **already** in the same group, that new edge is redundant — it connects two nodes that were already reachable from each other — which is exactly a cycle, detected the instant it would be created, without needing any separate traversal step afterward.

### Python code
```python
def validTree(n, edges):
    if len(edges) != n - 1:
        return False

    parent = list(range(n))

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]
            x = parent[x]
        return x

    def union(x, y):
        root_x, root_y = find(x), find(y)
        if root_x == root_y:
            return False  # already connected -> this edge would create a cycle
        parent[root_x] = root_y
        return True

    for a, b in edges:
        if not union(a, b):
            return False

    return True
```

### Line-by-line explanation

- The `len(edges) != n - 1` pre-check is kept for the same reason as before — cheap and catches the "too many edges" case even faster than letting Union-Find discover it edge by edge (though Union-Find alone, without this check, would still correctly reject a cycle-containing edge list; the pre-check is a helpful but not strictly required optimization here since a cycle would trigger `union` returning `False` regardless).
- `union(x, y)`: if `x` and `y` already share the same root (`find(x) == find(y)`), they're already connected — adding this edge would be redundant, i.e., create a cycle — return `False` to signal that.
- The main loop processes every edge, and if *any* edge would create a cycle, we immediately know it's not a valid tree.
- If we make it through all edges without any redundant union, combined with the `n-1` edge count, the structure is confirmed to be both cycle-free and (since exactly `n-1` non-redundant unions among `n` nodes necessarily merges everything into a single group) fully connected.

### Time & space complexity

- **Time: O(n · α(n))** — near-linear, thanks to path compression in `find`.
- **Space: O(n)** for the `parent` array.

---

## Common mistakes & misconceptions

1. **Forgetting the `n - 1` edge count check and relying purely on a connectivity check.** A graph can be connected (DFS reaches every node) while *also* having a cycle (extra redundant edges) — connectivity alone doesn't confirm "no cycles"; you need both conditions, not just one. (In the Union-Find version, the redundant-edge detection substitutes for this, but if you dropped that check too, you could incorrectly validate a graph with more than `n-1` edges as a tree.)
2. **Adding an edge only to one node's adjacency list** (forgetting the graph is undirected) — silently breaks connectivity checks, since a DFS might fail to find a valid path that actually exists in the real, symmetric graph.
3. **Assuming a "tree" input can't have self-loops or duplicate edges and skipping validation.** Most versions of this problem guarantee clean input, but it's worth knowing that duplicate edges or self-loops would also be caught correctly by this same logic (a duplicate edge is exactly a redundant union / would push the edge count above `n-1`).
4. **Trying to adapt the Course Schedule-style `in_path` cycle detection here**, which is built for **directed** graphs. For an **undirected** graph, the correct cycle-detection subtlety is different: you must track the parent you just came from, since walking back to your immediate parent along the same edge is *not* a cycle (it's just the edge you arrived on), but reaching any *other* already-visited node is. The edge-count trick used in this problem elegantly sidesteps needing that logic entirely.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Edge count + DFS connectivity | O(n) | O(n) | Clean, exploits the `n-1` edges ⟺ tree structural fact. |
| Union-Find | O(n·α(n)) | O(n) | Detects the disqualifying cycle the instant a redundant edge is unioned. |

**Key takeaway:** "is this a valid tree" reduces to two independent, checkable facts (exactly `n-1` edges, and connected) rather than needing general-purpose directed-graph cycle detection — recognizing when a special structural property (like the edge-count identity here) replaces a more complex general algorithm is a recurring theme worth watching for across graph problems.
