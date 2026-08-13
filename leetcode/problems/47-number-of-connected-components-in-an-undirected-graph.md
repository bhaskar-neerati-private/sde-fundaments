# 47. Number of Connected Components in an Undirected Graph

**LeetCode:** [#323 - Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) (Premium) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

Given `n` nodes labeled `0` to `n-1` and a list of undirected `edges`, return the number of **connected components** in the graph.

## Applicable approaches

- **DFS/BFS with an outer loop over unvisited nodes** — direct extension of Number of Islands' pattern.
- **Union-Find** — arguably the more natural fit for this specific problem, and the one worth mastering here since this problem is the canonical Union-Find teaching example.

## Approach 1: DFS/BFS with Outer Loop

### Intuition

This is structurally identical to Number of Islands, just with an explicit adjacency list instead of an implicit grid-based one: every time the outer scan finds a node that hasn't been visited yet, that node must belong to a **new** connected component (if it belonged to an already-discovered component, it would already be marked visited, since a full DFS/BFS from any node marks its *entire* component in one pass) — increment the count, then flood-fill (DFS/BFS) to mark every node in that component as visited before continuing the outer scan.

### Python code
```python
def countComponents(n, edges):
    graph = {i: [] for i in range(n)}
    for a, b in edges:
        graph[a].append(b)
        graph[b].append(a)

    visited = set()
    count = 0

    def dfs(node):
        visited.add(node)
        for neighbor in graph[node]:
            if neighbor not in visited:
                dfs(neighbor)

    for node in range(n):
        if node not in visited:
            count += 1
            dfs(node)

    return count
```

### Line-by-line explanation

- Build the undirected adjacency list — same double-append discipline as Graph Valid Tree.
- The outer `for node in range(n)` loop with the `if node not in visited` guard is exactly Number of Islands' "outer scan discovers the first cell of each new group" pattern, just operating on explicit node labels instead of grid coordinates.
- Each `dfs(node)` call, once triggered, marks every node reachable from `node` — i.e., the entire component — as visited, guaranteeing the outer loop never double-counts a component.

### Time & space complexity

- **Time: O(V + E)** — every node is visited once (each triggering at most one DFS call from the outer loop, since after that it's marked visited), and every edge is traversed a constant number of times total across all the DFS calls combined.
- **Space: O(V + E)** for the graph, plus O(V) for `visited` and recursion depth.

---

## Approach 2: Union-Find

### Intuition

Start with `n` separate components (every node is its own group, since we haven't processed any edges yet). Each time we union two nodes that are in **different** groups, we've merged two components into one — reducing the total component count by exactly 1. If the two nodes are *already* in the same group, the edge is redundant (doesn't reduce the count further, since they were already counted as one component). This is precisely the same "count decreases only on a genuine merge" logic used in Number of Islands' Union-Find variant, just without the grid — a more general and arguably clearer illustration of why Union-Find naturally tracks a running component count as edges are added incrementally, exactly the use case the topic overview highlights Union-Find for ("especially when edges are being added incrementally").

### Python code
```python
def countComponents(n, edges):
    parent = list(range(n))
    count = n  # start with every node as its own component

    def find(x):
        while parent[x] != x:
            parent[x] = parent[parent[x]]  # path compression
            x = parent[x]
        return x

    def union(x, y):
        nonlocal count
        root_x, root_y = find(x), find(y)
        if root_x != root_y:
            parent[root_x] = root_y
            count -= 1  # two components merged into one

    for a, b in edges:
        union(a, b)

    return count
```

### Line-by-line explanation

- `count = n` — every node starts as its own isolated component, before any edges are processed.
- `find(x)` — walks up the parent pointers to the representative ("root") of `x`'s current group, with **path compression** (`parent[x] = parent[parent[x]]`) flattening the structure as we go, so future `find` calls on the same or nearby nodes are faster — this is what keeps Union-Find operations close to O(1) amortized rather than degrading toward O(n) per call on a long chain.
- `union(x, y)`: if `x` and `y` are already in the same group (`root_x == root_y`), do nothing — this edge doesn't merge anything new. Otherwise, attach one root under the other and **decrement `count`** — this is the key bookkeeping insight: we don't need a separate final pass to count distinct roots; we can track the count incrementally, decreasing by exactly 1 on every genuine merge.
- Process every edge with `union` — by the end, `count` holds exactly the number of distinct connected components.

### Dry run

`n = 5`, `edges = [[0,1],[1,2],[3,4]]`.

- `parent = [0,1,2,3,4]`, `count = 5`.
- `union(0,1)`: `find(0)=0`, `find(1)=1`, different → `parent[0]=1` → `parent=[1,1,2,3,4]`, `count=4`.
- `union(1,2)`: `find(1)=1`, `find(2)=2`, different → `parent[1]=2` → `parent=[1,2,2,3,4]`, `count=3`.
- `union(3,4)`: `find(3)=3`, `find(4)=4`, different → `parent[3]=4` → `parent=[1,2,2,4,4]`, `count=2`.

Final `count = 2` ✅ — two components: `{0,1,2}` and `{3,4}`.

### Time & space complexity

- **Time: O((V + E) · α(V))** — near-linear, thanks to path compression (and, in a fuller implementation, union by rank/size, though even path compression alone gets very close to O(1) amortized per operation).
- **Space: O(V)** for the `parent` array.

---

## Common mistakes & misconceptions

1. **Forgetting to initialize `count = n`** (or equivalently computing it as "number of distinct roots" at the end via a separate pass) — either works, but the incremental-decrement version shown here is the cleaner, more standard idiom, and it's worth understanding *why* it's correct (every genuine merge reduces the true component count by exactly 1, no more, no less) rather than just memorizing it.
2. **Skipping path compression in `find`.** Without it, Union-Find degrades toward O(n) per `find` call in the worst case (e.g., if unions happen to build a long chain), losing the near-O(1) amortized guarantee — path compression (or union by rank, or both) is what makes Union-Find genuinely efficient rather than just "a different way to write DFS."
3. **Using DFS/BFS and assuming it's meaningfully worse than Union-Find for this static problem.** For a *fixed*, one-time list of edges (as given here), DFS/BFS and Union-Find have the same overall O(V+E) complexity class — Union-Find's real advantage emerges specifically when edges arrive **incrementally over time** and you need to answer "how many components right now?" after each one, which a from-scratch DFS/BFS would have to redo entirely each time, but Union-Find handles by just continuing to merge.
4. **Forgetting the graph is undirected when building the adjacency list for the DFS approach** — same double-append reminder as Graph Valid Tree and the topic overview's general undirected-graph warning.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DFS/BFS + outer loop | O(V+E) | O(V+E) | Direct analog of Number of Islands, using explicit adjacency lists. |
| Union-Find | O((V+E)·α(V)) | O(V) | The canonical teaching example for Union-Find's incremental component counting. |

**Key takeaway:** this problem is essentially "Number of Islands without the grid" — the exact same connected-components-counting idea, now on an explicit adjacency list instead of implicit grid adjacency, and it's the cleanest place to see *why* Union-Find tracks component count incrementally (`count -= 1` on every genuine merge) rather than needing a separate final counting pass.
