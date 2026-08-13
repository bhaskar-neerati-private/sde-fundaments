# 44. Pacific Atlantic Water Flow

**LeetCode:** [#417 - Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) · **Topic:** [Graphs](../topics/11-graphs.md) · **Difficulty:** Medium

## Problem statement

Given an `m x n` grid `heights` representing island terrain heights, the **Pacific Ocean** touches the left and top edges, and the **Atlantic Ocean** touches the right and bottom edges. Water flows from a cell to an adjacent cell only if the adjacent cell's height is **less than or equal to** the current cell's height (water flows downhill or on flat ground). Return the list of cells from which water can flow to **both** oceans.

## Applicable approaches

- **Brute Force — From every cell, check if it can reach both oceans (DFS/BFS per cell).**
- **Optimal — Reverse the Flow: DFS/BFS Outward from Each Ocean's Border.**

## Approach 1: Brute Force — From Every Cell, Check Both Oceans

### Intuition

For each individual cell, run a DFS/BFS following the "water flows downhill" rule outward from it, and check whether that exploration ever reaches the Pacific border (top or left edge) and separately whether it ever reaches the Atlantic border (bottom or right edge). This directly implements the problem statement, but treats every cell as an independent starting point for a full search, even though nearby cells' searches likely overlap enormously (a cell right next to another cell that can reach the Pacific will almost certainly also be able to reach it via a nearly-identical path).

### Why this is inefficient

This means running a full grid traversal **starting from every single cell** (m·n starting points), each traversal potentially visiting a large portion of the grid — leading to roughly O((m·n)²) time in the worst case. This is correct but very slow; the optimal approach below flips the direction of the search entirely to avoid this, rather than trying to patch the redundancy within this same per-cell-search structure.

---

## Approach 2: Optimal — Reverse the Flow, Search Outward from Each Ocean

### Intuition

Instead of asking, for every cell, "can water starting here reach the ocean?" (checking m·n cells, each potentially requiring a large search), we flip the question: **"starting FROM the ocean's border, which cells could water have flowed FROM, going uphill (the reverse of the actual flow direction)?"** This only requires **two searches total** — one starting from all Pacific-adjacent border cells, one starting from all Atlantic-adjacent border cells — each of which visits every reachable cell **at most once**. A cell reachable via *both* reversed searches is exactly a cell whose water can reach *both* oceans in the real, forward direction, since "can flow to X" and "X could have been flowed to from here" are logically the same relationship, just stated in opposite directions.

**Why reversing the flow condition works:** in the real world, water flows from a higher (or equal) cell to a lower (or equal) one. If we're searching *backward* from the ocean, we're asking "which cells could flow into where I currently am?" — which means we should move to a neighboring cell whenever that neighbor's height is **greater than or equal to** the current cell's height (the reverse of the original "flows downhill" condition) — since if the neighbor is higher, water *there* could have flowed *down* to here, which is exactly the forward-direction rule, just traversed backward.

### Algorithm

1. Create two `visited` sets: `pacific` and `atlantic`.
2. Start a DFS/BFS from **every cell along the Pacific border** (entire top row + entire left column) simultaneously, using the reversed flow condition (move to a neighbor if `neighbor_height >= current_height`), marking every reachable cell into the `pacific` set.
3. Do the same starting from **every cell along the Atlantic border** (entire bottom row + entire right column), marking reachable cells into the `atlantic` set.
4. The answer is every cell that appears in **both** sets.

### Python code
```python
def pacificAtlantic(heights):
    if not heights:
        return []

    rows, cols = len(heights), len(heights[0])
    pacific, atlantic = set(), set()

    def dfs(r, c, visited, prev_height):
        if (r < 0 or r >= rows or c < 0 or c >= cols
                or (r, c) in visited
                or heights[r][c] < prev_height):
            return
        visited.add((r, c))
        for dr, dc in [(1,0),(-1,0),(0,1),(0,-1)]:
            dfs(r + dr, c + dc, visited, heights[r][c])

    for c in range(cols):
        dfs(0, c, pacific, heights[0][c])         # top row -> Pacific
        dfs(rows - 1, c, atlantic, heights[rows-1][c])  # bottom row -> Atlantic

    for r in range(rows):
        dfs(r, 0, pacific, heights[r][0])          # left column -> Pacific
        dfs(r, cols - 1, atlantic, heights[r][cols-1])  # right column -> Atlantic

    return [[r, c] for r in range(rows) for c in range(cols) if (r, c) in pacific and (r, c) in atlantic]
```

### Line-by-line explanation

- `pacific, atlantic = set(), set()` — separately track which cells are reachable (via reversed flow) from each ocean's border, exactly the `visited` set pattern from the topic overview, just run twice with two independent sets.
- `dfs(r, c, visited, prev_height)` — explores outward from `(r, c)`, but only continues into a neighbor if that neighbor's height is **≥** the height we just came from (`heights[r][c] < prev_height` is the failure condition — meaning this cell is *too low* to have been flowed-from by the previous, higher cell, so real water couldn't have taken this path in reverse).
- The bounds/visited/height check combined — stop this branch if out of bounds, already visited (avoiding cycles/redundant work, same as any grid DFS per the topic overview), or this cell is lower than where we came from (violates the reversed flow condition).
- `visited.add((r, c))` then recurse into all 4 neighbors, passing `heights[r][c]` as the new `prev_height` for the next step.
- The two `for` loops seed the search: every cell along the **top row** and **left column** starts a Pacific search; every cell along the **bottom row** and **right column** starts an Atlantic search — note that all of these starting points share the *same* `pacific` (or `atlantic`) `visited` set, so the combined effect is one unified "reachable from Pacific" region, built up from multiple starting points at once (a multi-source search, per the topic overview's "Multi-source BFS" pattern, here applied with DFS instead), not m+n separate independent searches.
- Final list comprehension — any cell present in both `pacific` and `atlantic` can reach both oceans.

### Dry run (small example)

```
heights = [[1,2],
           [4,3]]
```
Pacific border: top row `(0,0),(0,1)` and left column `(0,0),(1,0)`.

- `dfs(0,0, pacific, heights[0][0]=1)`: `heights[0][0]=1 >= prev_height(1)` ✓ (starting cell always passes, since `prev_height` is initialized to its own height). Mark `(0,0)`. Try neighbors: `(1,0)` height=4, `4 >= 1` ✓ → recurse: mark `(1,0)`; its neighbors: `(1,1)` height=3, `3>=4`? No → stop. `(0,0)` already visited → stop. Back to `(0,0)`'s other neighbor `(0,1)` height=2, `2>=1` ✓ → recurse: mark `(0,1)`; neighbor `(1,1)` height=3, `3>=2` ✓ → recurse: mark `(1,1)`; its neighbors already visited or checked.
- Result after top-row/left-col Pacific seeding: `pacific` ends up containing all 4 cells (since this small grid happens to let Pacific-reversed-flow reach everywhere).

Atlantic border: bottom row `(1,0),(1,1)` and right column `(0,1),(1,1)`. A similar reversed search from these cells would also end up covering all 4 cells (following the reverse-flow logic upward/leftward from the higher-height regions).

Given both oceans can reach all 4 cells in this particular small grid, the final answer would be all 4 coordinates: `[[0,0],[0,1],[1,0],[1,1]]` (matches the known expected output pattern for this classic small example, where every cell in a 2x2 grid touching both starting borders directly ends up in both sets).

### Time & space complexity

- **Time: O(m · n)** — each of the two searches (Pacific and Atlantic) visits each cell at most once, regardless of how many border starting points feed into it (since the shared `visited` set prevents re-exploring a cell already reached from an earlier starting point) — this is the direct fix for Approach 1's O((m·n)²), since we've replaced "one search per cell" with "one search per ocean," and there are only two oceans regardless of grid size.
- **Space: O(m · n)** for the two visited sets, plus O(m · n) recursion depth in the worst case.

---

## Common mistakes & misconceptions

1. **Forgetting to reverse the flow condition.** Using the original "downhill" condition (`neighbor <= current`) instead of the reversed one (`neighbor >= current`) when searching *from* the ocean produces a completely different (and incorrect) set of reachable cells — always double-check which direction you're searching and whether the condition needs to be flipped accordingly.
2. **Running the Pacific and Atlantic searches with a *shared* visited set instead of two separate ones.** This would incorrectly prevent the Atlantic search from ever visiting a cell the Pacific search already claimed, when in reality a cell can legitimately be reachable from *both* oceans — the entire point of the problem is finding that overlap, which requires two independent sets.
3. **Believing this problem needs to check every cell as a starting point "to be safe," out of habit from the brute force.** The reversed-search insight specifically eliminates that need — trust the proof (searching backward from a fixed small set of border cells finds exactly the same reachability information as searching forward from every interior cell) rather than falling back to the exhaustive version.
4. **Off-by-one or wrong-edge errors when seeding the border cells.** Forgetting that "left column" means `c=0` for every row (not `r=0`), or mixing up which borders belong to which ocean — always re-derive from the problem statement ("Pacific touches left and top," "Atlantic touches right and bottom") rather than trusting memory.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force (per-cell search) | O((m·n)²) worst case | O(m·n) | Correct, but each cell potentially triggers a large redundant search. |
| Reverse flow from ocean borders | O(m · n) | O(m·n) | The standard, expected optimal solution. |

**Key takeaway:** when a problem asks "which starting points can reach a target region," check whether it's cheaper to **search backward from the target(s)** instead of forward from every possible starting point — this flips a potentially O(n²)-style "check every start" brute force into a single O(n) search (or a small constant number of O(n) searches, one per target region here), a very general and powerful trick beyond just this specific problem, and one worth recognizing whenever a problem's "check everything against everything" structure has a small, fixed number of true "destinations."
