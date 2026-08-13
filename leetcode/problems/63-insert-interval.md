# 63. Insert Interval

**LeetCode:** [#57 - Insert Interval](https://leetcode.com/problems/insert-interval/) · **Topic:** [Intervals](../topics/16-intervals.md) · **Difficulty:** Medium

## Problem statement

Given a list of **non-overlapping** intervals `intervals`, sorted by start time, and a `newInterval`, insert `newInterval` into the list (merging if necessary) so the result is still sorted and non-overlapping.

**Example:**
```
Input: intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
```

## Applicable approaches

- **Linear Search + Insert, then Merge Pass** - a two-step, straightforward approach.
- **Optimal - Single Pass (three phases: before, overlapping, after).**

## Approach 1: Two-Step (Insert, Then Merge Everything)

### Intuition
Since the input isn't already sorted *with* the new interval included, one simple approach: append `newInterval` to the list, sort everything by start time, then run a standard "merge overlapping intervals" pass (see the next problem, Merge Intervals) over the whole thing.

### Python code
```python
def insert(intervals, newInterval):
    intervals = intervals + [newInterval]
    intervals.sort(key=lambda iv: iv[0])

    merged = []
    for interval in intervals:
        if merged and interval[0] <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], interval[1])
        else:
            merged.append(interval)

    return merged
```

### Time & space complexity
- **Time: O(n log n)** - dominated by the sort.
- **Space: O(n)**.

*(Correct, but doesn't take advantage of the fact that `intervals` was *already* sorted before inserting - the optimal approach below exploits that to avoid re-sorting entirely.)*

---

## Approach 2: Optimal - Single Pass (Three Phases)

### Intuition
Since `intervals` is already sorted and non-overlapping, we can process it in a **single linear pass**, thinking of it in three phases: (1) intervals that end **before** `newInterval` starts (no overlap possible - keep them as-is), (2) intervals that **overlap** `newInterval` (merge them all into one combined interval), (3) intervals that start **after** `newInterval` ends (no overlap possible - keep them as-is). No sorting needed at all, since the input's existing order is preserved throughout.

### Algorithm
1. **Phase 1:** while the current interval ends strictly before `newInterval` starts, add it directly to the result and move on.
2. **Phase 2:** while the current interval overlaps `newInterval` (starts at or before `newInterval`'s end), merge it into `newInterval` by expanding `newInterval`'s bounds (`min` of starts, `max` of ends), and move on. After this phase, add the now-fully-merged `newInterval` to the result.
3. **Phase 3:** add all remaining intervals directly to the result (they're guaranteed to start after the merged interval ends).

### Python code
```python
def insert(intervals, newInterval):
    result = []
    i = 0
    n = len(intervals)

    # phase 1: intervals ending before newInterval starts
    while i < n and intervals[i][1] < newInterval[0]:
        result.append(intervals[i])
        i += 1

    # phase 2: intervals overlapping newInterval - merge them all in
    while i < n and intervals[i][0] <= newInterval[1]:
        newInterval = [min(newInterval[0], intervals[i][0]), max(newInterval[1], intervals[i][1])]
        i += 1
    result.append(newInterval)

    # phase 3: remaining intervals start after newInterval ends
    while i < n:
        result.append(intervals[i])
        i += 1

    return result
```

### Line-by-line explanation
- `while i < n and intervals[i][1] < newInterval[0]:` - **phase 1 condition**: this existing interval ends *before* `newInterval` even begins, so there's definitely no overlap - safe to add directly, unchanged.
- `while i < n and intervals[i][0] <= newInterval[1]:` - **phase 2 condition**: this existing interval starts *at or before* the current (possibly already-expanded) end of `newInterval` - meaning it overlaps (or touches) `newInterval` - merge it in.
- `newInterval = [min(...), max(...)]` - expand `newInterval` to cover both ranges - note that `newInterval` itself keeps growing across multiple iterations of this loop if several consecutive intervals all overlap it, which is exactly why we check against the *current* (possibly already-updated) `newInterval[1]`, not the original.
- `result.append(newInterval)` - once phase 2's loop ends (no more overlapping intervals), the fully-merged `newInterval` is finalized and added.
- **Phase 3:** any remaining intervals are guaranteed (by the sorted, non-overlapping nature of the input, and the fact that phase 2 just ended) to start strictly after the merged interval - add them unchanged.

### Dry run
`intervals = [[1,3],[6,9]]`, `newInterval = [2,5]`

**Phase 1:** `i=0`: `intervals[0][1]=3 < newInterval[0]=2`? No (`3 < 2` is false) → phase 1 loop doesn't execute at all. `result = []`, `i` stays 0.

**Phase 2:** `i=0`: `intervals[0][0]=1 <= newInterval[1]=5`? Yes → merge: `newInterval = [min(2,1), max(5,3)] = [1,5]`. `i=1`. Check again: `intervals[1][0]=6 <= newInterval[1]=5`? No (`6<=5` false) → phase 2 loop ends. `result.append([1,5])` → `result=[[1,5]]`.

**Phase 3:** `i=1`: add `intervals[1]=[6,9]` → `result=[[1,5],[6,9]]`. `i=2=n`, loop ends.

Final: `[[1,5],[6,9]]` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(n)** - a single linear pass, no sorting needed (the input was already sorted).
- **Space: O(n)** for the result list.

---

## Common mistakes & misconceptions

1. **Using `<` instead of `<=` (or vice versa) in the phase boundary conditions**, especially around whether touching endpoints count as overlapping - per the topic overview, this varies by problem statement, and Insert Interval's phase 1/phase 2 boundary (`intervals[i][1] < newInterval[0]`) specifically treats touching intervals (e.g. `[1,3]` and `[3,5]`) as overlapping, not separate - getting the strict vs. non-strict comparison backward silently misclassifies boundary cases.
2. **Forgetting that `newInterval` itself can keep growing across multiple iterations of phase 2.** A common bug is checking overlap against the *original* `newInterval` bounds instead of the continuously-updated ones - since phase 2's loop condition and merge step both need to reference the *current* (possibly already-expanded) `newInterval`, not a stale copy from before the loop started.
3. **Not handling the edge case where `newInterval` doesn't overlap anything** (phase 2 never executes) - the code must still correctly append `newInterval` on its own after phase 2 (even with zero iterations), which the code above handles correctly since `result.append(newInterval)` sits unconditionally after the phase 2 loop, not inside it.
4. **Defaulting to the "append, sort, merge" approach without noticing the input's sorted guarantee.** As the key takeaway stresses, this is the single biggest missed optimization opportunity in this specific problem - always check problem constraints/guarantees before reaching for a "safe default" like re-sorting.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Append + re-sort + merge | O(n log n) | O(n) | Simple, but wastefully re-sorts already-sorted data. |
| Single pass (3 phases) | O(n) | O(n) | The standard, expected optimal solution - exploits the guarantee that input is already sorted. |

**Key takeaway:** always check whether a problem's input is **already sorted** before defaulting to "sort it and merge" - here, that guarantee is exactly what allows a single O(n) linear pass instead of an O(n log n) resort, by splitting the work into three clean, sequential phases (before, overlapping, after).
