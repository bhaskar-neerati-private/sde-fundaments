# 68. Rotate Image

**LeetCode:** [#48 - Rotate Image](https://leetcode.com/problems/rotate-image/) · **Topic:** [Math & Geometry](../topics/17-math-geometry.md) · **Difficulty:** Medium

## Problem statement

Given an `n x n` 2D matrix representing an image, rotate the image **90 degrees clockwise**, **in-place** (don't allocate a separate matrix).

**Example:**
```
Input:  [[1,2,3],       Output: [[7,4,1],
         [4,5,6],                [8,5,2],
         [7,8,9]]                [9,6,3]]
```

## Applicable approaches

- **Brute Force - Copy into a New Rotated Matrix.** Simple, but uses O(n²) extra space, violating the in-place requirement.
- **Optimal - Transpose, Then Reverse Each Row.** The standard, expected in-place solution.
- **Optimal (alternative) - Four-Way Element Swap (Layer by Layer).** A more direct, single-pass in-place approach.

## Approach 1: Brute Force - Copy Into a New Matrix

### Intuition
For a 90° clockwise rotation, the element at `matrix[r][c]` moves to position `[c][n-1-r]` in the new matrix. Directly build a new matrix using this formula.

### Python code
```python
def rotate(matrix):
    n = len(matrix)
    rotated = [[0] * n for _ in range(n)]

    for r in range(n):
        for c in range(n):
            rotated[c][n - 1 - r] = matrix[r][c]

    for r in range(n):
        matrix[r] = rotated[r]
```

### Time & space complexity
- **Time: O(n²)**, **Space: O(n²)** - violates the "in-place" requirement, even though the final copy-back makes `matrix` end up correct.

---

## Approach 2: Optimal - Transpose, Then Reverse Each Row

### Intuition
A 90° clockwise rotation can be decomposed into two much simpler, well-understood operations performed in sequence: first **transpose** the matrix (flip it across its main diagonal, swapping `matrix[r][c]` with `matrix[c][r]`), then **reverse each row**. Composing these two simple, individually-easy-to-get-right operations is far less error-prone than trying to derive a single-pass rotation index formula directly.

### Algorithm
1. **Transpose:** for every `r < c`, swap `matrix[r][c]` and `matrix[c][r]` (only need to process the upper triangle, since swapping `(r,c)` with `(c,r)` handles both positions at once - processing the full matrix would swap everything back).
2. **Reverse each row** in place.

### Python code
```python
def rotate(matrix):
    n = len(matrix)

    # step 1: transpose
    for r in range(n):
        for c in range(r + 1, n):
            matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]

    # step 2: reverse each row
    for row in matrix:
        row.reverse()
```

### Line-by-line explanation
- `for r in range(n): for c in range(r + 1, n):` - **note `c` starts at `r + 1`, not 0** - this ensures we only process each pair `(r,c)`/`(c,r)` exactly once (processing the full square would swap every pair twice, undoing the transpose).
- `matrix[r][c], matrix[c][r] = matrix[c][r], matrix[r][c]` - Python's tuple-swap syntax, cleanly exchanging the two positions.
- `for row in matrix: row.reverse()` - reverses each row in place (`list.reverse()` is an in-place O(row length) operation).

### Why transpose + reverse-rows = clockwise rotation
Transposing alone flips the matrix across the main diagonal (top-left to bottom-right) - this is actually a counter-clockwise-ish flip combined with a mirror. Reversing each row afterward completes the transformation into a genuine 90° **clockwise** rotation. (If counter-clockwise rotation were needed instead, you'd transpose and then reverse each **column** instead - or equivalently, reverse the row order first, then transpose - always worth verifying the exact composition against a small example when direction matters.)

### Dry run
`matrix = [[1,2,3],[4,5,6],[7,8,9]]`

**Transpose** (swap `(r,c)` with `(c,r)` for `r<c`): swap `(0,1)` &`(1,0)`: 2↔4. swap `(0,2)`&`(2,0)`: 3↔7. swap `(1,2)`&`(2,1)`: 6↔8.

After transpose:
```
1 4 7
2 5 8
3 6 9
```

**Reverse each row:**
- Row 0: `[1,4,7]` → `[7,4,1]`
- Row 1: `[2,5,8]` → `[8,5,2]`
- Row 2: `[3,6,9]` → `[9,6,3]`

Final:
```
7 4 1
8 5 2
9 6 3
```
✅ matches expected output exactly.

### Time & space complexity
- **Time: O(n²)** - both the transpose and the row-reversal touch every element a constant number of times.
- **Space: O(1)** extra - genuinely in-place, no new matrix allocated.

---

## Approach 3: Optimal (Alternative) - Four-Way Element Swap, Layer by Layer

### Intuition
Directly rotate the matrix "one ring (layer) at a time," working from the outermost layer inward. For each layer, cycle groups of 4 corresponding elements (top, right, bottom, left positions that all map to each other under rotation) directly into their final rotated positions, using one temporary variable per group of 4.

### Python code
```python
def rotate(matrix):
    n = len(matrix)

    for layer in range(n // 2):
        first, last = layer, n - 1 - layer
        for i in range(first, last):
            offset = i - first

            top = matrix[first][i]

            matrix[first][i] = matrix[last - offset][first]
            matrix[last - offset][first] = matrix[last][last - offset]
            matrix[last][last - offset] = matrix[i][last]
            matrix[i][last] = top
```

### Time & space complexity
- **Time: O(n²)**, **Space: O(1)** - also genuinely in-place, no auxiliary matrix.

---

## Common mistakes & misconceptions

1. **Transposing the full matrix instead of only the upper triangle (`c` starting from `r+1`, not 0).** Iterating over every `(r,c)` pair (including `r > c` and `r == c`) would swap every off-diagonal pair *twice*, undoing the transpose entirely and leaving the matrix unchanged (aside from redundant self-swaps on the diagonal).
2. **Reversing columns instead of rows (or reversing before transposing), producing a counter-clockwise rotation instead of clockwise**, or vice versa - always verify the exact composition against a small worked example (as done in the dry run) when rotation direction matters, rather than trusting memory alone.
3. **Believing the "copy into a new matrix" approach is an acceptable solution when the problem explicitly requires in-place, O(1) extra space.** It produces the correct final values but fails the actual constraint - worth being explicit that "correct output" and "meets the stated space constraint" are two separate success criteria here.
4. **In the four-way swap approach, getting the layer bounds or `offset` arithmetic wrong**, especially for the middle layer of an odd-sized matrix (`n // 2` correctly excludes the innermost single-cell "layer," which needs no swapping since it maps to itself under rotation) - worth tracing through a small odd-sized example if this approach is used, since its index arithmetic is considerably more error-prone than transpose+reverse.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Copy into new matrix | O(n²) | O(n²) | Simple, but violates the in-place requirement. |
| Transpose + reverse rows | O(n²) | O(1) | The standard, most commonly taught in-place solution - easy to remember as two simple composed steps. |
| Four-way swap (layer by layer) | O(n²) | O(1) | A more direct single-pass alternative, slightly more index-arithmetic-heavy. |

**Key takeaway:** when a geometric transformation seems complex to derive directly (like a single-formula in-place rotation), check whether it can be **decomposed into two or more simpler, individually well-known operations** (here: transpose + row-reversal) - composing simple, correct pieces is usually far more reliable than deriving one complex formula from scratch.
