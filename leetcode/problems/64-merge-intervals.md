# 64. Merge Intervals

**LeetCode:** [#56 - Merge Intervals](https://leetcode.com/problems/merge-intervals/) · **Topic:** [Intervals](../topics/16-intervals.md) · **Difficulty:** Medium

## Problem statement

Given an array of intervals, merge all **overlapping** intervals, and return the resulting array of non-overlapping intervals.

**Example:**
```
Input: intervals = [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

## Applicable approaches

- **Brute Force - Repeatedly scan and merge any overlapping pair, until no more merges are possible.**
- **Optimal - Sort by Start Time, then Single Pass Merge.** The standard, expected solution.

## Approach 1: Brute Force - Repeated Scanning

### Intuition
Repeatedly scan through the list looking for any two intervals that overlap; merge them into one, and repeat the whole scan again from scratch, until an entire pass finds no more overlaps.

### Why this is inefficient
Without sorting first, we might need to compare many pairs of intervals repeatedly across multiple full scans, since a merge early in the list could suddenly create a new overlap with something much later - this is why the approach below always sorts first, which structurally guarantees only *adjacent* comparisons are ever needed.

---

## Approach 2: Optimal - Sort by Start Time, Then Single Pass

### Intuition
If we **sort intervals by start time** first, a hugely convenient property emerges: any interval that's going to overlap with our current "merged so far" interval must be the **very next one** in the sorted order (since everything comes in increasing start-time order, we'd have already encountered and merged in anything with a smaller or equal start time). This means we only ever need to compare each interval to the **most recently finalized** merged interval - one single linear pass, no repeated re-scanning needed.

### Algorithm
1. Sort `intervals` by start time.
2. Initialize `merged = [intervals[0]]` (start with the first interval as our current "in progress" merged interval).
3. For each subsequent interval: if it overlaps with `merged`'s last entry (its start is ≤ the last entry's end), extend the last entry's end to cover both (`max` of the two ends). Otherwise, it doesn't overlap - append it as a brand new entry in `merged`.
4. Return `merged`.

### Python code
```python
def merge(intervals):
    intervals.sort(key=lambda iv: iv[0])
    merged = [intervals[0]]

    for start, end in intervals[1:]:
        if start <= merged[-1][1]:
            merged[-1][1] = max(merged[-1][1], end)
        else:
            merged.append([start, end])

    return merged
```

### Line-by-line explanation
- `intervals.sort(key=lambda iv: iv[0])` - sort by start time - the essential setup step that makes the rest of the algorithm valid.
- `merged = [intervals[0]]` - seed the result with the first (now earliest-starting) interval.
- `for start, end in intervals[1:]:` - process every remaining interval in sorted order.
- `if start <= merged[-1][1]:` - does this interval's start fall at or before the *end* of the most recently finalized merged interval? If so, they overlap (or touch).
- `merged[-1][1] = max(merged[-1][1], end)` - extend the last merged interval's end to cover this one too (using `max`, since the current interval's end might actually be *shorter* than what's already merged, e.g. a small interval fully contained within a larger one - we must never accidentally shrink the merged range).
- `else: merged.append([start, end])` - no overlap with the most recent merged interval - this starts a genuinely new, separate group.

### Dry run
`intervals = [[1,3],[2,6],[8,10],[15,18]]` (already sorted by start time in this example)

`merged = [[1,3]]`.

| interval | start <= merged[-1][1]? | action | merged after |
|---|---|---|---|
| [2,6] | `2 <= 3`? Yes | extend: `merged[-1][1] = max(3,6)=6` | `[[1,6]]` |
| [8,10] | `8 <= 6`? No | append new | `[[1,6],[8,10]]` |
| [15,18] | `15 <= 10`? No | append new | `[[1,6],[8,10],[15,18]]` |

Final: `[[1,6],[8,10],[15,18]]` ✅ matches expected output exactly.

### Time & space complexity
- **Time: O(n log n)** - dominated by the sort; the merge pass itself is O(n).
- **Space: O(n)** for the result (and O(log n) to O(n) for the sort's own internal space, depending on implementation), or O(1) extra if in-place sorting is used and we don't count the output.

---

## Common mistakes & misconceptions

1. **Forgetting to sort at all before the single pass.** Without sorting by start time first, the "only compare to the most recent merged interval" shortcut is simply invalid - an unsorted input could have a later-listed interval that actually overlaps an earlier one seen several steps back, which a single forward pass without sorting would miss entirely.
2. **Using `merged[-1][1] = end` instead of `merged[-1][1] = max(merged[-1][1], end)`.** This is a subtle but real bug: if the current interval is fully *contained within* the already-merged range (e.g. merged is `[1,10]` and the current interval is `[2,3]`), blindly overwriting with `end=3` would incorrectly *shrink* the merged interval, losing coverage up to 10.
3. **Comparing the current interval's start against the *original* (un-merged) interval's end, instead of the possibly-already-extended `merged[-1][1]`.** Since `merged[-1]` may have already grown from earlier merges, comparisons must always reference its *current* (updated) end, not any original input value.
4. **Mutating the input list's inner interval lists in place** (e.g. via `interval[1] = ...`) when the problem or surrounding code expects the original `intervals` input to remain unmodified - worth being deliberate about whether in-place mutation of nested lists is acceptable in a given context.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Repeated brute-force scanning | Potentially O(n²) or worse without sorting | O(n) | Inefficient without a guaranteed processing order. |
| Sort by start, single pass | O(n log n) | O(n) | The standard, expected, essentially-only-reasonable solution. |

**Key takeaway:** this is *the* canonical Intervals-topic problem - sort by start time, then compare each interval only to the most recently finalized merged interval, extending or starting fresh as needed. This exact pattern (sort, then single pass comparing to "the last thing kept") is the direct template for several other interval problems, including Insert Interval's merge phase.
