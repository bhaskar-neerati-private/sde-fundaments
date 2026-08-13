# 36. Find Median from Data Stream

**LeetCode:** [#295 - Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) · **Topic:** [Heap / Priority Queue](../topics/08-heap-priority-queue.md) · **Difficulty:** Hard

## Problem statement

Design a data structure that supports adding numbers one at a time (`addNum`), and can efficiently return the **median** of all numbers added so far (`findMedian`) at any point.

- If there's an even count of numbers, the median is the average of the two middle values.
- If odd, it's the single middle value.

**Example:**
```
addNum(1), addNum(2) -> findMedian() = 1.5
addNum(3) -> findMedian() = 2
```

## Applicable approaches

- **Brute Force — Keep a sorted list, insert in the right position each time.**
- **Better — Keep an unsorted list, sort only when `findMedian` is called.**
- **Optimal — Two Heaps (max-heap for the smaller half, min-heap for the larger half).** The standard, expected solution.

## Approach 1: Brute Force — Keep a Sorted List

### Intuition

Maintain the numbers in a fully sorted list at all times, using binary search to find where a new number should be inserted, then physically inserting it there. This makes `findMedian` trivially fast (direct index access into an already-sorted list), but it pays for that by making `addNum` expensive — exactly the same array-insertion cost tradeoff described in the Arrays & Hashing topic.

### Python code
```python
import bisect

class MedianFinder:
    def __init__(self):
        self.nums = []

    def addNum(self, num):
        bisect.insort(self.nums, num)  # binary search for position, then insert

    def findMedian(self):
        n = len(self.nums)
        mid = n // 2
        if n % 2 == 1:
            return float(self.nums[mid])
        return (self.nums[mid - 1] + self.nums[mid]) / 2
```

### Time & space complexity

- **Time: `addNum` is O(n).** Even though `bisect.insort` *finds* the insertion point in O(log n) (a binary search, per the Binary Search topic), actually **inserting** into a Python list at an arbitrary position is O(n) — everything after it must physically shift, the same "array insertion in the middle is O(n)" fact from Arrays & Hashing. **`findMedian` is O(1)** (direct index access, since the list is always kept sorted).
- **Space: O(n)**.

---

## Approach 2: Better — Sort Only on `findMedian`

### Intuition

If `addNum` is called far more often than `findMedian`, it might be cheaper to just append (O(1)) on every `addNum`, and only pay the sorting cost when `findMedian` is actually called — deferring the work until it's actually needed, rather than doing it eagerly on every insertion regardless of whether a median query ever follows.

### Python code
```python
class MedianFinder:
    def __init__(self):
        self.nums = []

    def addNum(self, num):
        self.nums.append(num)

    def findMedian(self):
        self.nums.sort()
        n = len(self.nums)
        mid = n // 2
        if n % 2 == 1:
            return float(self.nums[mid])
        return (self.nums[mid - 1] + self.nums[mid]) / 2
```

### Time & space complexity

- **Time: `addNum` is O(1)**, but **`findMedian` is O(n log n)** — it re-sorts from scratch every single call, which is wasteful if `findMedian` is called repeatedly (each call redoes the sorting work of every previous call, since nothing is remembered between calls).
- **Space: O(n)**.

*(Neither brute-force variant is great when both operations are called frequently and interleaved — the optimal approach below achieves O(log n) for `addNum` and O(1) for `findMedian`, genuinely the best of both worlds, not just a different tradeoff point on the same curve.)*

---

## Approach 3: Optimal — Two Heaps

### Intuition

The median sits at the **boundary** between "the smaller half of the numbers" and "the larger half." So, maintain two heaps: a **max-heap** called `small` holding the smaller half of all numbers seen so far (so its root — the *largest* of the small half — is right at the boundary from below), and a **min-heap** called `large` holding the larger half (so its root — the *smallest* of the large half — is right at the boundary from above). Keep both heaps **balanced in size** (equal, or `small` having exactly one more element than `large`). Then the median is always derivable from just the two roots — no full sort ever needed, because we've arranged for the *only* values that ever need comparing at query time to already be sitting at O(1)-accessible positions.

### Algorithm

**`addNum(num)`:**
1. Push `num` onto `small` (the max-heap side) first.
2. To maintain the invariant "everything in `small` ≤ everything in `large`," move `small`'s current max over to `large`: pop it from `small`, push it onto `large`.
3. **Rebalance sizes:** if `large` now has more elements than `small`, move `large`'s min back over to `small`.

(This "push to one side, then shuffle the extreme value across, then rebalance sizes" sequence looks roundabout at first, but it's a standard, robust way to guarantee both the ordering invariant between the heaps *and* the size-balance invariant, without special-casing "should this number go left or right?" directly — instead of deciding upfront, we always route through `small` first and let the shuffle-and-rebalance steps sort out where it actually belongs.)

**`findMedian()`:**
- If `small` has more elements than `large` (odd total count), the median is `small`'s root (the max-heap's top).
- If they're equal size (even total count), the median is the average of both heaps' roots.

### Python code
```python
import heapq

class MedianFinder:
    def __init__(self):
        self.small = []  # max-heap (store negated values), holds the smaller half
        self.large = []  # min-heap, holds the larger half

    def addNum(self, num):
        heapq.heappush(self.small, -num)          # push to max-heap (negated)
        heapq.heappush(self.large, -heapq.heappop(self.small))  # move small's max to large

        if len(self.large) > len(self.small):
            heapq.heappush(self.small, -heapq.heappop(self.large))  # rebalance

    def findMedian(self):
        if len(self.small) > len(self.large):
            return float(-self.small[0])
        return (-self.small[0] + self.large[0]) / 2
```

### Line-by-line explanation

- `self.small = []` — simulates a **max-heap** by storing negated values (the standard Python `heapq` trick from the topic overview, since it only provides min-heap behavior natively).
- `self.large = []` — a regular min-heap.
- `heapq.heappush(self.small, -num)` — add the new number to `small`, negated (so `small`'s "smallest negated value" corresponds to the *largest actual value* currently in `small`).
- `heapq.heappush(self.large, -heapq.heappop(self.small))` — pop `small`'s current maximum (undo the negation with another `-`), and push it into `large`. This step guarantees that after every `addNum` call, **every value in `small` is ≤ every value in `large`** (since we always route the newest number through `small` first, then immediately promote whatever the current largest-of-small is over to `large`, that value is guaranteed to be ≥ everything remaining in `small`, and it becomes the new minimum candidate for `large`).
- `if len(self.large) > len(self.small): heapq.heappush(self.small, -heapq.heappop(self.large))` — if `large` grew too big (more than `small`), move its minimum back to `small` (negating again to store it correctly as a max-heap element) — this restores the size balance, keeping `small`'s size always equal to or exactly one more than `large`'s, which is exactly the invariant `findMedian` relies on.
- `findMedian()`: `if len(self.small) > len(self.large): return float(-self.small[0])` — odd total count, `small` has the extra element, so the true median is exactly `small`'s max (un-negate it).
- `return (-self.small[0] + self.large[0]) / 2` — even total count (equal sizes), median is the average of the two boundary values.

### Dry run

`addNum(1)`: push `-1` to small → `small=[-1]`. Pop small's max: pops `-1` (the only element), negate → `1`, push `1` to large → `large=[1]`. Check sizes: `len(large)=1 > len(small)=0`? Yes → move large's min (`1`) back: pop `1` from large, push `-1` to small → `small=[-1]`, `large=[]`.

`addNum(2)`: push `-2` to small → `small=[-2,-1]` (heap order, `-2` is the min i.e. the actual max=2 is on top). Pop small's max: pops `-2` (smallest negated = largest actual), negate→`2`, push to large → `large=[2]`. `small` now `[-1]`. Check sizes: `len(large)=1 > len(small)=1`? No (equal) → no rebalance.

`findMedian()`: `len(small)=1`, `len(large)=1`, equal → `(-small[0] + large[0])/2 = (-(-1) + 2)/2 = (1+2)/2 = 1.5` ✅

`addNum(3)`: push `-3` to small → `small=[-3,-1]`. Pop small's max: pops `-3`, negate→`3`, push to large → `large=[2,3]` (heap order, `2` on top). `small` now `[-1]`. Check sizes: `len(large)=2 > len(small)=1`? Yes → move large's min (`2`) back: pop `2` from large, push `-2` to small → `small=[-2,-1]`, `large=[3]`.

`findMedian()`: `len(small)=2 > len(large)=1` → return `-small[0]` → `small[0]` is `-2` (the min of the negated heap, i.e. the actual max of `small`) → `-(-2) = 2` ✅ (median of `[1,2,3]` is indeed `2`).

### Time & space complexity

- **Time: `addNum` is O(log n)** — a constant number of heap push/pop operations (three, at most), each O(log n) per the topic overview's operation-cost table. **`findMedian` is O(1)** — just peeking at both heaps' roots, no computation.
- **Space: O(n)** — both heaps together hold every number added so far, one heap or the other, with no duplication or waste.

---

## Common mistakes & misconceptions

1. **Forgetting to negate values for the max-heap (`small`).** Without negation, `heapq` operations on `small` would behave as a min-heap, meaning `small[0]` would be the *smallest* of the small half, not the largest — exactly the wrong value for finding the boundary closest to the median.
2. **Skipping the size-rebalancing step.** Without it, one heap could grow arbitrarily larger than the other over many `addNum` calls, and the median formula (which assumes a specific, bounded size relationship between the two heaps) would silently produce wrong answers.
3. **Comparing heap sizes with `>=` instead of `>` in the rebalance check**, which can cause an unnecessary or incorrect rebalance and subtly break the intended "small has 0 or 1 more elements than large" invariant.
4. **Trying to skip the "route everything through `small` first" step and instead deciding upfront which heap a new number belongs in** (e.g. by comparing it to the current medians). This is a common instinct, but it requires extra edge-case handling (what if the heaps are currently empty? what if the new number is exactly at the boundary?) that the "always push to `small`, then shuffle and rebalance" approach sidesteps entirely by making the shuffle step do that decision-making implicitly and uniformly.

## Summary

| Approach | addNum | findMedian | Notes |
|---|---|---|---|
| Sorted list (insert in place) | O(n) | O(1) | Insertion position found fast, but shifting elements is slow. |
| Unsorted list, sort on query | O(1) | O(n log n) | Fine if `findMedian` is called rarely relative to `addNum`. |
| Two Heaps | O(log n) | O(1) | The standard, best-in-class solution when both operations are frequent. |

**Key takeaway:** "maintain a running statistic (median, in this case) as a stream of data arrives" problems often benefit from splitting the data into two balanced groups tracked by two heaps, rather than keeping everything in one fully-sorted structure. This two-heap technique is specifically worth remembering as a named pattern — it doesn't generalize as broadly as the single-heap "top k" pattern, but it's the standard, expected tool for streaming-median problems specifically.
