# Topic 17: Math & Geometry

## Core concepts / data structures

This topic doesn't share one single data structure or algorithm the way earlier topics do - instead, it's a collection of problems that revolve around **2-D grid/matrix manipulation using careful indexing and geometric reasoning**, rather than a specific named technique. The core skill is being comfortable translating a spatial/geometric idea ("rotate this," "walk in a spiral," "mark this row and column") into precise index arithmetic.

### Working with a matrix (2-D array) in Python
```python
matrix = [[1,2,3],
          [4,5,6],
          [7,8,9]]
rows, cols = len(matrix), len(matrix[0])
```
- `matrix[r][c]` accesses row `r`, column `c`.
- **Transpose** (flip rows and columns): `matrix[c][r]` for every `(r, c)` swaps the element's position across the main diagonal.
- **Reverse a row**: `row[::-1]` (Python slice notation for "reversed").

### Rotating a matrix 90 degrees - a classic combination trick
Rotating a matrix 90° clockwise **in-place** is commonly done as **two simpler steps combined**: first **transpose** the matrix (swap `matrix[r][c]` with `matrix[c][r]` for all `r < c`), then **reverse each row**. This two-step combination is worth memorizing as a named trick, since deriving the single-step index formula for rotation directly is much more error-prone than composing two simpler, well-understood operations.

### Traversing a matrix in a spiral
Track **four boundaries**: `top`, `bottom`, `left`, `right`, representing the current "unvisited" rectangular region. Walk along the top row (left to right), then the right column (top to bottom), then the bottom row (right to left, if it still exists), then the left column (bottom to top, if it still exists) - shrinking the boundary inward by one after each side, and stopping once the boundaries cross.

### Using the matrix itself as marker storage (in-place signaling)
A common space-optimization trick (seen in Set Matrix Zeroes): instead of allocating separate marker arrays/sets to remember "which rows/columns need special treatment," use the **first row and first column of the matrix itself** as the marker storage - since we can determine what needs marking, mark it directly within the existing structure, and only need a couple of extra flags for edge cases (like "does the first row/column itself need to end up zeroed").

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Transpose + reverse rows** | Rotating a matrix 90° in-place. |
| **Four shrinking boundaries** | Spiral traversal of a matrix. |
| **Layer-by-layer processing** | Similar to spiral traversal, but processing matrix "rings" from outside in, useful for various in-place transformations. |
| **Use existing structure as marker storage** | Avoiding extra space by encoding "needs processing" flags directly into unused/reusable parts of the input itself (e.g. the first row/column). |
| **Careful boundary/off-by-one checking** | Nearly every problem in this topic lives or dies on correctly handling edge/corner cases in index arithmetic - more so than in most other topics. |

## Key terminology

- **In-place** - modifying the given matrix directly, without allocating a full second matrix (O(1) extra space, beyond a few marker variables).
- **Transpose** - flipping a matrix across its main diagonal (`matrix[r][c]` and `matrix[c][r]` swap).
- **Clockwise vs. counter-clockwise rotation** - determines whether you reverse rows or columns (and in which order relative to the transpose) - always double-check the specific direction a problem asks for.
- **Boundary shrinking** - the general technique of tracking `top/bottom/left/right` (or similar) limits that move inward as regions of a grid are fully processed.

## Common beginner mistakes

1. **Getting clockwise vs. counter-clockwise rotation backward** - transpose-then-reverse-rows gives clockwise; transpose-then-reverse-columns (or reverse-rows-then-transpose) gives counter-clockwise - easy to mix these up without testing on a small example first.
2. **Off-by-one errors in spiral traversal's boundary updates** - forgetting to check `if top <= bottom` (or similar) before processing the "bottom row" and "left column" steps, which can cause a single row/column to be visited twice, or cause an error on non-square matrices.
3. **Modifying the matrix while relying on its original values elsewhere in the same computation** - a classic trap in "mark cells, then use those marks" problems (like Set Matrix Zeroes) - overwriting a value too early can corrupt information needed later in the same pass, unless carefully ordered (e.g. processing the first row/column marker cells last, or using a specific sentinel that can't be confused with a real value).
4. **Not handling non-square matrices correctly** - some geometric operations (like simple 90° rotation) technically only make sense for square matrices in their simplest in-place form; watch for whether a specific problem's constraints guarantee square input or need to handle rectangular input differently.

## How this compares to other topics

Math & Geometry problems don't introduce a fundamentally new algorithmic paradigm the way Trees, Graphs, or DP do - they're primarily an exercise in **precise, careful index manipulation** applied to 2-D structures, often combining several small, individually-simple transformations (like transpose + reverse) into a solution for a more complex-looking spatial operation. The main transferable skill is a general comfort with grid indexing that also benefits Graph problems represented as grids (like Number of Islands) and 2-D DP problems.

## Starter problems

Given this topic (in your Blind 75 list) has only 3 problems and they're all fairly self-contained, the most useful "warm-up" is simply working through **Rotate Image** first (the cleanest, most self-contained trick: transpose + reverse), since its "compose two simple operations" mindset directly primes you for the boundary-tracking style of Spiral Matrix and the marker-reuse trick in Set Matrix Zeroes.

## What carries over from here

The careful index-arithmetic discipline built in this topic directly reinforces grid-based Graph problems (Number of Islands, Pacific Atlantic Water Flow) and 2-D DP (Unique Paths) from earlier topics - there's no single new algorithm to carry forward from here, but the general comfort with "reasoning precisely about rows, columns, and boundaries" is broadly useful throughout any problem involving a 2-D structure.
