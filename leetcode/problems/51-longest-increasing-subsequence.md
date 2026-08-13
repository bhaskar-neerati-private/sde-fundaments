# 51. Longest Increasing Subsequence

**LeetCode:** [#300 - Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) · **Topic:** [1-D Dynamic Programming](../topics/13-1d-dynamic-programming.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums`, return the length of the longest **strictly increasing subsequence** (elements don't need to be contiguous in the array, just appear in increasing order and in the same relative order as the original array).

**Example:**
```
Input: nums = [10,9,2,5,3,7,101,18]
Output: 4   ([2,3,7,101] or [2,3,7,18])
```

## Applicable approaches

- **Brute Force - Check every subsequence.** Exponential, shown for intuition only.
- **DP - "longest sequence ending at i"** - O(n²), the standard expected solution.
- **Optimal - Binary Search + Patience Sorting** - O(n log n), a further optimization worth knowing exists.

## Approach 1 (Conceptual): Brute Force

### Intuition
Try every possible subsequence (2^n of them) and check which are strictly increasing, tracking the longest. This is exponential and impractical - shown only to motivate why we need a smarter approach.

---

## Approach 2: DP - "Longest Sequence Ending at Index i"

### Intuition
Define `dp[i]` = the length of the longest increasing subsequence that **ends exactly at index i** (using `nums[i]` as its last element). To compute `dp[i]`, look at every earlier index `j < i`: if `nums[j] < nums[i]`, then we could extend whatever subsequence ends at `j` by adding `nums[i]` onto the end of it - so `dp[i]` could be `dp[j] + 1`. Take the best such extension over all valid `j`. The final answer is the **maximum** value across the entire `dp` array (the longest subsequence might end anywhere, not necessarily at the last index).

### Algorithm
1. Initialize `dp = [1] * n` (every single element, by itself, is a valid increasing subsequence of length 1 - the base case for every position).
2. For each `i` from 1 to n-1: for each `j` from 0 to i-1: if `nums[j] < nums[i]`, update `dp[i] = max(dp[i], dp[j] + 1)`.
3. Return `max(dp)`.

### Python code
```python
def lengthOfLIS(nums):
    n = len(nums)
    dp = [1] * n

    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)
```

### Line-by-line explanation
- `dp = [1] * n` - every element alone forms a subsequence of length 1; this is both the base case and the minimum possible value for every position.
- `for i in range(1, n):` - compute `dp[i]` for each position, building left to right.
- `for j in range(i):` - check every earlier position as a potential predecessor.
- `if nums[j] < nums[i]:` - only a strictly smaller earlier value can be extended by `nums[i]` while staying strictly increasing.
- `dp[i] = max(dp[i], dp[j] + 1)` - if extending the subsequence ending at `j` (which has length `dp[j]`) by adding `nums[i]` gives a longer result than what we've found so far for position `i`, take it.
- `return max(dp)` - the overall longest increasing subsequence could end at any position, so scan the whole `dp` array for the best.

### Dry run
`nums = [10,9,2,5,3,7,101,18]`

`dp` starts as `[1,1,1,1,1,1,1,1]` (indices 0-7, values 10,9,2,5,3,7,101,18).

- `i=1` (nums[1]=9): `j=0` (nums[0]=10): `10<9`? No. `dp[1]` stays `1`.
- `i=2` (nums[2]=2): `j=0`(10<2? No), `j=1`(9<2? No). `dp[2]=1`.
- `i=3` (nums[3]=5): `j=0`(10<5?No), `j=1`(9<5?No), `j=2`(2<5?Yes)→`dp[3]=max(1,dp[2]+1)=max(1,2)=2`.
- `i=4` (nums[4]=3): `j=2`(2<3?Yes)→`dp[4]=max(1,dp[2]+1)=2`. (j=0,1,3 don't help: 10,9 not <3; nums[3]=5 not<3).
- `i=5` (nums[5]=7): `j=2`(2<7)→`dp[5]=max(1,dp[2]+1)=2`. `j=3`(5<7)→`dp[5]=max(2,dp[3]+1)=max(2,3)=3`. `j=4`(3<7)→`dp[5]=max(3,dp[4]+1)=max(3,3)=3`.
- `i=6` (nums[6]=101): every earlier value is <101 → `dp[6]=max over all dp[j]+1 for j=0..5` = `dp[5]+1=4` is the best (since dp[5]=3 is the max among dp[0..5]). `dp[6]=4`.
- `i=7` (nums[7]=18): `j=5`(7<18)→`dp[7]=max(1,dp[5]+1)=4`. `j=6`(101<18? No). Other j's (2,5,3 all <18) give at most `dp[4]+1=3` or similar, less than 4. `dp[7]=4`.

Final `dp = [1,1,1,2,2,3,4,4]`. `max(dp) = 4` ✅ matches expected output (corresponding to subsequences like `[2,3,7,101]` or `[2,3,7,18]`, both length 4).

### Time & space complexity
- **Time: O(n²)** - for each of n positions, checking all earlier positions.
- **Space: O(n)** for the `dp` array.

---

## Approach 3: Optimal - Binary Search + Patience Sorting (O(n log n))

### Intuition
This is a cleverer, less obviously-intuitive approach worth knowing exists (a common O(n log n) follow-up in interviews). Maintain a list `tails`, where `tails[k]` represents **the smallest possible "tail" value** of any increasing subsequence of length `k+1` found so far (not necessarily a real subsequence itself - just tracking the best possible ending value for each length). For each new number, use **binary search** to find where it belongs in `tails`: if it's larger than everything in `tails`, it extends the longest subsequence found so far (append it). Otherwise, it replaces the first element in `tails` that's ≥ it (keeping `tails` as small/optimal as possible for future extensions, without changing the *length* of the longest subsequence found so far at that point).

### Python code
```python
import bisect

def lengthOfLIS(nums):
    tails = []

    for num in nums:
        pos = bisect.bisect_left(tails, num)
        if pos == len(tails):
            tails.append(num)
        else:
            tails[pos] = num

    return len(tails)
```

### Line-by-line explanation
- `tails = []` - `tails[k]` will represent the smallest tail value achievable for an increasing subsequence of length `k+1`.
- `pos = bisect.bisect_left(tails, num)` - binary search for where `num` would need to be inserted to keep `tails` sorted (finds the leftmost position where `num` could go).
- `if pos == len(tails): tails.append(num)` - `num` is larger than everything currently in `tails`, meaning it can extend the longest subsequence found so far by one - append it, increasing the tracked length.
- `else: tails[pos] = num` - `num` isn't large enough to extend the longest subsequence, but it **can** replace the existing value at `tails[pos]` with a smaller one (since `num` is smaller, by the nature of where binary search placed it) - this doesn't change the *length* recorded so far, but gives future numbers a better (smaller) value to potentially extend from, which could allow longer subsequences later.
- `return len(tails)` - the final length of `tails` is exactly the length of the longest increasing subsequence (even though `tails` itself, after replacements, may not represent an actual valid subsequence from the original array - only its *length* is guaranteed correct).

### Why this works (brief intuition, not a full proof)
`tails` is always sorted, and its length always equals the length of the longest increasing subsequence found *so far*. Replacing a value in `tails` with a smaller one can only ever help (or do nothing) for future numbers, never hurt - a smaller tail value for a given subsequence length is strictly more flexible for extending later, so this greedy replacement never causes us to miss a longer subsequence that could otherwise have been found.

### Time & space complexity
- **Time: O(n log n)** - n numbers, each requiring an O(log n) binary search.
- **Space: O(n)** for the `tails` array in the worst case (a fully increasing input).

---

## Common mistakes & misconceptions

1. **Believing `dp[i]` represents "the LIS within `nums[0:i+1]`" rather than "the LIS ending exactly at index i."** This distinction is critical - `dp[i]` specifically requires `nums[i]` to be the *last* element used, which is why the final answer needs `max(dp)` rather than simply `dp[n-1]` (the longest subsequence overall might end at any index, not necessarily the last one).
2. **Using `<=` instead of `<` when comparing `nums[j]` and `nums[i]`.** The problem asks for **strictly** increasing - using `<=` would incorrectly allow equal values to extend a subsequence, which is a different problem (non-decreasing subsequence).
3. **Believing the `tails` array in the O(n log n) approach represents an actual valid subsequence.** It doesn't, in general - after replacements, `tails` only reliably represents the correct *length*, not necessarily elements that ever appeared together in a real increasing subsequence from the original array; trying to read an actual answer sequence out of `tails` directly is a common and understandable but incorrect assumption.
4. **Assuming the O(n log n) approach is "the same DP, just faster."** It's a genuinely different algorithmic idea (greedy tail-tracking + binary search, not a filled-in DP table) - being honest about this distinction matters, since the two approaches don't share intermediate state or reasoning, only the final answer's correctness.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute force | O(2^n) | O(n) | Impractical, exponential. |
| DP ("longest ending at i") | O(n²) | O(n) | The standard, most commonly taught solution. |
| Binary Search + Patience Sorting | O(n log n) | O(n) | A well-known further optimization, good to be aware of even if the O(n²) DP is your primary answer. |

**Key takeaway:** the O(n²) DP approach - "the best answer ending exactly at position i, built from all valid earlier positions" - is the standard entry point for this whole family of "longest/best subsequence" problems; the O(n log n) binary search trick is a specific, clever optimization worth knowing exists, but the DP formulation is the transferable skill that generalizes to many other problems in this topic.
