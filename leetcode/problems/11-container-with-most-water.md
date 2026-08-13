# 11. Container With Most Water

**LeetCode:** [#11 - Container With Most Water](https://leetcode.com/problems/container-with-most-water/) · **Topic:** [Two Pointers](../topics/02-two-pointers.md) · **Difficulty:** Medium

## Problem statement

Given an array `height` where `height[i]` is the height of a vertical line at position `i`, find two lines that, together with the x-axis, form a container that holds the **most water**. Return the maximum amount of water it can hold.

The area between two lines at indices `i` and `j` is: `min(height[i], height[j]) * (j - i)` (width times the shorter of the two heights — water can't rise above the shorter wall, since it would spill over).

**Example:**
```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49
Explanation: lines at index 1 (height 8) and index 8 (height 7): min(8,7) * (8-1) = 7 * 7 = 49
```

## Applicable approaches

- **Brute Force** — check every pair of lines.
- **Optimal — Two Pointers** — the standard, expected O(n) solution.

## Approach 1: Brute Force

### Intuition

The most direct interpretation: try every possible pair of lines, compute the area each pair would hold, and keep track of the maximum. This checks every candidate container exhaustively, so it's trivially correct — the entire interesting content of this problem is in *proving* that most of those n² pairs never need to be checked at all, which is what the optimal approach does.

### Algorithm

1. For every pair of indices `i < j`, compute `min(height[i], height[j]) * (j - i)`.
2. Track the maximum area seen.

### Python code
```python
def maxArea(height):
    n = len(height)
    best = 0
    for i in range(n):
        for j in range(i + 1, n):
            area = min(height[i], height[j]) * (j - i)
            best = max(best, area)
    return best
```

### Line-by-line explanation

- Nested loops generate every pair `(i, j)` with `i < j`.
- `min(height[i], height[j])` — water can only rise as high as the *shorter* of the two walls (it would spill over the shorter one otherwise) — this directly encodes the physical constraint described in the problem statement.
- `* (j - i)` — the width of the container between the two lines.
- `best = max(best, area)` — track the largest area found.

### Time & space complexity

- **Time: O(n²)** — every pair checked, no pair skipped.
- **Space: O(1)**.

---

## Approach 2: Optimal — Two Pointers

### Intuition

Start with the **widest possible container** — the two lines at the very ends of the array — and shrink inward, using a rule that provably never discards the true optimal answer.

**The key insight, worked out rigorously (this is the part worth understanding, not memorizing):** suppose the left pointer's height is *shorter* than the right pointer's. The current container's area is `height[left] * (right - left)` — limited by the *shorter* wall, `height[left]`. Now consider every other container you could form using this same `left` line paired with some line strictly *between* the current `left` and `right`. Any such container has:
- **Width** ≤ `right - left` (it's narrower or equal, since its right edge is now somewhere closer in).
- **Height** ≤ `height[left]` (still capped by the shorter of its two walls — and `height[left]` is still one of those two walls, so the cap can't exceed it).

Since *both* dimensions of any such alternative container (using the current `right` position or anything before it) are bounded by the current container's dimensions, **no container that keeps `right` fixed and moves `left` rightward past its current position, while pairing with something ≤ the current `right`'s position, can ever beat the container we're looking at right now.** That means it's safe to permanently discard the current `right` line from ever being paired with anything to its left of the current `left` position — which is exactly what happens if we now move `left` forward and leave `right` untouched. Symmetric reasoning applies when `height[right]` is the shorter wall: move `right` inward instead.

This is a genuine proof of safety, not a heuristic — it's the same style of argument the topic overview insists every Two Pointers move needs, just phrased for area-maximization instead of sorted-array pair sums.

### Algorithm

1. Set `left = 0`, `right = n - 1` (widest possible container to start).
2. Loop while `left < right`:
   - Compute the current area: `min(height[left], height[right]) * (right - left)`.
   - Track the maximum area seen.
   - Move the pointer at the **shorter** line inward (if `height[left] < height[right]`, move `left += 1`; otherwise move `right -= 1`).
3. Return the maximum area found.

### Python code
```python
def maxArea(height):
    left, right = 0, len(height) - 1
    best = 0

    while left < right:
        width = right - left
        current_height = min(height[left], height[right])
        area = width * current_height
        best = max(best, area)

        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return best
```

### Line-by-line explanation

- `left, right = 0, len(height) - 1` — start at the widest possible container, since width is at its maximum here and can only ever shrink as the pointers move inward — this guarantees we've considered the widest option before narrowing.
- `width = right - left` — current container's width.
- `current_height = min(height[left], height[right])` — water level is limited by the shorter of the two walls.
- `area = width * current_height` — area of the current container.
- `best = max(best, area)` — keep the best area found so far.
- `if height[left] < height[right]: left += 1` — the left wall is the limiting (shorter) one, so move it inward — per the proof above, this is the *only* pointer move that has any chance of finding a taller limiting wall for a future (necessarily narrower) container; moving `right` instead would be provably useless here.
- `else: right -= 1` — otherwise (right is shorter or equal), move `right` inward instead, by the mirrored argument.

### Dry run

`height = [1,8,6,2,5,4,8,3,7]`

| left | right | height[left] | height[right] | width | min height | area | best | move |
|---|---|---|---|---|---|---|---|---|
| 0 | 8 | 1 | 7 | 8 | 1 | 8 | 8 | left shorter → left+=1 |
| 1 | 8 | 8 | 7 | 7 | 7 | 49 | 49 | right shorter → right-=1 |
| 1 | 7 | 8 | 3 | 6 | 3 | 18 | 49 | right shorter → right-=1 |
| 1 | 6 | 8 | 8 | 5 | 8 | 40 | 49 | equal → right-=1 (else branch) |
| 1 | 5 | 8 | 4 | 4 | 4 | 16 | 49 | right shorter → right-=1 |
| 1 | 4 | 8 | 5 | 3 | 5 | 15 | 49 | right shorter → right-=1 |
| 1 | 3 | 8 | 2 | 2 | 2 | 4 | 49 | right shorter → right-=1 |
| 1 | 2 | 8 | 6 | 1 | 6 | 6 | 49 | right shorter → right-=1 |

Now `right = 1 = left`, loop ends. Final answer: `49` ✅ (found at step 2, and correctly never beaten afterward — worth noticing that once the true maximum was found early, the algorithm still had to continue narrowing, since it can't know in advance that 49 is the final answer; it only knows, at each step, which moves are safe to make, not which moves will find the optimum fastest).

### Time & space complexity

- **Time: O(n)** — each pointer moves inward at most n times total across the whole loop, so the total number of iterations is at most n. This is a direct consequence of the monotonic-movement property from the topic overview: `left` only ever increases, `right` only ever decreases, and together they cover at most n total steps before meeting.
- **Space: O(1)** — just two pointers and a running best.

---

## Common mistakes & misconceptions

1. **Moving the taller pointer instead of the shorter one.** This is the single most common error, and it silently produces a wrong (too-low) answer rather than crashing — moving the taller wall inward can only ever shrink width while the height cap stays bounded by the (unmoved) shorter wall, so it can never discover a better answer, and worse, it discards the *actually promising* direction (the shorter wall might have a taller line waiting further in).
2. **Trying to move both pointers on every step "to converge faster."** Unlike the "found a match, move both" case in 3Sum, there's no analogous justification here — moving both pointers when only one move is proven safe risks skipping over the true optimum, which might require the untouched pointer to stay in place while the other one searches for a taller wall.
3. **Believing this problem needs to check every pair to be "safe."** The brute force *is* safe by exhaustion, but the proof above shows a huge fraction of those pairs are provably dominated by pairs already considered — this problem is a strong example of a case where "checking everything" and "checking the necessary things" differ by a full order of complexity (O(n²) vs O(n)), and the gap is closed by a genuine proof, not a guess.
4. **Confusing this with the DP-topic "Trapping Rain Water" problem** (not in this specific Blind 75 list, but a very common companion problem elsewhere) — that problem computes *total* trapped water across the whole terrain profile using every wall simultaneously, a fundamentally different computation from "pick exactly two walls to maximize one container." Don't let superficial water/container-themed similarity suggest the same technique transfers directly.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Checks every pair; correct but slow. |
| Two Pointers | O(n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** when a brute-force solution checks every pair but you can *prove* that certain pairs can be safely eliminated without ever checking them — here, any pair using the current shorter wall alongside anything narrower than the current opposite pointer — Two Pointers turns O(n²) into O(n) by systematically discarding whole groups of pairs at once, one justified pointer move at a time. The value of this problem is specifically in being able to reconstruct that proof, not just recall the "move the shorter pointer" rule.
