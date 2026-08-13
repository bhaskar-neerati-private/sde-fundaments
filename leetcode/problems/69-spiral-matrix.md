# 69. Spiral Matrix

**LeetCode:** [#54 - Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) · **Topic:** [Math & Geometry](../topics/17-math-geometry.md) · **Difficulty:** Medium

## Problem statement

Given an `m x n` matrix, return all its elements in **spiral order** (starting top-left, going right, then down, then left, then up, spiraling inward).

**Example:**
```
Input: [[1,2,3],
        [4,5,6],
        [7,8,9]]
Output: [1,2,3,6,9,8,7,4,5]
```

## Applicable approaches

- **Recursive Ring Peeling** - process the outer ring, then recurse on the smaller inner sub-matrix.
- **Optimal - Iterative Shrinking Boundaries.** The standard, expected, more commonly used solution.

## Approach 1: Recursive Ring Peeling

### Intuition
Process the outermost "ring" of the current sub-matrix (top row left-to-right, right column top-to-bottom, bottom row right-to-left if it's a different row, left column bottom-to-top if it's a different column), then recursively do the same thing for the smaller sub-matrix left inside that ring.

### Time & space complexity
- **Time: O(m · n)** - every element processed once, across all recursive calls.
- **Space: O(min(m,n))** for the recursion depth (number of rings), plus O(m·n) for the output.

*(A valid, correct approach, but the iterative version below is more commonly used in practice and avoids recursion overhead.)*

---

## Approach 2: Optimal - Iterative Shrinking Boundaries

### Intuition
Track four boundaries: `top`, `bottom`, `left`, `right`, representing the current unprocessed rectangular region. Repeatedly walk along each of the four sides of this region (top row left-to-right, right column top-to-bottom, bottom row right-to-left, left column bottom-to-top), shrinking the corresponding boundary inward after each side is processed, until the boundaries cross (nothing left to process).

### Algorithm
1. Initialize `top=0`, `bottom=rows-1`, `left=0`, `right=cols-1`.
2. While `top <= bottom` and `left <= right`:
   - Walk the **top row** from `left` to `right`; then `top += 1`.
   - Walk the **right column** from `top` to `bottom`; then `right -= 1`.
   - **If `top <= bottom` still** (there's still a distinct row left, i.e. we haven't already fully consumed this region into a single row that was just processed as "top"): walk the **bottom row** from `right` to `left`; then `bottom -= 1`.
   - **If `left <= right` still** (similarly, a distinct column remains): walk the **left column** from `bottom` to `top`; then `left += 1`.
3. Return the collected result.

### Python code
```python
def spiralOrder(matrix):
    result = []
    top, bottom = 0, len(matrix) - 1
    left, right = 0, len(matrix[0]) - 1

    while top <= bottom and left <= right:
        for c in range(left, right + 1):
            result.append(matrix[top][c])
        top += 1

        for r in range(top, bottom + 1):
            result.append(matrix[r][right])
        right -= 1

        if top <= bottom:
            for c in range(right, left - 1, -1):
                result.append(matrix[bottom][c])
            bottom -= 1

        if left <= right:
            for r in range(bottom, top - 1, -1):
                result.append(matrix[r][left])
            left += 1

    return result
```

### Line-by-line explanation
- `top, bottom, left, right` - the four boundaries of the currently-unprocessed rectangular region.
- `while top <= bottom and left <= right:` - keep spiraling as long as a valid (non-empty, non-crossed) region remains.
- **Top row:** `for c in range(left, right + 1): result.append(matrix[top][c])` - walk left to right along the current top row; `top += 1` shrinks the region, excluding this row from future consideration.
- **Right column:** `for r in range(top, bottom + 1): ...` - **note `top` has already been incremented**, so this correctly starts just below the row we just processed, avoiding revisiting the top-right corner; walk top to bottom; `right -= 1` shrinks the region.
- `if top <= bottom:` - **this check is essential**: after processing the top row and right column, it's possible the region has been fully consumed (e.g. a single-row matrix) - without this check, we'd incorrectly re-process the same row as if it were a distinct "bottom" row.
- **Bottom row (only if it still exists as distinct from top):** walk right to left; `bottom -= 1`.
- `if left <= right:` - similarly essential check, preventing double-processing a single remaining column.
- **Left column (only if it still exists as distinct from right):** walk bottom to top; `left += 1`.

### Dry run
`matrix = [[1,2,3],[4,5,6],[7,8,9]]`

`top=0,bottom=2,left=0,right=2`.

**Iteration 1:**
- Top row (`c` from 0 to 2): append `matrix[0][0..2]` = `1,2,3`. `top=1`.
- Right column (`r` from 1 to 2): append `matrix[1][2],matrix[2][2]` = `6,9`. `right=1`.
- `top(1) <= bottom(2)`? Yes → bottom row (`c` from 1 down to 0): append `matrix[2][1],matrix[2][0]` = `8,7`. `bottom=1`.
- `left(0) <= right(1)`? Yes → left column (`r` from 1 down to 1): append `matrix[1][0]` = `4`. `left=1`.

`result = [1,2,3,6,9,8,7,4]` so far. Check loop condition: `top(1)<=bottom(1)` and `left(1)<=right(1)` → both true, continue.

**Iteration 2:**
- Top row (`c` from 1 to 1): append `matrix[1][1]` = `5`. `top=2`.
- Right column (`r` from 2 to 1): `range(2,2)` is empty (since `bottom=1 < 2`) → nothing appended. `right=0`.
- `top(2) <= bottom(1)`? No → skip bottom row.
- `left(1) <= right(0)`? No → skip left column.

`result = [1,2,3,6,9,8,7,4,5]`. Check loop condition: `top(2)<=bottom(1)`? No → loop ends.

Final: `[1,2,3,6,9,8,7,4,5]` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(m · n)** - every element visited exactly once, across all boundary shrinks.
- **Space: O(1)** extra (not counting the output list, which is required and necessarily O(m·n)).

---

## Common mistakes & misconceptions

1. **Omitting the `if top <= bottom` / `if left <= right` guards before processing the bottom row / left column.** As the dry run's second iteration shows, this is the single most common bug in spiral traversal - without these guards, a matrix that collapses to a single remaining row or column mid-spiral gets that row/column visited (and appended to the result) twice.
2. **Processing the right column's range using the *old* (pre-increment) value of `top`.** The right column walk must start from the *already-incremented* `top` (after the top row was just processed) - using the stale value would incorrectly re-visit the top-right corner cell a second time.
3. **Assuming spiral traversal only works on square matrices.** The four-boundary technique handles rectangular `m x n` matrices just as correctly as square ones - the boundary-crossing checks (`top <= bottom`, `left <= right`) are exactly what makes it robust to any aspect ratio, including single-row or single-column matrices.
4. **Getting the direction order wrong** (e.g. going down the left column before going right along the top row) - always match the exact order specified by the problem (typically right → down → left → up, starting from top-left), since a valid-looking but differently-ordered spiral produces a completely different output sequence.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Recursive ring peeling | O(m·n) | O(min(m,n)) recursion + O(m·n) output | Valid, conceptually clean, less commonly used in practice. |
| Iterative shrinking boundaries | O(m·n) | O(1) extra + O(m·n) output | The standard, most commonly taught and used solution. |

**Key takeaway:** the "four shrinking boundaries" technique is the standard tool for any spiral/ring-based matrix traversal - the trickiest part is correctly handling the **edge cases where the region collapses to a single row or column mid-spiral** (the `if top <= bottom` / `if left <= right` guards), which is exactly what prevents double-counting elements on matrices that aren't perfectly square with an odd side length.
