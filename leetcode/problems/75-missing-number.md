# 75. Missing Number

**LeetCode:** [#268 - Missing Number](https://leetcode.com/problems/missing-number/) · **Topic:** [Bit Manipulation](../topics/18-bit-manipulation.md) · **Difficulty:** Easy

## Problem statement

Given an array `nums` containing `n` distinct numbers taken from the range `[0, n]` (so exactly one number in that range is missing), return the missing number.

**Example:**
```
Input: nums = [3,0,1]
Output: 2   (n=3, range is [0,3], and 2 is missing)
```

## Applicable approaches

- **Sorting.**
- **Hash Set.**
- **XOR Bit Trick.**
- **Sum Formula (Math).**

All four are valid and worth knowing; this problem is a great one for seeing several different topics' techniques applied to the same task.

## Approach 1: Sorting

### Intuition
Sort the array; the missing number is wherever the sorted sequence "skips" a value (or it's missing from either end).

### Python code
```python
def missingNumber(nums):
    nums = sorted(nums)
    for i, num in enumerate(nums):
        if num != i:
            return i
    return len(nums)
```

### Time & space complexity
- **Time: O(n log n)** - dominated by the sort.
- **Space: O(n)** or O(1) depending on sort implementation/whether input mutation is allowed.

---

## Approach 2: Hash Set

### Intuition
Put every number from `nums` into a set, then check which value in the full expected range `[0, n]` is absent.

### Python code
```python
def missingNumber(nums):
    num_set = set(nums)
    n = len(nums)

    for i in range(n + 1):
        if i not in num_set:
            return i
```

### Time & space complexity
- **Time: O(n)**, **Space: O(n)** for the set.

---

## Approach 3: XOR Bit Trick

### Intuition
Recall XOR's "cancel out" property: `x ^ x = 0`. If we XOR together **every index from 0 to n** *and* **every value in `nums`**, every number that's present in `nums` (and therefore also in the range) will be XORed with itself exactly once and cancel out to 0 - leaving only the **one** number that's in the range `[0,n]` but *not* in `nums` (the missing one), since it never gets paired with a matching cancel-partner.

### Algorithm
1. Initialize `result = n` (accounts for the index `n` itself, which isn't naturally covered by a simple 0-to-n-1 enumeration loop over the array - see below).
2. For each index `i` and value `nums[i]`: XOR both `i` and `nums[i]` into `result`.
3. Return `result`.

### Python code
```python
def missingNumber(nums):
    n = len(nums)
    result = n

    for i in range(n):
        result ^= i
        result ^= nums[i]

    return result
```

### Line-by-line explanation
- `result = n` - start by including the index `n` itself, since the loop below only naturally covers indices `0` to `n-1` (there are only `n` elements in `nums`, at indices `0..n-1`, but the *value range* we need to fully cover is `0..n` inclusive) - pre-seeding with `n` accounts for this "extra" index cleanly.
- `for i in range(n): result ^= i; result ^= nums[i]` - XOR in every index from `0` to `n-1`, and every actual value present in `nums`.
- By the end, `result` has XORed together: `{0,1,...,n}` (every possible value in the full range, via the `n` seed plus the loop's `i` values) and `{nums[0], nums[1], ..., nums[n-1]}` (every actual value present). Every number that's present in `nums` appears **twice** in this combined XOR (once as a "range" value, once as an "actual" value) and cancels to 0; the one number missing from `nums` appears only **once** (as a "range" value, with no matching "actual" value to cancel it) - so it survives as the final `result`.

### Dry run
`nums = [3,0,1]`, `n = 3`

`result = 3` (seeded).

| i | result ^= i | result ^= nums[i] |
|---|---|---|
| 0 | `3^0=3` | `nums[0]=3` → `3^3=0` |
| 1 | `0^1=1` | `nums[1]=0` → `1^0=1` |
| 2 | `1^2=3` | `nums[2]=1` → `3^1=2` |

Final: `result = 2` ✅ matches expected output.

**Sanity check via the "pairs cancel" framing:** values XORed in total: range values `{3(seed),0,1,2}` and actual values `{3,0,1}` (from `nums`, XORed in the order encountered). Combined multiset: `0` appears twice (range index 0, and `nums[1]=0`) → cancels. `1` appears twice (range index 1, and `nums[2]=1`) → cancels. `2` appears once (range index 2 only - never appears in `nums`) → survives. `3` appears twice (the seed, and `nums[0]=3`) → cancels. Only `2` survives, matching the result.

### Time & space complexity
- **Time: O(n)**, **Space: O(1)** - no extra data structure at all, genuinely constant extra space.

---

## Approach 4: Sum Formula (Math)

### Intuition
The sum of every integer from `0` to `n` has a well-known closed-form formula: `n * (n+1) / 2` (Gauss's formula). The missing number is simply this **expected total sum** minus the **actual sum** of the numbers present in `nums` - whatever's "left over" is exactly the missing value.

### Python code
```python
def missingNumber(nums):
    n = len(nums)
    expected_sum = n * (n + 1) // 2
    actual_sum = sum(nums)
    return expected_sum - actual_sum
```

### Line-by-line explanation
- `expected_sum = n * (n + 1) // 2` - the sum of every integer from `0` to `n`, via Gauss's formula.
- `actual_sum = sum(nums)` - the sum of what's actually present.
- `expected_sum - actual_sum` - the difference is exactly the one missing value.

### Time & space complexity
- **Time: O(n)** - computing `sum(nums)` requires one pass.
- **Space: O(1)**.

*(Simple and elegant, though worth knowing that for extremely large numbers, this approach could theoretically be more prone to overflow issues in fixed-width-integer languages than the XOR approach - not a practical concern in Python specifically, given its arbitrary-precision integers.)*

---

## Common mistakes & misconceptions

1. **Forgetting to seed `result` with `n` in the XOR approach**, and only XOR-ing indices `0` to `n-1` from the loop. Since the array has `n` elements at indices `0..n-1`, but the *value range* to cover is `0..n` inclusive, omitting the seed silently drops `n` from consideration - producing a wrong answer specifically whenever `n` itself happens to be the missing number, or subtly miscounting otherwise.
2. **Believing the XOR trick requires the array to already be sorted or the missing number to be at a predictable position.** It doesn't - XOR's cancel-out property is entirely order-independent (XOR is commutative and associative), so the array can be in any order and the trick still works identically.
3. **Using the sum formula in a language with fixed-width integer overflow without considering large `n`.** In Python this is a non-issue (arbitrary-precision integers), but it's worth explicitly noting as a real concern in fixed-width languages - `n * (n+1)` can overflow a 32-bit int well before `n` gets particularly large, which the XOR approach avoids entirely since it never computes a large intermediate sum.
4. **Treating all four approaches as strictly equivalent and picking arbitrarily.** They differ meaningfully in space (hash set and sorting approaches use O(n) extra space or mutate/copy the input, while XOR and sum-formula are genuinely O(1) extra space) - worth being able to articulate the trade-off, not just produce a working answer.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Sorting | O(n log n) | O(n) or O(1) | Simplest to reason about, not the fastest. |
| Hash Set | O(n) | O(n) | Straightforward, uses extra space. |
| XOR trick | O(n) | O(1) | Elegant, no arithmetic overflow concerns even in fixed-width languages. |
| Sum formula | O(n) | O(1) | Simplest O(n)/O(1) solution to write, given the direct formula. |

**Key takeaway:** this problem is a nice showcase of how the same task can be approached through multiple different topics' lenses (sorting, hashing, bit manipulation, plain math) - the XOR "cancel out pairs, isolate the odd one out" trick specifically is worth remembering as the standard bit-manipulation answer whenever a problem involves finding a single unmatched or missing value among otherwise-paired/complete data.
