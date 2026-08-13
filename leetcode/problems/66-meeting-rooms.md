# 66. Meeting Rooms

**LeetCode:** [#252 - Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) (Premium - video/article write-ups are free) · **Topic:** [Intervals](../topics/16-intervals.md) · **Difficulty:** Easy

## Problem statement

Given an array of meeting time intervals, determine if a person could attend **all** of them (i.e. return `true` if **no two intervals overlap**).

**Example:**
```
Input: intervals = [[0,30],[5,10],[15,20]]
Output: false  ([0,30] overlaps with both [5,10] and [15,20])
```

## Applicable approaches

- **Brute Force - Check every pair of intervals for overlap.**
- **Optimal - Sort by Start Time, Check Adjacent Pairs.** The standard, expected solution.

## Approach 1: Brute Force - Check Every Pair

### Intuition
Directly check every pair of intervals to see if any two overlap.

### Python code
```python
def canAttendMeetings(intervals):
    n = len(intervals)
    for i in range(n):
        for j in range(i + 1, n):
            a, b = intervals[i], intervals[j]
            if a[0] < b[1] and b[0] < a[1]:  # overlap check
                return False
    return True
```

### Time & space complexity
- **Time: O(n²)** - checks every pair.
- **Space: O(1)**.

---

## Approach 2: Optimal - Sort by Start Time, Check Adjacent Pairs

### Intuition
Same core idea as Merge Intervals: once sorted by start time, you only ever need to compare each interval to the **one right before it** - if a meeting starts before the previous one (in sorted order) has ended, there's a conflict. No need to check every pair; adjacent comparisons in sorted order are enough.

### Algorithm
1. Sort intervals by start time.
2. For each interval starting from the second one, check if its start is strictly before the *previous* interval's end - if so, they overlap, return `False`.
3. If no such overlap is found across the whole scan, return `True`.

### Python code
```python
def canAttendMeetings(intervals):
    intervals.sort(key=lambda iv: iv[0])

    for i in range(1, len(intervals)):
        if intervals[i][0] < intervals[i - 1][1]:
            return False

    return True
```

### Line-by-line explanation
- `intervals.sort(key=lambda iv: iv[0])` - sort by start time.
- `for i in range(1, len(intervals)):` - compare each interval to the one immediately before it in sorted order.
- `if intervals[i][0] < intervals[i - 1][1]:` - if the current meeting starts before the previous one ends, they overlap - a person can't attend both.
- `return False` - conflict found, immediately fail.
- `return True` - if every adjacent pair in sorted order is conflict-free, then (crucially, because of the sorted order) **every** pair overall is conflict-free too - no need to check non-adjacent pairs separately, since if `intervals[i-1]` and `intervals[i]` don't overlap, and starts only increase from there, nothing later can overlap with `intervals[i-1]` either (its end has already been "passed" by `intervals[i]`'s start, and everything after `intervals[i]` starts even later still).

### Dry run
`intervals = [[0,30],[5,10],[15,20]]` → sorted by start: `[[0,30],[5,10],[15,20]]` (already in this order).

- `i=1`: `intervals[1][0]=5 < intervals[0][1]=30`? **Yes** → return `False` immediately ✅ (correctly detects the overlap between `[0,30]` and `[5,10]`, without even needing to check `[15,20]`).

### Time & space complexity
- **Time: O(n log n)** - dominated by the sort.
- **Space: O(1)** extra (not counting the sort's own space).

---

## Common mistakes & misconceptions

1. **Believing you still need to check every pair even after sorting.** As the line-by-line explanation proves, once sorted by start time, checking only adjacent pairs is provably sufficient to catch *any* overlap anywhere in the collection - this isn't an approximation or a "usually works" heuristic, it's a direct consequence of the sort order.
2. **Using `<=` instead of `<` in the overlap check** (or vice versa) without checking the problem's exact definition - here, `intervals[i][0] < intervals[i-1][1]` means two meetings that exactly touch (one starts exactly when the other ends) are **not** considered a conflict, matching typical real-world meeting-room semantics (a 9-10am meeting and a 10-11am meeting can share the same room).
3. **Forgetting to sort at all**, and instead trying to reason about adjacency directly on the original (possibly unordered) input - the adjacent-pairs-suffice argument depends entirely on the sort being done first.
4. **Confusing this problem (a yes/no check) with Meeting Rooms II (a "how many rooms" count).** They sound similar and share the same input format, but require genuinely different techniques - this problem needs only a sort and adjacent comparison, while Meeting Rooms II needs to track simultaneously-active meetings via a heap or sweep line.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force (check every pair) | O(n²) | O(1) | Correct, but unnecessary once sorted. |
| Sort by start, check adjacent pairs | O(n log n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** this problem is a simple, clean illustration of the general "sort by start time, then only compare adjacent elements" principle that underlies nearly this entire topic - once sorted, adjacent comparisons are provably sufficient to catch *any* overlap anywhere in the whole collection, avoiding the need to check every pair explicitly.
