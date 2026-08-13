# 70. Set Matrix Zeroes

**LeetCode:** [#73 - Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/) · **Topic:** [Math & Geometry](../topics/17-math-geometry.md) · **Difficulty:** Medium

## Problem statement

Given an `m x n` matrix, if an element is `0`, set its **entire row and entire column** to `0` - and do it **in-place**, ideally using O(1) extra space.

**Example:**
```
Input:  [[1,1,1],       Output: [[1,0,1],
         [1,0,1],                [0,0,0],
         [1,1,1]]                [1,0,1]]
```

## Applicable approaches

- **Brute Force - Copy of the Original Matrix.** Uses O(m·n) space.
- **Better - Marker Sets for Rows/Columns.** Uses O(m+n) space.
- **Optimal - Use the Matrix's Own First Row/Column as Markers.** O(1) extra space.

## Approach 1: Brute Force - Copy the Original Matrix

### Intuition
Make a full copy of the original matrix first (so we always know the *original* values, unaffected by any zeroing we do). Then, scan the *copy* for zeros, and for each one found, zero out the corresponding row/column in the *actual* matrix.

### Time & space complexity
- **Time: O(m · n)**, **Space: O(m · n)** for the copy.

*(Correct, but doesn't meet the O(1) extra space goal.)*

---

## Approach 2: Better - Marker Sets

### Intuition
Instead of copying the whole matrix, just remember **which rows and which columns** need to end up zeroed, using two sets (or boolean arrays). First scan the whole matrix to populate these sets; then do a second pass, zeroing any cell whose row or column is marked.

### Python code
```python
def setZeroes(matrix):
    rows_to_zero = set()
    cols_to_zero = set()

    for r in range(len(matrix)):
        for c in range(len(matrix[0])):
            if matrix[r][c] == 0:
                rows_to_zero.add(r)
                cols_to_zero.add(c)

    for r in range(len(matrix)):
        for c in range(len(matrix[0])):
            if r in rows_to_zero or c in cols_to_zero:
                matrix[r][c] = 0
```

### Time & space complexity
- **Time: O(m · n)**, **Space: O(m + n)** for the two marker sets - better, but not O(1).

---

## Approach 3: Optimal - Use the Matrix's Own First Row/Column as Markers

### Intuition
Instead of allocating separate marker structures, **reuse the matrix's own first row and first column as the marker storage**. If `matrix[r][c] == 0` (for `r > 0` and `c > 0`), mark that fact by setting `matrix[r][0] = 0` and `matrix[0][c] = 0` - "recording" the needed zeroing directly within the existing structure, no extra space needed. The only complication: the first row and first column themselves might *originally* contain a real zero that needs to trigger their own row/column being zeroed too - so we need **two separate boolean flags** (not reused matrix cells) to track whether the first row and first column themselves need to end up entirely zero.

### Algorithm
1. Check up front: does the first row contain any zero? Does the first column contain any zero? (Store as two flags.)
2. For every other cell `(r, c)` with `r > 0` and `c > 0`: if it's zero, mark `matrix[r][0] = 0` and `matrix[0][c] = 0`.
3. Using these markers (now stored in the first row/column), zero out the corresponding cells for the rest of the matrix (again, `r > 0, c > 0` only, to avoid disturbing the markers themselves prematurely).
4. Finally, using the two flags from step 1, zero out the first row and/or first column entirely, if needed.

### Python code
```python
def setZeroes(matrix):
    rows, cols = len(matrix), len(matrix[0])

    first_row_has_zero = any(matrix[0][c] == 0 for c in range(cols))
    first_col_has_zero = any(matrix[r][0] == 0 for r in range(rows))

    # use first row/column as markers for the rest of the matrix
    for r in range(1, rows):
        for c in range(1, cols):
            if matrix[r][c] == 0:
                matrix[r][0] = 0
                matrix[0][c] = 0

    # zero out cells based on the markers (excluding row 0 / col 0 themselves for now)
    for r in range(1, rows):
        for c in range(1, cols):
            if matrix[r][0] == 0 or matrix[0][c] == 0:
                matrix[r][c] = 0

    # finally, handle the first row and first column themselves
    if first_row_has_zero:
        for c in range(cols):
            matrix[0][c] = 0

    if first_col_has_zero:
        for r in range(rows):
            matrix[r][0] = 0
```

### Line-by-line explanation
- `first_row_has_zero`, `first_col_has_zero` - captured **before** any modifications, since the first row/column are about to be repurposed as marker storage, and we need to remember their *original* zero-status separately.
- First main loop (`r,c` starting from 1): for any zero found in the "interior" (excluding row 0 and column 0), record it by zeroing the corresponding marker positions in row 0 and column 0 - **this doesn't touch the interior cell itself**, just plants a marker.
- Second main loop: now that markers are fully planted, zero out any interior cell whose row-marker (`matrix[r][0]`) or column-marker (`matrix[0][c]`) is 0 - **this must happen as a genuinely separate pass** after all markers are planted, since planting a marker for one cell shouldn't be confused with reading a marker for a different cell mid-way through the same pass.
- Finally, using the two flags saved at the very start (not derived from the now-modified row 0/column 0), zero out the first row and/or column entirely if they originally contained a zero.

### Dry run
`matrix = [[1,1,1],[1,0,1],[1,1,1]]`

`first_row_has_zero`: row 0 is `[1,1,1]`, no zero → `False`. `first_col_has_zero`: column 0 is `[1,1,1]` (values `matrix[0][0]=1, matrix[1][0]=1, matrix[2][0]=1`), no zero → `False`.

**Marker-planting pass** (`r,c` from 1): `matrix[1][1]=0` → mark `matrix[1][0]=0` and `matrix[0][1]=0`. Other interior cells (`matrix[1][2]=1`, `matrix[2][1]=1`, `matrix[2][2]=1`) are non-zero, no marking.

Matrix now: `[[1,0,1],[0,0,1],[1,1,1]]` (markers planted at `[0][1]` and `[1][0]`).

**Zero-out pass** (`r,c` from 1): `matrix[1][1]`: check `matrix[1][0]==0`? Yes → zero it (already 0, no visible change). `matrix[1][2]`: check `matrix[1][0]==0`(yes) or `matrix[0][2]==0`(no) → row marker is 0 → zero it: `matrix[1][2]=0`. `matrix[2][1]`: check `matrix[2][0]==0`(no) or `matrix[0][1]==0`(yes) → column marker is 0 → zero it: `matrix[2][1]=0`. `matrix[2][2]`: check `matrix[2][0]==0`(no) or `matrix[0][2]==0`(no) → neither → stays 1.

Matrix now: `[[1,0,1],[0,0,0],[1,0,1]]`.

**Final row/column handling:** `first_row_has_zero=False` → don't touch row 0. `first_col_has_zero=False` → don't touch column 0.

Final matrix: `[[1,0,1],[0,0,0],[1,0,1]]` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(m · n)** - a small constant number of full passes over the matrix.
- **Space: O(1)** extra - only two boolean flags, no separate arrays/sets - the true in-place, optimal solution.

---

## Common mistakes & misconceptions

1. **Zeroing cells during the very first scan, instead of first collecting all the zero-positions/markers and only then zeroing.** If you zero a cell as soon as you find one, you corrupt the original matrix values that later checks in the *same* scan depend on - a cell that was originally non-zero might get zeroed because an earlier cell in the same row/column was zero, and then that *newly-zeroed* cell would incorrectly look like "a zero was originally here," cascading incorrect zeroing across the matrix. This is exactly the "modifying the matrix while relying on its original values elsewhere" trap the topic overview calls out.
2. **Capturing `first_row_has_zero` / `first_col_has_zero` too late** - specifically, capturing them *after* the marker-planting loop has already potentially written zeros into row 0 / column 0. These flags must be captured **before any modification whatsoever**, since the marker-planting step deliberately overwrites `matrix[0][c]` and `matrix[r][0]`, which would make a later check of "did row/column 0 originally have a zero" unreliable.
3. **Running the "zero out based on markers" pass over the *entire* matrix (including row 0 and column 0) instead of restricting it to `r > 0, c > 0`.** Since row 0 and column 0 themselves are being used as marker storage, they must be left untouched until the very final step (handled separately via the two flags) - zeroing them prematurely during the general pass would destroy markers that later cells still need to read.
4. **Believing the two flags could be replaced by just checking `matrix[0][0]`.** A single corner cell can't distinguish "row 0 has a zero" from "column 0 has a zero" from "both do" - two independent flags are genuinely necessary, not a redundant precaution.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Copy the matrix | O(m·n) | O(m·n) | Simple, but far from the O(1) space goal. |
| Marker sets | O(m·n) | O(m+n) | Better, still not O(1). |
| First row/column as markers | O(m·n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** when a problem asks for O(1) extra space but seems to need "remember which rows/columns are special," check whether **part of the existing input structure itself can double as marker storage** - here, the first row and column, combined with two separate flags to protect their own original state, achieve genuine O(1) space where a naive approach would use O(m+n) or more.
