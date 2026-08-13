# 42. Number of Islands

**LeetCode:** [#200 - Number of Islands](https://leetcode.com/problems/number-of-islands/) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

Given an `m x n` 2D grid of `'1'`s (land) and `'0'`s (water), return the number of **islands** — groups of adjacent (up/down/left/right, not diagonal) `'1'`s, surrounded by water.

**Example:**
```
grid = [
  ["1","1","0","0","0"],
  ["1","1","0","0","0"],
  ["0","0","1","0","0"],
  ["0","0","0","1","1"]
]
Output: 3
```

## Applicable approaches

- **DFS (flood fill)** — the most common approach.
- **BFS (flood fill)** — equally valid alternative.
- **Union-Find** — a third valid approach, useful to know for a broader class of "connectivity" problems.

## Approach 1: DFS (Flood Fill)

### Intuition

Scan every cell in the grid. Whenever we find an unvisited `'1'`, we've discovered a **new** island — increment the count, and then use DFS to "flood fill" outward from that cell, marking every connected `'1'` as visited, so we don't count any part of this same island again later in the scan. This treats the grid as an implicit graph (each land cell is a node, each adjacency between two land cells is an edge), directly applying the topic overview's "DFS/BFS with a visited set" pattern, with "connected component" being exactly "island."

### Algorithm

1. Initialize `count = 0`.
2. For every cell `(r, c)` in the grid: if it's `'1'` and not yet visited, increment `count`, then DFS outward from `(r, c)`, marking every reachable `'1'` (via up/down/left/right) as visited.
3. Return `count`.

### Python code
```python
def numIslands(grid):
    rows, cols = len(grid), len(grid[0])
    visited = set()
    count = 0

    def dfs(r, c):
        if (r < 0 or r >= rows or c < 0 or c >= cols
                or grid[r][c] == "0" or (r, c) in visited):
            return
        visited.add((r, c))
        dfs(r + 1, c)
        dfs(r - 1, c)
        dfs(r, c + 1)
        dfs(r, c - 1)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1" and (r, c) not in visited:
                count += 1
                dfs(r, c)

    return count
```

### Line-by-line explanation

- `visited = set()` — tracks every cell we've already determined belongs to some already-counted island — the exact same `visited` set mechanism the topic overview says is non-negotiable for graph traversal, since a grid-as-graph can absolutely have "cycles" in the sense of multiple paths between two cells.
- `dfs(r, c)` — marks the current cell as visited, then recursively does the same for every land neighbor — "flooding outward" through the entire connected island.
- The combined bounds/water/visited check — stop this branch of the flood fill if we've gone off the grid, hit water, or already visited this cell (prevents infinite loops, since two different paths could reach the same land cell — exactly the scenario the `visited` set exists to guard against).
- Outer double loop — the moment we find an unvisited `'1'`, it must be part of an island we haven't counted yet (if it were part of an already-counted island, it would already be in `visited`, since flood fill marks the *entire* connected island in one go) — increment `count`, then flood-fill to mark this whole island as visited before continuing the scan. This mirrors the topic overview's "outer loop tries every unvisited node" pattern for handling disconnected components (here, disconnected islands).

### Dry run

```
1 1 0
0 1 0
0 0 1
```
- `(0,0)='1'`, not visited → `count=1`. `dfs(0,0)`: mark `(0,0)`, recurse to `(1,0)`='0' (stop), `(-1,0)` out of bounds (stop), `(0,1)`='1' not visited → mark, recurse from there: `(1,1)`='1' not visited → mark, recurse further ((2,1)='0' stop, (1,2)='0' stop, (1,0) already visited stop, (0,1) already visited stop). `(0,1)`'s other neighbor `(0,2)`='0' stop. Back up, `(0,0)`'s neighbor `(0,-1)` out of bounds, stop. Flood fill for this island complete — marked `(0,0),(0,1),(1,1)`.
- Continue outer scan: `(0,1)` visited, skip. `(0,2)`='0' skip. `(1,0)`='0' skip. `(1,1)` visited, skip. `(1,2)`='0' skip. `(2,0)`,`(2,1)`='0' skip. `(2,2)`='1' not visited → `count=2`. `dfs(2,2)`: marks just this cell (all its neighbors are water or out of bounds).

Final: `count = 2` ✅ (matches the two separate islands in this smaller example).

### Time & space complexity

- **Time: O(m · n)** — every cell is visited a constant number of times total (once as part of the outer scan, and at most once as part of some island's flood fill, since once marked visited a cell is never processed again).
- **Space: O(m · n)** worst case for the `visited` set (if the entire grid is one big island) and the recursion call stack.

---

## Approach 2: BFS (Flood Fill)

### Intuition

Identical logic to DFS, just using a queue instead of recursion to explore each island — visits the same cells, just in a different order (level by level outward from the starting cell, rather than depth-first), the exact DFS-vs-BFS interchangeability the Trees topic first demonstrated with Invert Binary Tree, now applied to graphs.

### Python code
```python
from collections import deque

def numIslands(grid):
    rows, cols = len(grid), len(grid[0])
    visited = set()
    count = 0

    def bfs(r, c):
        queue = deque([(r, c)])
        visited.add((r, c))
        while queue:
            row, col = queue.popleft()
            for dr, dc in [(1,0),(-1,0),(0,1),(0,-1)]:
                nr, nc = row + dr, col + dc
                if (0 <= nr < rows and 0 <= nc < cols
                        and grid[nr][nc] == "1" and (nr, nc) not in visited):
                    visited.add((nr, nc))
                    queue.append((nr, nc))

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1" and (r, c) not in visited:
                count += 1
                bfs(r, c)

    return count
```

### Line-by-line explanation

- `bfs(r, c)` — starts a queue with the initial cell (already marked visited), and repeatedly dequeues a cell, checking all 4 neighbors — any unvisited land neighbor gets marked visited and enqueued for future processing.
- **Important detail:** we mark a cell visited **at the moment we enqueue it**, not when we dequeue it — this prevents the same cell from being added to the queue multiple times by different neighbors before it's actually processed (a subtle but important difference from some BFS implementations, and worth being deliberate about, since two different already-queued cells could otherwise both discover the same third cell as their neighbor before either has been dequeued).
- The rest of the structure (outer scan, counting new islands) is identical to the DFS version.

### Time & space complexity

- **Time: O(m · n)**, **Space: O(m · n)** worst case for the `visited` set and the queue — same overall complexity as DFS, just different constant-factor behavior and no recursion depth concerns (BFS avoids any risk of hitting a recursion depth limit on a very large connected island, unlike the DFS version).

---

## Approach 3: Union-Find

### Intuition

Union-Find (Disjoint Set Union) is a data structure specifically designed for "group things together and ask if two things are in the same group" problems, per the Graphs topic overview. Treat every land cell as its own initial group; whenever two adjacent cells are both land, **union** their groups together. At the end, the number of distinct remaining groups (among land cells) is the number of islands — this reframes "count connected components" as "count how many groups remain after merging everything that should be merged," a genuinely different mechanism from flood-fill traversal.

### Python code
```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.count = 0  # number of distinct groups (islands), tracked incrementally

    def find(self, x):
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]  # path compression
            x = self.parent[x]
        return x

    def union(self, x, y):
        root_x, root_y = self.find(x), self.find(y)
        if root_x != root_y:
            self.parent[root_x] = root_y
            self.count -= 1  # merging two groups reduces the total count by 1

def numIslands(grid):
    rows, cols = len(grid), len(grid[0])
    uf = UnionFind(rows * cols)

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                uf.count += 1  # each land cell starts as its own island

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                index = r * cols + c
                for dr, dc in [(1, 0), (0, 1)]:  # only check right & down to avoid double-processing
                    nr, nc = r + dr, c + dc
                    if nr < rows and nc < cols and grid[nr][nc] == "1":
                        uf.union(index, nr * cols + nc)

    return uf.count
```

### Time & space complexity

- **Time: O(m · n · α(m·n))** where α is the inverse Ackermann function (grows so slowly it's effectively a small constant for any realistic input size) — `find` and `union` with path compression are extremely close to O(1) amortized.
- **Space: O(m · n)** for the `parent` array.

---

## Common mistakes & misconceptions

1. **Forgetting to check bounds before checking `grid[r][c]`.** Accessing `grid[-1][c]` in Python doesn't raise an error (Python allows negative indexing, wrapping around to the last row!) — this can silently produce wrong results instead of crashing, making it a genuinely dangerous bug to miss; always check bounds *before* indexing, and check them in the right order (bounds first, then the actual value lookup) as the combined condition in Approach 1 does.
2. **Marking a cell visited in `visited.add()` but then re-checking it against `grid[r][c] == "0"` using a *different* mutation strategy elsewhere** (e.g. mixing an approach that also overwrites `grid[r][c] = "0"` in-place with the separate `visited` set) — pick one tracking mechanism and use it consistently; mixing them is a common source of subtle bugs.
3. **In the Union-Find approach, only checking two of the four directions (right and down) and assuming this misses connections.** It doesn't — checking only right and down for every cell still discovers every adjacency in the grid exactly once (each edge between two cells is "owned" by whichever check considers it first when scanning left-to-right, top-to-bottom), avoiding the redundant double-checking that scanning all 4 directions from every cell would cause.
4. **Assuming DFS will always be safe from recursion depth issues.** For a very large, fully-connected grid (e.g. a huge single island), the DFS recursion can go extremely deep (up to m·n frames) — the BFS or Union-Find versions avoid this risk entirely, which is worth considering for grids with very large dimensions.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DFS flood fill | O(m·n) | O(m·n) | Simple, most common answer. |
| BFS flood fill | O(m·n) | O(m·n) | Same complexity, avoids recursion depth risk. |
| Union-Find | O(m·n·α(m·n)) | O(m·n) | Overkill for this exact problem, but the standard tool for problems involving *dynamically* merging groups. |

**Key takeaway:** "count connected groups" problems are the classic use case for DFS/BFS flood fill — always mark cells visited as you explore, and the outer scan-and-count loop only ever "discovers" the first cell of each new group, since flood fill immediately claims the rest. Union-Find is worth knowing as an alternative, and becomes the *preferred* tool specifically when connectivity needs to be tracked incrementally as edges are added over time, rather than computed once from a static grid.
