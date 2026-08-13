# 71. Number of 1 Bits

**LeetCode:** [#191 - Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) · **Topic:** [Bit Manipulation](../topics/18-bit-manipulation.md) · **Difficulty:** Easy

## Problem statement

Given an unsigned integer `n`, return the number of `1` bits it has (also known as its **Hamming weight**).

**Example:**
```
Input: n = 11 (binary: 1011)
Output: 3
```

## Applicable approaches

- **Check All 32 Bit Positions One by One.**
- **Optimal - `n & (n-1)` Trick (Clear the Lowest Set Bit).** The standard, expected, more efficient solution.

## Approach 1: Check All 32 Bit Positions

### Intuition
Walk through every one of a 32-bit integer's positions, and for each, check whether that specific bit is set (1), using a right-shift and a check against the lowest bit.

### Python code
```python
def hammingWeight(n):
    count = 0
    for i in range(32):
        if (n >> i) & 1:
            count += 1
    return count
```

### Line-by-line explanation
- `for i in range(32):` - check every one of the 32 bit positions (since the problem specifies a 32-bit unsigned integer).
- `(n >> i) & 1` - shift `n` right by `i` positions (moving the bit we care about down to the lowest position), then `& 1` isolates just that lowest bit, giving `1` if it was set, `0` otherwise.
- `count += 1` - tally up every set bit found.

### Time & space complexity
- **Time: O(32) = O(1)** (technically bounded by the fixed bit-width, so constant time for a fixed-size integer) - though often described as O(k) where k is the number of bits, which matters if generalizing beyond a fixed 32-bit width.
- **Space: O(1)**.

*(Correct, and technically already O(1) for a fixed bit-width, but always does exactly 32 iterations regardless of how many bits are actually set - the optimal approach below does work proportional to the number of set bits instead.)*

---

## Approach 2: Optimal - `n & (n-1)` Trick

### Intuition
Recall the general bit trick: **`n & (n - 1)` clears the lowest set bit of `n`**, leaving every other bit unchanged. Why: subtracting 1 from `n` flips the lowest set bit to 0, and flips every bit *after* it (which were all 0s) to 1s; ANDing this modified value back with the original `n` keeps only the bits that were 1 in **both** - which is every original bit **except** that lowest set one (since it's now 0 in the subtracted version). So, repeatedly applying `n = n & (n-1)` removes exactly one set bit each time - **counting how many times we can do this before `n` becomes 0 directly gives the total count of set bits.**

### Algorithm
1. Initialize `count = 0`.
2. While `n != 0`: apply `n = n & (n - 1)` (clearing the lowest set bit), and increment `count`.
3. Return `count`.

### Python code
```python
def hammingWeight(n):
    count = 0
    while n:
        n &= (n - 1)
        count += 1
    return count
```

### Line-by-line explanation
- `count = 0` - tracks how many set bits we've cleared so far.
- `while n:` - keep going as long as `n` still has at least one set bit (`n != 0` is truthy in Python; `n == 0` is falsy).
- `n &= (n - 1)` - clears exactly the lowest currently-set bit (shorthand for `n = n & (n - 1)`).
- `count += 1` - one more set bit accounted for.
- `return count` - once `n` reaches 0 (every set bit has been cleared), `count` holds the total.

### Dry run
`n = 11` (binary `1011`)

| n (binary) | n - 1 (binary) | n & (n-1) (binary) | count after |
|---|---|---|---|
| 1011 | 1010 | 1010 | 1 |
| 1010 | 1001 | 1000 | 2 |
| 1000 | 0111 | 0000 | 3 |

`n` is now `0`, loop ends. `count = 3` ✅ matches expected output (binary `1011` has three `1` bits).

**Verifying the trick step by step for the first iteration:** `n=1011` (11 in decimal). `n-1 = 1010` (10 in decimal) - subtracting 1 flipped the lowest set bit (the rightmost `1`) to `0`, and there were no trailing zeros after it to flip to 1 in this particular case. `n & (n-1) = 1011 & 1010 = 1010` - the lowest set bit (position 0) has been cleared, while the rest (`101` in the higher positions) remains unchanged.

### Time & space complexity
- **Time: O(k)** where k = the number of set bits in `n` (not the total bit-width) - since each iteration clears exactly one set bit, this only does as many iterations as there are 1s, which is at most 32 but often much fewer.
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Confusing `&` (bitwise AND) with `and` (logical AND).** `(n >> i) & 1` relies on the bitwise operator to isolate a single bit; accidentally writing `and` would instead evaluate truthiness of the two operands as whole values, giving a completely different (and almost always wrong) result.
2. **Forgetting operator precedence when combining bitwise operations with comparisons**, e.g. writing `if n & 1 == 1` without parentheses - Python evaluates `==` before `&` in this context due to precedence rules, silently parsing as `n & (1 == 1)`, i.e. `n & True`, which is not the intended check. Always parenthesize explicitly: `if (n & 1) == 1`.
3. **Believing the `n & (n-1)` trick "skips checking some bits" and might miss a set bit.** It doesn't skip anything - each application removes exactly the *lowest currently-set* bit, and since we loop until `n` is entirely 0, every set bit is guaranteed to be found and counted exactly once, just not in a fixed-position order like the 32-iteration approach.
4. **Assuming the two approaches always take the same amount of work.** They don't - approach 1 always takes exactly 32 iterations regardless of input, while approach 2's iteration count equals the number of set bits (which could be anywhere from 0 to 32) - this distinction matters when comparing average-case performance across many calls with typically-sparse bit patterns.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Check all 32 positions | O(32) = O(1), but always 32 iterations | O(1) | Simple, always does fixed work regardless of actual bit count. |
| `n & (n-1)` trick | O(k), k = number of set bits | O(1) | The standard, more elegant, expected solution - work proportional to the actual answer. |

**Key takeaway:** the `n & (n-1)` trick (clearing the lowest set bit) is one of the most fundamental, frequently-reused bit manipulation tricks - beyond counting bits, it's also the basis for a clean power-of-2 check (`n > 0 and n & (n-1) == 0`, since a power of 2 has exactly one set bit, and clearing it leaves exactly 0), worth having memorized cold.
