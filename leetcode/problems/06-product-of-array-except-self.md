# 6. Product of Array Except Self

**LeetCode:** [#238 - Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Medium

## Problem statement

Given an integer array `nums`, return an array `answer` such that `answer[i]` equals the product of every element of `nums` **except** `nums[i]`.

- You must write an algorithm that runs in O(n) time.
- You cannot use the division operator.

**Example:**
```
Input: nums = [1,2,3,4]
Output: [24,12,8,6]
Explanation: answer[0] = 2*3*4=24, answer[1] = 1*3*4=12, answer[2] = 1*2*4=8, answer[3] = 1*2*3=6
```

## Applicable approaches

- **Brute Force** — for each index, multiply all other elements.
- **Tempting but disallowed — Division** — compute the total product, then divide by `nums[i]` for each index.
- **Optimal — Prefix and Suffix Products** — the standard, expected O(n) solution with no division.

**Why hashing doesn't apply here:** this is the one Arrays & Hashing-topic problem that isn't really about hashing at all — there's no "have I seen this value" question and no grouping. It belongs to this topic because it's fundamentally an array/prefix-computation problem, and it's grouped here because the prefix/suffix technique is a close cousin of the running-total ideas that also show up in hashing-adjacent problems. Don't force a hash map into this solution; it doesn't help.

## Approach 1: Brute Force

### Intuition

The most literal reading: for each position, directly compute the product of everything except that position by looping through the array again and skipping the current index. This is correct and simple, but notice it recomputes an almost-identical product from scratch for every single output position — most of that work is duplicated across positions, which is exactly what the optimal approach eliminates.

### Algorithm

1. For each index `i`: loop through the whole array, multiplying together every element where the loop index `j != i`.
2. Store that product as `answer[i]`.

### Python code
```python
def productExceptSelf(nums):
    n = len(nums)
    answer = [1] * n
    for i in range(n):
        product = 1
        for j in range(n):
            if j != i:
                product *= nums[j]
        answer[i] = product
    return answer
```

### Line-by-line explanation

- `answer = [1] * n` — result array, initialized to 1s (will be overwritten). Using `1` as the starting value matters: multiplying by 1 is the identity operation, so it never distorts the running product before real values are multiplied in.
- Outer loop `for i in range(n)` — one iteration per output position.
- `product = 1` — reset the running product for this position; **this reset is why the algorithm is O(n²)** — every outer iteration restarts from scratch instead of reusing any work from the previous iteration, even though consecutive positions' "skip one element" products overlap heavily.
- Inner loop `for j in range(n): if j != i: product *= nums[j]` — multiply everything except the current index.
- `answer[i] = product` — store the result for this position.

### Dry run

`nums = [1,2,3,4]`, computing `answer[1]` (skip index 1, value 2):
`product = 1*3*4 = 12` (skipping `nums[1]=2`). Matches expected `answer[1] = 12`. Note this required 3 multiplications; computing `answer[0]` (skip index 0) required a *different* 3 multiplications (`2*3*4`), even though both computations share the sub-product `3*4` — that shared work is thrown away and redone, which is the specific waste the next approach targets.

### Time & space complexity

- **Time: O(n²)** — an inner O(n) loop for every one of the n output positions, precisely because each position's computation starts over rather than reusing overlapping sub-products.
- **Space: O(1)** extra, not counting the output array (which the problem doesn't count against space complexity, since it's required output).

---

## Approach 2 (Tempting but Disallowed): Division

### Intuition

The obvious shortcut: compute `total = product of all elements` once, then `answer[i] = total / nums[i]` — this is O(n) and looks like a free win. **It's explicitly forbidden by the problem, and it's worth understanding exactly why it's fragile even if it weren't forbidden**: it breaks the moment **any element is 0**. If exactly one element is `0`, dividing by it crashes. If **two or more** elements are `0`, every `answer[i]` should correctly be `0` (since removing any single element still leaves at least one zero in the product) — but the division approach can't recover this correctly from a single `total` value, because `total` itself is already `0`, and `0 / 0` is undefined. This isn't a minor edge case to patch around; it's a structural flaw in using division for a problem whose real content is about which sub-products exist, not a single global product.

---

## Approach 3: Optimal — Prefix and Suffix Products

### Intuition

The brute force's redundant work — recomputing overlapping products from scratch — points directly at the fix: **`answer[i]` is exactly (product of everything to the left of i) × (product of everything to the right of i)**. Those two pieces, "everything left of i" and "everything right of i," are exactly the kind of running totals that can each be built in a single O(n) pass, reused across every position instead of recomputed per position. If we precompute a **prefix product** array (product of everything strictly left of each index) and a **suffix product** array (product of everything strictly right of each index), then `answer[i]` becomes a single multiplication: `prefix[i] * suffix[i]`. No division needed — the "exclude `nums[i]`" logic is baked directly into *how* prefix and suffix are defined (strictly left, strictly right), rather than removed after the fact via division.

We can even build these prefix/suffix products **directly into the answer array itself**, using it first to hold prefix products, then multiplying in the suffix products in a second pass — keeping extra space down to O(1) (not counting the output array).

### Algorithm

1. Create `answer` array of size n, all 1s.
2. **Left-to-right pass:** walk through `nums`, keeping a running `prefix` product of everything seen *before* the current index. Set `answer[i] = prefix` (product of everything to the left of i), then update `prefix *= nums[i]` for the next iteration.
3. **Right-to-left pass:** walk through `nums` backwards, keeping a running `suffix` product of everything seen *after* the current index. Multiply it into the existing `answer[i]`: `answer[i] *= suffix`, then update `suffix *= nums[i]`.
4. Return `answer`.

### Python code
```python
def productExceptSelf(nums):
    n = len(nums)
    answer = [1] * n

    prefix = 1
    for i in range(n):
        answer[i] = prefix
        prefix *= nums[i]

    suffix = 1
    for i in range(n - 1, -1, -1):
        answer[i] *= suffix
        suffix *= nums[i]

    return answer
```

### Line-by-line explanation

- `answer = [1] * n` — starts as all 1s (the identity value for multiplication — multiplying by 1 changes nothing yet, so this is safe to build on).
- **First loop (left to right):**
  - `prefix = 1` — nothing is to the left of index 0 yet, so the "product of everything to the left" starts at 1 (an empty product is conventionally 1, the same way an empty sum is conventionally 0).
  - `answer[i] = prefix` — **this line runs *before* updating `prefix`**, which is the entire trick: at this point, `prefix` holds the product of `nums[0..i-1]` (everything strictly before `i`), not including `nums[i]` itself — exactly the "exclude self" property we need, achieved by ordering rather than by division.
  - `prefix *= nums[i]` — now include `nums[i]` in the running product, ready for the *next* index's prefix.
- **Second loop (right to left):**
  - `suffix = 1` — nothing is to the right of the last index yet, same "empty product is 1" convention.
  - `answer[i] *= suffix` — multiply in the product of everything to the *right* of `i`. At this point `answer[i]` already held the left-side product from the first loop, so now it holds left × right = the full "everything except self" product — this line is where the two passes' work combines.
  - `suffix *= nums[i]` — include `nums[i]` for the next (leftward) index's suffix.

### Dry run

`nums = [1, 2, 3, 4]`, n = 4

**Left-to-right pass** (`prefix` starts at 1):
| i | answer[i] = prefix (before update) | prefix after (*= nums[i]) |
|---|---|---|
| 0 | 1 | 1 * 1 = 1 |
| 1 | 1 | 1 * 2 = 2 |
| 2 | 2 | 2 * 3 = 6 |
| 3 | 6 | 6 * 4 = 24 |

After this pass: `answer = [1, 1, 2, 6]` (each holds the product of everything strictly to its left).

**Right-to-left pass** (`suffix` starts at 1):
| i | answer[i] *= suffix | suffix after (*= nums[i]) |
|---|---|---|
| 3 | 6 * 1 = 6 | 1 * 4 = 4 |
| 2 | 2 * 4 = 8 | 4 * 3 = 12 |
| 1 | 1 * 12 = 12 | 12 * 2 = 24 |
| 0 | 1 * 24 = 24 | 24 * 1 = 24 |

Final `answer = [24, 12, 8, 6]` ✅ matches expected output exactly. Trace `answer[2] = 8` specifically: left product of `nums[0..1] = 1*2 = 2`; right product of `nums[3] = 4`; `2 * 4 = 8` ✅ — exactly the definition, achieved without ever computing a full product or dividing.

### Time & space complexity

- **Time: O(n)** — exactly two linear passes over the array; no repeated work across positions, unlike the brute force's O(n²), because each pass builds a running total incrementally rather than recomputing from scratch.
- **Space: O(1) extra** — not counting the output array, we only ever use two running variables (`prefix`, `suffix`). (If the output array is counted, it's O(n), but that's unavoidable since we must return n values.)

---

## Common mistakes & misconceptions

1. **Using division "just this once" because it's simpler.** As explained above, this isn't a minor style choice — it's a genuine correctness bug on any input containing a zero, and the problem explicitly disallows it regardless.
2. **Computing `answer[i] = prefix` *after* updating `prefix`, instead of before.** Swapping the order includes `nums[i]` in its own prefix, which computes "product including self" rather than "excluding self" — a subtle off-by-one that only shows up as wrong output, not a crash, making it easy to miss without a careful dry run.
3. **Assuming the suffix pass needs its own separate output array.** Because the suffix pass *multiplies into* the existing `answer[i]` (rather than overwriting it), it correctly combines with the first pass's result — allocating a second array here would work but wastes the O(1)-extra-space property that makes this the fully optimal solution.
4. **Forgetting that "empty product = 1" is the correct base case, not 0.** Initializing `prefix`/`suffix` to `0` instead of `1` would zero out every subsequent product immediately, since anything multiplied by 0 stays 0 — this mirrors (and is the multiplicative analog of) initializing a running sum to 0 instead of some non-identity value.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) extra | Too slow — recomputes overlapping products from scratch. |
| Division (not allowed) | O(n) | O(1) extra | Simple but breaks on zeros; forbidden by the problem. |
| Prefix + Suffix Products | O(n) | O(1) extra | The correct, expected optimal solution. |

**Key takeaway:** when a value at position `i` depends on "everything except position i," look for a way to split the computation into "everything to the left" and "everything to the right," each buildable with a single linear pass that reuses work across positions — this prefix/suffix pattern generalizes far beyond products, to sums, running max/min, and more.
