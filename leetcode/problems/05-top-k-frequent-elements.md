# 5. Top K Frequent Elements

**LeetCode:** [#347 - Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. You may return the answer in any order.

**Example:**
```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```

## Applicable approaches

- **Brute Force** — count frequencies, then sort by frequency.
- **Better — Heap** — count frequencies, then use a heap to get the top k without a full sort.
- **Optimal — Bucket Sort** — count frequencies, then bucket by frequency for a true O(n) solution.

**What doesn't apply, and why:** Two Pointers and Sliding Window don't apply — this problem has no notion of a contiguous window or a sorted-array boundary to walk inward from; it's fundamentally a "count, then select the best k" problem, not a positional-scanning one.

## Approach 1: Brute Force — Count, then Sort

### Intuition

Break the problem into two separate, simpler questions: first, "how often does each number appear?" (frequency counting — the same pattern from Valid Anagram), and second, "which k numbers have the highest counts?" The most direct way to answer the second question, once you have the counts, is to sort everything by frequency and take the top k — but notice this fully *orders* every unique number by frequency, when the problem only asked for the top k, not a complete ranking. That gap is exactly what the later approaches exploit.

### Algorithm

1. Build a frequency dict: number → count.
2. Get the list of `(number, count)` pairs.
3. Sort this list by count, descending.
4. Take the first `k` numbers from the sorted list.

### Python code
```python
def topKFrequent(nums, k):
    counts = {}
    for num in nums:
        counts[num] = counts.get(num, 0) + 1

    sorted_items = sorted(counts.items(), key=lambda pair: pair[1], reverse=True)
    return [num for num, freq in sorted_items[:k]]
```

### Line-by-line explanation

- `counts = {}` then the loop — standard frequency counting, same as earlier problems.
- `counts.items()` — gives `(number, count)` pairs.
- `sorted(..., key=lambda pair: pair[1], reverse=True)` — sorts those pairs by the second element (`pair[1]`, the count) from highest to lowest. **This is the step that does more work than necessary**: full sorting establishes a total order over *every* unique number, even though we only need to distinguish "the top k" from "everything else" — we never actually need to know, say, whether the 40th-most-frequent number beats the 41st.
- `sorted_items[:k]` — the first k entries after sorting are the k most frequent.
- `[num for num, freq in ...]` — extract just the numbers (we don't need to return the counts, only the values).

### Dry run

`nums = [1,1,1,2,2,3]`, `k = 2`

- counts: `{1: 3, 2: 2, 3: 1}`
- items sorted by count descending: `[(1,3), (2,2), (3,1)]`
- take first 2: `[(1,3), (2,2)]`
- extract numbers: `[1, 2]` ✅

### Time & space complexity

- **Time: O(n log n)** — building counts is O(n), but sorting the (up to n) unique items is O(n log n), which dominates. The inefficiency, precisely: we're paying a full-sort cost to answer a "top k" question, when k could be far smaller than the number of unique values.
- **Space: O(n)** — the frequency dict and the sorted list.

---

## Approach 2: Better — Heap (Priority Queue)

### Intuition

Sorting *all* unique numbers by frequency does strictly more work than the problem needs, precisely because it fully orders elements we're going to throw away. A **min-heap of size k** fixes this directly: instead of sorting everything and slicing off the top k afterward, maintain *only* the k best candidates seen so far, discarding the worst one whenever a better candidate shows up. The heap never needs to know the relative order of anything outside the current top-k — that's the entire savings.

The mechanism: push each `(frequency, number)` pair onto the heap; whenever the heap grows past size k, pop the *smallest* frequency in it (Python's `heapq` is a min-heap, so popping removes the smallest first) — since a min-heap of size k always keeps the k *largest* values seen so far by continuously evicting whichever is currently the weakest of that top-k group.

### Algorithm

1. Build the frequency dict as before.
2. Create an empty min-heap.
3. For each `(number, count)` pair: push it onto the heap.
4. If the heap grows past size `k`, pop the smallest.
5. After processing all pairs, the heap contains the k largest-frequency items.

### Python code
```python
import heapq

def topKFrequent(nums, k):
    counts = {}
    for num in nums:
        counts[num] = counts.get(num, 0) + 1

    heap = []
    for num, freq in counts.items():
        heapq.heappush(heap, (freq, num))
        if len(heap) > k:
            heapq.heappop(heap)

    return [num for freq, num in heap]
```

### Line-by-line explanation

- `counts` — same frequency count as before.
- `heap = []` — Python's `heapq` module works on a plain list, treating it as a min-heap (smallest item is always at the front / poppable first).
- `heapq.heappush(heap, (freq, num))` — push a `(frequency, number)` tuple. Tuples compare element-by-element, so the heap orders primarily by frequency; `number` only acts as a tiebreaker if two frequencies match (and is never actually compared unless frequencies tie, since Python short-circuits tuple comparison at the first differing element).
- `if len(heap) > k: heapq.heappop(heap)` — **this is the entire mechanism that avoids full sorting**: if the heap has grown beyond size k, remove the smallest-frequency item — keeping the heap at exactly size k, always holding the k *largest* frequencies seen so far, without ever needing to compare or order anything beyond that boundary.
- `[num for freq, num in heap]` — after processing everything, extract just the numbers from the remaining k pairs.

### Dry run

`nums = [1,1,1,2,2,3]`, `k = 2` → counts (built in insertion order `1,2,3`): `{1: 3, 2: 2, 3: 1}`

| num | freq | pair pushed | heap after push+trim |
|---|---|---|---|
| 1 | 3 | (3,1) | `[(3,1)]` |
| 2 | 2 | (2,2) | `[(2,2),(3,1)]` |
| 3 | 1 | (1,3) | push → `[(1,3),(3,1),(2,2)]`, size 3 > 2, pop smallest = `(1,3)` → `[(2,2),(3,1)]` |

Final heap holds `(2,2)` and `(3,1)` → numbers `2` and `1` → `[2, 1]` or `[1, 2]` depending on iteration order (a heap's internal list isn't fully sorted, just "any order" which the problem allows). Both `1` (freq 3) and `2` (freq 2) are correctly the two most frequent — `3` (freq 1) was correctly evicted the moment the heap exceeded size k, without ever needing to compare it against the eventual full ranking.

### Time & space complexity

- **Time: O(n log k)** — n unique numbers processed, each heap push/pop costs O(log k) since the heap never grows past size k+1 (the trim happens immediately after each push). This is strictly better than O(n log n) whenever k is meaningfully smaller than the number of unique values — the entire improvement over Approach 1 comes from bounding the heap's size at k instead of letting it grow to n.
- **Space: O(n + k)** — O(n) for the frequency dict, O(k) for the heap.

---

## Approach 3: Optimal — Bucket Sort

### Intuition

Both previous approaches pay a logarithmic factor (from sorting or heap operations) because they treat frequency as an unbounded value that needs comparison-based ordering. But frequency isn't unbounded here: **a number can appear at most `n` times** (n = length of the array), so frequency values only ever range from `1` to `n` — a small, bounded range. Whenever a value you're "sorting by" is a bounded integer, you can often skip comparison-based sorting entirely and use its value directly as an array index — this is the core idea of bucket sort, and it's why this approach achieves true O(n), no log factor anywhere.

Concretely: create `n` "buckets," where bucket `i` holds every number that appears **exactly `i` times**. Walking the buckets from highest frequency to lowest and collecting numbers requires no comparisons at all — just placing things in the right bucket and reading them back out in a fixed order.

### Algorithm

1. Build the frequency dict as before.
2. Create a list of `n + 1` empty buckets (index 0 to n), where `buckets[freq]` holds every number with that exact frequency.
3. For each `(number, freq)` pair, append `number` to `buckets[freq]`.
4. Walk the buckets from index `n` down to `1` (highest frequency first). For each bucket, add its numbers to the result list.
5. Stop as soon as the result list has `k` numbers.

### Python code
```python
def topKFrequent(nums, k):
    n = len(nums)
    counts = {}
    for num in nums:
        counts[num] = counts.get(num, 0) + 1

    buckets = [[] for _ in range(n + 1)]  # buckets[freq] = list of numbers with that frequency
    for num, freq in counts.items():
        buckets[freq].append(num)

    result = []
    for freq in range(n, 0, -1):
        for num in buckets[freq]:
            result.append(num)
            if len(result) == k:
                return result
    return result
```

### Line-by-line explanation

- `n = len(nums)` — the maximum possible frequency any single number can have; this bound is what makes bucketing possible instead of needing an unbounded sort.
- `counts` — frequency dict, same as always.
- `buckets = [[] for _ in range(n + 1)]` — a list of n+1 empty lists, **built with a list comprehension, not `[[]] * (n+1)`** — this distinction matters: `[[]] * (n+1)` would create n+1 references to the *same* inner list, so appending to `buckets[3]` would incorrectly also affect `buckets[7]` and every other slot. `buckets[0]` is unused in practice (nothing has frequency 0), `buckets[n]` would hold a number that's the *only* value in the whole array.
- `for num, freq in counts.items(): buckets[freq].append(num)` — place each number into the bucket matching its exact frequency. Numbers with the same frequency land in the same bucket together; this is O(1) per number, since it's a direct index computation, not a comparison.
- `for freq in range(n, 0, -1)` — walk buckets from the highest possible frequency down to 1 — this naturally visits numbers from most frequent to least frequent, with zero sorting, because the bucket *index itself* already encodes the ordering.
- `for num in buckets[freq]: result.append(num)` — collect all numbers at this frequency level.
- `if len(result) == k: return result` — stop the instant we've collected exactly k numbers — no need to keep scanning lower-frequency buckets, which is a further saving on top of the algorithm already being O(n).

### Dry run

`nums = [1,1,1,2,2,3]`, `k = 2`, `n = 6`

- counts: `{1: 3, 2: 2, 3: 1}`
- buckets (index = frequency): `buckets[1] = [3]`, `buckets[2] = [2]`, `buckets[3] = [1]`, everything else (`buckets[0]`, `buckets[4..6]`) empty.
- Walk from `freq = 6` down to `1`:
  - freq 6, 5, 4: empty, skip.
  - freq 3: `buckets[3] = [1]` → result = `[1]`, len 1 ≠ k(2), keep going.
  - freq 2: `buckets[2] = [2]` → result = `[1, 2]`, len 2 == k → return `[1, 2]` ✅

### Time & space complexity

- **Time: O(n).** Building counts is O(n), building buckets is O(n) (one bucket-append per unique number, and the total number of appends across all buckets can't exceed the number of unique values, which is ≤ n), and reading buckets back out visits at most n items total across all frequency levels. No sorting, no log factor anywhere — this is the truly optimal answer, and the reason is specifically that frequency is a *bounded integer* we can use as a direct array index rather than something we need comparison-based ordering for.
- **Space: O(n)** — the counts dict and the buckets list.

---

## Common mistakes & misconceptions

1. **Building buckets with `[[]] * (n+1)` instead of a list comprehension.** This is a subtle, classic Python bug: it creates n+1 *references* to a single shared list, so every "different" bucket is secretly the same list — appending to one appends to all. Always use `[[] for _ in range(n+1)]` when you need genuinely independent inner lists.
2. **Using a max-heap of ALL elements instead of a min-heap of size k.** Some intuition-first attempts push every element into a max-heap and pop k times — this is O(n + k log n), which is *worse* than the size-k min-heap approach (O(n log k)) whenever k is small relative to n, because it still pays to maintain heap order across the *entire* dataset rather than just the top-k window.
3. **Forgetting the early-return in bucket sort.** Without `if len(result) == k: return result`, the bucket-walk would needlessly continue scanning lower-frequency buckets after already collecting enough — still O(n) overall since the scan is bounded, but the early exit is a meaningful, free constant-factor improvement worth including.
4. **Assuming heap tuples `(freq, num)` and `(num, freq)` are interchangeable.** They're not: `heapq` compares tuples element-by-element, so the *first* element determines the primary sort key. Putting `num` first would make the heap order (and therefore evict) by number value instead of frequency — a silent correctness bug, not just a style difference.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Count + Sort | O(n log n) | O(n) | Simplest to write, does more work than necessary — fully orders everything, not just the top k. |
| Count + Heap (size k) | O(n log k) | O(n + k) | Better when k is much smaller than n — only ever orders k things at once. |
| Count + Bucket Sort | O(n) | O(n) | True optimal; relies on frequency being bounded by n, so no comparison-based ordering is needed at all. |

**Key takeaway:** whenever you need the "top k" or "k most/least frequent" and the values being ranked are counts (which are always bounded, non-negative integers no larger than n), bucket sort can eliminate sorting entirely by using the bounded value directly as an array index. This "bounded range → use it as array indices instead of sorting" trick is worth remembering well beyond just this problem.
