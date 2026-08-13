# 8. Longest Consecutive Sequence

**LeetCode:** [#128 - Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Medium

## Problem statement

Given an unsorted array of integers `nums`, return the length of the longest run of **consecutive** integers (e.g. `[3,4,5,6]` is a consecutive run of length 4). The numbers don't need to be consecutive *in the array* — just consecutive in *value* — and you must find them in **O(n)** time.

**Example:**
```
Input: nums = [100,4,200,1,3,2]
Output: 4
Explanation: The longest consecutive run of values is [1, 2, 3, 4]. Length = 4.
```

## Applicable approaches

- **Brute Force** — for every number, keep counting upward as long as the next number exists in the array.
- **Better — Sorting** — sort first, then scan for consecutive runs.
- **Optimal — Hash Set with Smart Starting Points** — the standard O(n) expected solution.

## Approach 1: Brute Force

### Intuition

The most direct interpretation: for every number in the array, try extending a run upward one step at a time (`num+1`, `num+2`, ...), checking each time whether the next number exists anywhere in the array, and track the longest run found this way. This is correct — every possible run does get discovered from *some* starting point — but it does an enormous amount of avoidable, repeated existence-checking, as the complexity analysis below makes precise.

### Algorithm

1. For each number `num` in `nums`: starting from `num`, keep checking if `num + 1`, `num + 2`, ... exist in `nums` (using `in`, which searches the whole array), counting how far this run extends.
2. Track the maximum run length found across all starting numbers.

### Python code
```python
def longestConsecutive(nums):
    longest = 0
    for num in nums:
        length = 1
        while (num + length) in nums:  # O(n) search each time!
            length += 1
        longest = max(longest, length)
    return longest
```

### Line-by-line explanation

- `for num in nums` — try starting a sequence from *every* single number, including numbers that are obviously in the *middle* of a run, not just its start — this "trying from everywhere" is the specific redundancy the optimal approach eliminates.
- `while (num + length) in nums` — keep extending as long as the next value exists somewhere in the array. **`in nums` on a `list` is O(n)** — this is the expensive part, and it's called potentially many times per starting number.
- `length += 1` — extend the run.
- `longest = max(longest, length)` — track the best run seen so far.

### Dry run (conceptual)

`nums = [100,4,200,1,3,2]` — starting from `1`: is `2` in nums? yes. Is `3`? yes. Is `4`? yes. Is `5`? no. Run length = 4. Starting from `100`: is `101` in nums? no. Run length = 1. The correct answer (4) does get found this way — but notice we *also* separately started runs from `2`, `3`, and `4` (each re-discovering part of the same `[1,2,3,4]` run from a later starting point), redoing work that starting from `1` had already covered.

### Time & space complexity

- **Time: O(n³)** in the worst case — O(n) starting numbers, each potentially extending up to O(n) steps, each step doing an O(n) `in`-on-list search. (Some looser analyses call this O(n²) by not fully accounting for run length, but the honest worst case, with an O(n) list search at every step of an O(n)-long run tried from O(n) starting points, is cubic.)
- **Space: O(1)** extra.

---

## Approach 2: Better — Sorting

### Intuition

The brute force's two separate wastes are: (1) re-searching the array with a linear scan every time it asks "does this value exist," and (2) starting a fresh run-attempt from numbers that are actually in the *middle* of a run someone else already covers. Sorting fixes the first waste directly: once sorted, consecutive values (after removing duplicates) sit **adjacent** to each other, so checking "is the next value present" becomes "is the next array slot exactly one more than this slot" — an O(1) comparison instead of an O(n) search.

### Algorithm

1. Sort the array (and it's worth deduplicating too, e.g. via `set(nums)`, since duplicates would otherwise incorrectly extend a run's length — two equal adjacent values aren't a "run of 2 consecutive integers").
2. Walk through the sorted, deduplicated list. Keep a running current-run length.
3. If the current number is exactly one more than the previous number, extend the run. Otherwise (there's a gap, or it's the very first number), reset the run to length 1.
4. Track the maximum run length seen.

### Python code
```python
def longestConsecutive(nums):
    if not nums:
        return 0

    nums = sorted(set(nums))
    longest = 1
    current = 1

    for i in range(1, len(nums)):
        if nums[i] == nums[i - 1] + 1:
            current += 1
        else:
            current = 1
        longest = max(longest, current)

    return longest
```

### Line-by-line explanation

- `if not nums: return 0` — handle the empty-input edge case; without this, `longest = 1` below would be wrong (there is no run of length 1 if there are no elements at all).
- `nums = sorted(set(nums))` — `set(nums)` removes duplicates first (necessary, as explained above), `sorted(...)` orders them ascending.
- `longest = 1; current = 1` — with at least one number present (guaranteed by the check above), the minimum possible answer is 1 — a single number is trivially a "run" of length 1.
- `if nums[i] == nums[i - 1] + 1: current += 1` — consecutive, extend the current run.
- `else: current = 1` — a gap (or, after deduplication, this branch is only ever reached for a genuine gap, not a duplicate) — the run restarts here.
- `longest = max(longest, current)` — keep track of the best run found so far.

### Dry run

`nums = [100,4,200,1,3,2]` → dedup+sort: `[1, 2, 3, 4, 100, 200]`

| i | nums[i] | nums[i-1]+1 | consecutive? | current | longest |
|---|---|---|---|---|---|
| 1 | 2 | 1+1=2 | yes | 2 | 2 |
| 2 | 3 | 2+1=3 | yes | 3 | 3 |
| 3 | 4 | 3+1=4 | yes | 4 | 4 |
| 4 | 100 | 4+1=5 | no | 1 | 4 |
| 5 | 200 | 100+1=101 | no | 1 | 4 |

Final answer: `4` ✅

### Time & space complexity

- **Time: O(n log n)** — dominated by sorting; the scan afterward is O(n).
- **Space: O(n)** — the `set` and sorted list.

---

## Approach 3: Optimal — Hash Set with Smart Starting Points

### Intuition

Sorting fixes the "O(n) search per check" waste, but still pays O(n log n) for ordering — and it still doesn't fix the second waste identified earlier: even in the sorted version, we visit *every* element as part of *some* run-scan, which is unavoidable and fine, but the brute force's deeper issue was trying to start a **new count** from numbers that are provably not the start of their run. The key realization that gets us to true O(n): **we don't need to try extending a sequence from every number — only from numbers that are the true start of a sequence**, meaning `num - 1` does **not** exist in the set. If `num - 1` *does* exist, `num` cannot be the start of the longest run containing it — some earlier number already owns that role, and the full run will be correctly (and only once) discovered when we start from *that* earlier number instead.

Why this guarantees O(n) overall, not just "usually fast": each number in the array can be the *starting point* of an extension-walk at most once (only if it passes the `num - 1 not in set` filter), and each number can be *walked over* as part of someone else's extension at most once (once a number has been included in a run extending from its true start, no other starting point will ever reach it, because every other number in that same run fails the "is this a true start" filter and never launches its own walk). So the *total* work across the whole algorithm — filter-checks plus all extension-walk steps combined — is bounded by a small constant times n, not by n times the average run length.

### Algorithm

1. Put all numbers into a hash set (O(1) lookup, and automatically deduplicates).
2. For each number `num` in the set: check if `num - 1` is in the set.
3. If `num - 1` **is** in the set, skip — `num` is not the start of its sequence, some other number is.
4. If `num - 1` is **not** in the set, `num` is a genuine sequence start. Count upward (`num+1`, `num+2`, ...) as long as each next number is in the set, tracking the length.
5. Track the maximum length found across all valid starting points.

### Python code
```python
def longestConsecutive(nums):
    num_set = set(nums)
    longest = 0

    for num in num_set:
        if (num - 1) not in num_set:  # only start counting from the beginning of a sequence
            length = 1
            while (num + length) in num_set:
                length += 1
            longest = max(longest, length)

    return longest
```

### Line-by-line explanation

- `num_set = set(nums)` — O(1) average lookup for "does this value exist?", and duplicates are automatically removed.
- `for num in num_set` — consider every unique number as a *potential* sequence start; this loop alone is O(n), and the crucial claim is that the work done *inside* it, summed across all iterations, is also O(n), not O(n) per iteration.
- `if (num - 1) not in num_set` — **this is the entire mechanism that makes the algorithm O(n) instead of looking like it should be O(n²)**: it's a single O(1) check that determines whether this number is worth exploring from at all.
- `length = 1; while (num + length) in num_set: length += 1` — starting from a confirmed sequence start, walk upward one value at a time, extending the count as long as the next consecutive value exists.
- `longest = max(longest, length)` — track the best run.

### Why this is O(n) overall (not O(n²)) — the rigorous version

It looks like there's a loop inside a loop (`for num` with a `while` inside), which superficially suggests O(n²). But the `if (num - 1) not in num_set` check guarantees the inner `while` loop only ever *runs* for numbers that are true sequence starts — and, critically, **every number in the entire dataset can only ever be "walked over" by the inner while loop once**, as part of extending its own sequence's single, unique start. No number gets extended-into by more than one starting point, because being extended-into by a walk from `num` means `num + 1` (say) has a predecessor (`num`) in the set — which is exactly the condition that disqualifies `num + 1` from ever launching its *own* walk. So the *total* work done by all the inner while-loop iterations, summed across every outer iteration, is at most n — giving O(n) overall, not O(n²). This is the same "amortized across the whole algorithm, not per-iteration" reasoning used to show Sliding Window algorithms are linear despite a nested-loop appearance.

### Dry run

`nums = [100,4,200,1,3,2]` → `num_set = {100, 4, 200, 1, 3, 2}`

| num | num-1 in set? | is a start? | if yes: count upward | length | longest |
|---|---|---|---|---|---|
| 100 | 99 not in set | **yes** | 101 not in set → stop | 1 | 1 |
| 4 | 3 **in** set | no, skip | — | — | 1 |
| 200 | 199 not in set | **yes** | 201 not in set → stop | 1 | 1 |
| 1 | 0 not in set | **yes** | 2✅,3✅,4✅,5❌ → stop | 4 | 4 |
| 3 | 2 **in** set | no, skip | — | — | 4 |
| 2 | 1 **in** set | no, skip | — | — | 4 |

Only `1`, `100`, and `200` were ever true starting points (their `num-1` wasn't in the set), so the inner while loop only ran for those three — and the sequence `[1,2,3,4]` was only walked once, starting from `1`, touching `2`, `3`, `4` exactly once each as part of that single walk. Final answer: `4` ✅

(Iteration order over a Python `set` isn't guaranteed, so the exact order of this table can vary run to run, but the result and the "only 3 starts checked" property hold regardless of order — this is worth noting because it's tempting to assume set iteration is insertion-ordered the way dicts are in modern Python; sets make no such guarantee.)

### Time & space complexity

- **Time: O(n)** — every number is looked at a constant number of times: once as a candidate in the outer loop (checking `num - 1 not in num_set`), and at most once total as part of some sequence's inner while-loop walk (per the argument above).
- **Space: O(n)** — the hash set.

---

## Common mistakes & misconceptions

1. **Trying to launch an extension-walk from every number, forgetting the `num - 1 not in num_set` filter.** This silently degrades the algorithm back toward the brute force's redundancy — it will still produce the *correct* answer, but loses the O(n) guarantee, since numbers in the middle of long runs get walked over repeatedly from multiple starting points.
2. **Forgetting to deduplicate before counting, in the sorting approach.** Two adjacent equal values (e.g. `[1, 1, 2]` sorted) are not "consecutive integers" in the sense the problem means — using a plain `sorted(nums)` instead of `sorted(set(nums))` can inflate the run length incorrectly if you don't also explicitly skip equal-adjacent values in the scan.
3. **Assuming set iteration order matches insertion order.** Unlike Python's `dict` (which preserves insertion order as a language guarantee since 3.7), `set` iteration order is unspecified — code that implicitly relies on set order (e.g. assuming the smallest number is checked first) is relying on an accident of the current implementation, not a guarantee.
4. **Confusing this problem with "Longest Increasing Subsequence."** They sound similar but are different problems: this one requires values that are literally consecutive integers (`n, n+1, n+2, ...`), appearing anywhere in the array in any order; LIS (covered later, in the DP topic) only requires values to be increasing and in their original relative order, with no requirement that they be consecutive integers. The two problems need genuinely different techniques — don't reach for O(n²) DP here, and don't reach for a hash set for LIS.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n³) worst case | O(1) | Extremely slow; `in` on a list inside nested loops, plus redundant restart points. |
| Sorting | O(n log n) | O(n) | Simple and correct, but not the required O(n). |
| Hash Set + smart starts | O(n) | O(n) | The correct, expected optimal solution. |

**Key takeaway:** when a brute-force idea does redundant work (here: trying to extend a sequence from *every* number, even ones in the middle of a run that will get covered anyway), look for a cheap O(1) check that lets you **skip work you can prove will be repeated elsewhere** — here, "is this the true start of its run?" That one `if` check is what brings an apparent O(n²) nested loop down to a rigorously provable true O(n).
