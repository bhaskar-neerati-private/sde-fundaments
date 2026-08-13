# 38. Word Search

**LeetCode:** [#79 - Word Search](https://leetcode.com/problems/word-search/) · **Topic:** [Backtracking](../topics/09-backtracking.md) · **Difficulty:** Medium

## Problem statement

Given an `m x n` grid of characters `board` and a string `word`, return `true` if `word` exists in the grid, formed by moving to **adjacent** cells (up/down/left/right, not diagonal), **without reusing the same cell twice** within one path.

**Example:**
```
board = [["A","B","C","E"],
         ["S","F","C","S"],
         ["A","D","E","E"]]
word = "ABCCED"
Output: true
```

## Applicable approaches

- **Backtracking with a separate `visited` set.**
- **Optimal — Backtracking with In-Place Cell Marking (no extra set needed).**

## Approach 1: Backtracking with a Visited Set

### Intuition

Try starting the search from every cell in the grid. From each starting cell, recursively try to match the word character by character, moving to an adjacent, not-yet-visited cell for each next character — backtracking (undoing the visited mark, per the topic overview's core discipline) whenever a path doesn't work out, so other paths can still use that cell.

### Algorithm

1. For each cell `(r, c)` in the grid, try starting the search there.
2. `dfs(r, c, index)`: if `index == len(word)`, we've matched every character — success.
3. If `(r, c)` is out of bounds, or `board[r][c] != word[index]`, or `(r, c)` is already visited (as part of the *current* path), this path fails — return `False`.
4. Otherwise, mark `(r, c)` as visited, and try extending the path into each of the 4 neighboring cells for `word[index + 1]`. If any neighbor succeeds, this whole path succeeds.
5. **Unmark** `(r, c)` as visited before returning (whether success or failure) — this is the backtracking step, letting *other* starting paths reuse this cell.

### Python code
```python
def exist(board, word):
    rows, cols = len(board), len(board[0])
    visited = set()

    def dfs(r, c, index):
        if index == len(word):
            return True
        if (r < 0 or r >= rows or c < 0 or c >= cols
                or board[r][c] != word[index]
                or (r, c) in visited):
            return False

        visited.add((r, c))

        found = (dfs(r + 1, c, index + 1) or
                 dfs(r - 1, c, index + 1) or
                 dfs(r, c + 1, index + 1) or
                 dfs(r, c - 1, index + 1))

        visited.remove((r, c))  # undo - backtrack
        return found

    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True
    return False
```

### Line-by-line explanation

- `visited = set()` — tracks which cells are part of the **current** path being explored (not a global "ever visited" tracker — it gets cleared as we backtrack, so different starting points/paths can freely reuse the same cells, exactly matching the topic overview's "same cell might legitimately need to be revisited via a different path" point).
- `dfs(r, c, index)` — are we able to match `word[index:]` starting from cell `(r, c)`?
- `if index == len(word): return True` — matched every character of `word` — success (base case).
- The combined bounds/mismatch/visited check — three separate failure conditions checked together: out of the grid, wrong character at this cell, or this cell is already used earlier in the *current* path (can't revisit within one path, per the problem's rule).
- `visited.add((r, c))` — mark this cell as "in use" for the current path, before exploring further.
- `found = (dfs(r+1,c,...) or dfs(r-1,c,...) or dfs(r,c+1,...) or dfs(r,c-1,...))` — try all 4 directions; Python's `or` short-circuits, so the moment one direction succeeds, the others aren't even tried — a genuine efficiency detail, not just style.
- `visited.remove((r, c))` — **the backtracking step**: whether this path succeeded or failed, un-mark this cell before returning, so it's available again for other paths — either different neighbors tried by the caller, or entirely different starting cells tried by the outer loop.
- `return found` — propagate the result up.
- Outer double loop — try starting the search from every single cell, since the word could begin anywhere in the grid, and we have no way to know in advance which starting cell (if any) leads to success.

### Dry run (abbreviated)

`board` as given, `word = "ABCCED"`

Starting at `(0,0)` = `'A'` matches `word[0]='A'`. Mark visited. Try neighbors for `word[1]='B'`:
- Right `(0,1)='B'` matches! Mark visited. Try neighbors for `word[2]='C'`:
  - Right `(0,2)='C'` matches! Mark visited. Try neighbors for `word[3]='C'`:
    - Down `(1,2)='C'` matches! Mark visited. Try neighbors for `word[4]='E'`:
      - Down `(2,2)='E'` matches! Mark visited. Try neighbors for `word[5]='D'`:
        - Left `(2,1)='D'` matches! Mark visited. `index` now `6 == len(word)` → return `True` all the way up.

Path found: `A(0,0)->B(0,1)->C(0,2)->C(1,2)->E(2,2)->D(2,1)` ✅ (matches the known valid path for this classic example). Every cell along the successful path stays marked only *until the function returns* — if this particular path had failed partway, each cell would be unmarked as the recursion unwound, allowing alternate directions to be tried from earlier points in the path, exactly the "undo, then try the next alternative" backtracking rhythm.

### Time & space complexity

- **Time: O(m · n · 4^L)** where m,n = grid dimensions, L = word length — in the worst case, we try starting from every cell (m·n), and from each, explore up to 4 directions at each of the L steps (though in practice, the "already visited in this path" and "character mismatch" checks prune this significantly, since most cells won't match the required character at each step).
- **Space: O(L)** for the recursion depth (bounded by the word's length) plus O(L) for the `visited` set (at most L cells are ever marked at once, since that's the maximum path length before either success or exhausting the word).

---

## Approach 2: Optimal — In-Place Marking (No Extra Set)

### Intuition

Instead of maintaining a separate `visited` set, we can temporarily **overwrite** the board's own cell value with a sentinel character (something that can never match any letter in `word`, like `'#'`) while it's part of the current path, and restore its original value when backtracking. This avoids the overhead of set operations (hashing tuples) entirely, working directly on the existing grid structure — the same "reuse existing structure instead of allocating a separate tracker" idea seen in Set Matrix Zeroes (Math & Geometry topic) and the running preorder pointer in Construct Binary Tree.

### Python code
```python
def exist(board, word):
    rows, cols = len(board), len(board[0])

    def dfs(r, c, index):
        if index == len(word):
            return True
        if (r < 0 or r >= rows or c < 0 or c >= cols
                or board[r][c] != word[index]):
            return False

        temp = board[r][c]
        board[r][c] = "#"  # mark as "in use" directly on the board

        found = (dfs(r + 1, c, index + 1) or
                 dfs(r - 1, c, index + 1) or
                 dfs(r, c + 1, index + 1) or
                 dfs(r, c - 1, index + 1))

        board[r][c] = temp  # undo - restore the original character

        return found

    for r in range(rows):
        for c in range(cols):
            if dfs(r, c, 0):
                return True
    return False
```

### Line-by-line explanation

- The bounds/mismatch check no longer needs an `or (r, c) in visited` clause — instead, once a cell is temporarily overwritten with `'#'`, the *mismatch* check (`board[r][c] != word[index]`) naturally fails for it too, since `'#'` will never equal any real letter in `word` (assuming the word itself never contains `'#'`, which is safe for this problem's typical constraints of lowercase/uppercase letters) — the visited-tracking and the mismatch-checking have been merged into a single mechanism.
- `temp = board[r][c]; board[r][c] = "#"` — save the original character, then overwrite it as a "currently in use" marker.
- Same 4-direction exploration as before.
- `board[r][c] = temp` — **the backtracking/undo step**: restore the cell's true value before returning, whether the path succeeded or failed — critical so the grid is back to its original state for other exploration attempts, both other directions from a parent call, and entirely different starting cells from the outer loop.

### Why this is often preferred

Avoiding the `visited` set means avoiding the overhead of hashing `(r, c)` tuples and set insert/remove operations — directly mutating the grid (and restoring it) is typically faster in practice, even though the asymptotic time complexity is the same. It does, however, require read/write access to mutate the input grid temporarily, which is acceptable here since we always fully restore it before the function returns — worth noting as a design trade-off (mutating input, even temporarily, isn't always allowed or desirable in every context; some interviewers or codebases specifically forbid mutating function arguments).

### Time & space complexity

- **Time: O(m · n · 4^L)** — same as Approach 1.
- **Space: O(L)** for the recursion depth only — no separate `visited` set needed, a small but real space improvement, since we're no longer paying for hash-set overhead on top of the recursion.

---

## Common mistakes & misconceptions

1. **Forgetting to restore the cell after marking it (both approaches).** Without the undo step — either `visited.remove()` or restoring `board[r][c] = temp` — later starting points (or later branches from the same starting point) would incorrectly treat a cell from a *previous, failed* attempt as permanently unusable, causing valid paths to be missed.
2. **Using a global "ever visited" set instead of a per-path set.** This is a fundamentally different (and incorrect) tracking scheme — a cell used in one failed path attempt must be available again for a completely different path attempt starting elsewhere, which a global "ever visited anywhere" tracker would incorrectly forbid.
3. **In the in-place marking version, assuming `word` could contain `'#'` and not accounting for it.** If the word could genuinely contain the sentinel character, the mismatch check could falsely succeed on an already-marked cell — this specific approach implicitly relies on `'#'` being a character that can't appear in valid input, which is safe for this problem's usual constraints but worth being aware of as an assumption, not a universal guarantee.
4. **Not short-circuiting the 4-direction `or` chain**, e.g. by computing all four `dfs()` calls into separate variables first and then combining them with `or` afterward — this defeats the point of Python's short-circuit evaluation, doing unnecessary exploration even after an earlier direction already succeeded.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Backtracking + visited set | O(m·n·4^L) | O(L) | Clear and explicit about what "visited" means. |
| Backtracking + in-place marking | O(m·n·4^L) | O(L) | Same complexity, avoids extra set overhead; standard "optimal" answer. |

**Key takeaway:** for grid-based backtracking, temporarily marking cells **in place** (overwriting with a sentinel value, then restoring) is a common and slightly more efficient alternative to a separate `visited` set — both rely on the same essential backtracking discipline from the topic overview: mark before recursing, unmark after, no matter how the recursive call turned out, because the shared grid structure must return to a clean state before any sibling branch or starting point can correctly use it.
