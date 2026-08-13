# 65. Non-overlapping Intervals

**LeetCode:** [#435 - Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) · **Topic:** [Intervals](../topics/16-intervals.md) · **Difficulty:** Medium

## Problem statement

Given an array of intervals, return the **minimum number of intervals you need to remove** so that the rest are all non-overlapping.

**Example:**
```
Input: intervals = [[1,2],[2,3],[3,4],[1,3]]
Output: 1   (remove [1,3], leaving [1,2],[2,3],[3,4] - all non-overlapping)
```

## Applicable approaches

- **DP (Longest Non-Overlapping Subsequence)** - reframe as "keep the maximum number," an approach related to Longest Increasing Subsequence.
- **Optimal - Greedy, Sort by END Time.** The standard, expected, more efficient solution.

## Approach 1: DP (Reframe as "Keep the Maximum")

### Intuition
"Minimum to remove" is the same as `total count - maximum number you can keep without any overlaps`. Sort by start time, then find the longest possible chain of non-overlapping intervals using a DP approach similar in spirit to Longest Increasing Subsequence: `dp[i]` = the longest non-overlapping chain ending with interval `i`, built by checking all earlier intervals `j` that don't overlap with `i`.

### Time & space complexity
- **Time: O(n²)** - similar structure to the O(n²) Longest Increasing Subsequence approach.
- **Space: O(n)**.

*(Correct, but slower than necessary - the greedy approach below achieves O(n log n).)*

---

## Approach 2: Optimal - Greedy, Sort by End Time

### Intuition
This is a classic **"activity selection"** greedy problem. The key insight: to fit as many non-overlapping intervals as possible, you should always prefer to keep whichever interval **ends earliest** among your current options - because ending earlier leaves the most room for future intervals to also fit without overlapping. This means: **sort by end time** (not start time, unlike most other problems in this topic!), then greedily keep every interval that doesn't overlap with the last interval we decided to keep, discarding (counting as "removed") any that do overlap.

### Algorithm
1. Sort `intervals` by **end time**.
2. Track `last_end = -infinity` (the end time of the most recently *kept* interval) and `removed_count = 0`.
3. For each interval `(start, end)` in sorted order:
   - If `start < last_end`, this interval overlaps with the last kept one - it must be removed: increment `removed_count` (and, importantly, do **NOT** update `last_end`, since we're keeping the earlier-ending interval, which is always at least as good or better for fitting future intervals).
   - Otherwise, no overlap - keep this interval: update `last_end = end`.
4. Return `removed_count`.

### Python code
```python
def eraseOverlapIntervals(intervals):
    intervals.sort(key=lambda iv: iv[1])  # sort by END time

    last_end = float("-inf")
    removed_count = 0

    for start, end in intervals:
        if start < last_end:
            removed_count += 1
        else:
            last_end = end

    return removed_count
```

### Line-by-line explanation
- `intervals.sort(key=lambda iv: iv[1])` - **sort by end time** - this is the crucial, distinguishing choice for this specific problem (contrast with Merge Intervals and Insert Interval, which sort by start time).
- `last_end = float("-inf")` - initially, nothing has been kept yet, so any interval's start will trivially be ≥ this.
- `for start, end in intervals:` - process in sorted (end-time) order.
- `if start < last_end:` - this interval overlaps with the most recently *kept* interval (its start is before that interval's end) - we must remove one of the two to eliminate the overlap, and greedy logic says: **always remove the one with the LATER end time**, which - since we're processing in sorted-by-end order - is guaranteed to be the **current** interval, not the previously kept one (whatever was kept earlier necessarily has an end time ≤ the current interval's end time, since we're iterating in ascending end-time order).
- `removed_count += 1` - count this removal; crucially, we do **not** update `last_end` here - we're keeping the earlier-ending interval as our reference point, since it leaves more room for future intervals.
- `else: last_end = end` - no overlap - keep this interval, updating our reference point to its end time.
- `return removed_count`.

### Why sorting by END time (not start time) is essential here
If we sorted by start time instead, when we find an overlap, it wouldn't be immediately obvious *which* of the two overlapping intervals to discard to maximize future flexibility - we might keep an interval that ends very late, blocking many future intervals unnecessarily. Sorting by end time guarantees that whenever we encounter an overlap, the interval we've already committed to keeping is *always* the one (among the two) that leaves the most room for the future - we never even need to compare their end times explicitly, because the sort order already guarantees this.

### Dry run
`intervals = [[1,2],[2,3],[3,4],[1,3]]` → sorted by end time: `[[1,2],[2,3],[1,3],[3,4]]` (end times 2,3,3,4; for the tie between `[2,3]` and `[1,3]`, their relative order doesn't affect correctness here).

`last_end = -inf`, `removed_count = 0`.

| interval | start < last_end? | action | last_end after | removed_count |
|---|---|---|---|---|
| [1,2] | `1 < -inf`? No | keep | `2` | 0 |
| [2,3] | `2 < 2`? No | keep | `3` | 0 |
| [1,3] | `1 < 3`? **Yes** | remove | (unchanged) `3` | 1 |
| [3,4] | `3 < 3`? No | keep | `4` | 1 |

Final: `removed_count = 1` ✅ matches expected output.

### Time & space complexity
- **Time: O(n log n)** - dominated by the sort; the greedy pass itself is O(n).
- **Space: O(1)** extra (not counting the sort's own space), a clear improvement over the O(n²) DP approach.

---

## Common mistakes & misconceptions

1. **Sorting by start time instead of end time, out of habit from Merge Intervals/Insert Interval.** This is the single most important trap in this problem specifically - as explained above, sorting by start time doesn't give a clean rule for *which* of two overlapping intervals to discard, breaking the greedy argument entirely.
2. **Updating `last_end` when an overlap is found (i.e. discarding the wrong interval).** The greedy correctness relies on **always keeping the earlier-ending interval** - since we process in ascending end-time order, the interval we've already committed to keeping is guaranteed to end no later than the current one, so `last_end` must stay unchanged when an overlap forces a removal.
3. **Confusing "minimum to remove" with "maximum to keep."** They're complementary (`remove = total - keep`), but the code directly computes the removal count - if you're cross-checking against a DP formulation that computes the "keep" count instead, remember to subtract from the total before comparing answers.
4. **Treating touching intervals (`end == next.start`) as non-overlapping when the problem defines them as overlapping, or vice versa.** The condition `start < last_end` (strict inequality) specifically treats exactly-touching intervals as **not** overlapping (allowed to coexist) - double-check this matches the problem's exact overlap definition, since some interval problems use non-strict comparisons instead.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| DP (longest non-overlapping chain) | O(n²) | O(n) | Correct, related to LIS-style DP, but slower than necessary. |
| Greedy, sort by end time | O(n log n) | O(1) extra | The standard, expected optimal solution. |

**Key takeaway:** this is the one problem in the Intervals topic where sorting by **end time** (not start time) is essential - whenever a problem is really "greedily select the maximum number of non-overlapping items," sorting by end time and always preferring to keep the earliest-ending option is the classic, provably-correct "activity selection" greedy strategy, worth remembering as distinct from the "sort by start time" pattern used for merging/insertion problems.
