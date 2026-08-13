# 74. Sum of Two Integers

**LeetCode:** [#371 - Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) · **Topic:** [Bit Manipulation](../topics/18-bit-manipulation.md) · **Difficulty:** Medium

## Problem statement

Given two integers `a` and `b`, return their sum **without using the `+` or `-` operators**.

**Example:**
```
Input: a = 1, b = 2
Output: 3
```

## Applicable approaches

- **Naive (Not Allowed) - Just Use `+`.** Trivially disallowed by the problem, but useful to state why we need an alternative.
- **Optimal - Simulate Addition with XOR and AND (Bitwise), Handling Python's Special Overflow Behavior.** The standard, expected solution.

## Approach: Simulate Binary Addition with XOR and AND

### Intuition
Think about how addition works on paper, in binary, one bit position at a time: at each position, you add the two bits **plus any carry** from the previous position, writing down the result bit and generating a new carry if needed. Two specific bitwise operations directly capture this:
- **XOR (`^`)** gives the sum of two bits **ignoring any carry** (`0^0=0, 1^0=1, 0^1=1, 1^1=0` - notice `1^1=0`, which is exactly "1+1=0, carry the 1" with the carry dropped).
- **AND (`&`)** gives exactly the positions where **both** bits are 1 - precisely where a carry would be generated. Shifting this left by 1 (`(a & b) << 1`) moves that carry into the correct next position to be added in.

So: `a + b` (without a carry-forward algorithm built in) can be simulated as: **repeatedly** compute `xor_part = a ^ b` (sum ignoring carries) and `carry_part = (a & b) << 1` (the carries that need to be added in), then set `a = xor_part` and `b = carry_part`, and **repeat** until there's no carry left (`b == 0`) - at which point `a` holds the final sum. This mirrors exactly how you'd do long addition by hand, propagating carries one step at a time, except carries can cascade through multiple positions per "round" here rather than one row at a time.

### Algorithm (conceptual, before Python-specific overflow handling)
1. While `b != 0`:
   - `carry = (a & b) << 1`
   - `a = a ^ b`
   - `b = carry`
2. Return `a`.

### Why Python needs extra care (unlike languages with fixed-width integers)
In languages like Java or C++, integers are a fixed width (commonly 32 bits), and arithmetic naturally "wraps around" (overflows) at that boundary - which this algorithm relies on implicitly to correctly terminate and handle negative numbers via two's complement representation. **Python integers don't naturally overflow** - they grow arbitrarily large - so a direct translation of the algorithm above can loop forever or produce incorrect results for negative inputs, since there's no natural 32-bit "wraparound" happening. The fix: explicitly **mask** intermediate results to 32 bits, and manually convert the final result back to a properly signed value if it represents a negative number in 32-bit two's complement.

### Python code
```python
def getSum(a, b):
    mask = 0xFFFFFFFF  # 32 bits of all 1s

    while b != 0:
        carry = ((a & b) << 1) & mask
        a = (a ^ b) & mask
        b = carry

    # if a's highest bit (bit 31) is set, it represents a negative number
    # in 32-bit two's complement - convert it to Python's native negative representation
    if a > 0x7FFFFFFF:
        return ~(a ^ mask)

    return a
```

### Line-by-line explanation
- `mask = 0xFFFFFFFF` - a 32-bit mask (32 ones in binary), used to force every intermediate value to stay within 32 bits, simulating fixed-width overflow behavior.
- `while b != 0:` - keep propagating carries until there's nothing left to carry.
- `carry = ((a & b) << 1) & mask` - compute where carries are generated (`a & b`), shift them into position (`<< 1`), and mask to 32 bits (preventing the shift from letting a carry escape beyond the 32-bit window, matching real fixed-width overflow behavior).
- `a = (a ^ b) & mask` - compute the sum-ignoring-carries, masked to 32 bits.
- `b = carry` - the next round's "b" is whatever still needs to be carried in.
- `if a > 0x7FFFFFFF:` - `0x7FFFFFFF` is the largest positive 32-bit two's-complement number (`2^31 - 1`); if our masked result exceeds this, it actually represents a **negative** number in 32-bit two's complement, but Python is currently interpreting the raw bits as an unsigned positive value - we need to convert it correctly.
- `return ~(a ^ mask)` - this expression correctly converts the 32-bit two's-complement bit pattern (currently stored as a large positive Python integer) into the equivalent *actual* negative Python integer. (This specific conversion trick relies on how two's complement and Python's arbitrary-precision `~` operator interact - it's a standard, worth-recognizing idiom for this exact "convert a masked 32-bit pattern back to a signed Python int" situation, rather than something to necessarily re-derive from scratch every time.)
- `return a` - if the result fits within the positive range, return it directly.

### Dry run
`a = 1, b = 2` (binary: `a=01`, `b=10`)

- iteration 1: `carry = (1 & 2) << 1 = (01 & 10) << 1 = 00 << 1 = 0`. `a = 1 ^ 2 = 01 ^ 10 = 11 = 3`. `b = 0`.
- loop condition `b != 0`? No → loop ends.

`a = 3`. `3 > 0x7FFFFFFF`? No → return `3` ✅.

**A dry run with a real carry:** `a = 3, b = 5` (binary `011`, `101`), expected sum `8`.

- iter 1: `carry = (3 & 5) << 1 = (011 & 101) << 1 = 001 << 1 = 010 = 2`. `a = 3^5 = 011^101 = 110 = 6`. `b = 2`.
- iter 2: `carry = (6 & 2) << 1 = (110 & 010) << 1 = 010 << 1 = 100 = 4`. `a = 6^2 = 110^010 = 100 = 4`. `b = 4`.
- iter 3: `carry = (4 & 4) << 1 = 100 << 1 = 1000 = 8`. `a = 4^4 = 0`. `b = 8`.
- iter 4: `carry = (0 & 8) << 1 = 0`. `a = 0^8 = 8`. `b = 0`.
- loop ends (`b=0`). `a = 8` ✅ matches expected sum.

### Time & space complexity
- **Time: O(32) = O(1)** - the carry can cascade through at most 32 bit positions before terminating (fixed bit-width).
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Not masking intermediate results to 32 bits, and running into infinite loops or wrong results on negative inputs.** This is the single most important, Python-specific pitfall for this problem - since Python integers don't naturally overflow, omitting the `& mask` steps can cause `carry` to grow without bound (for negative inputs, which have infinite leading 1s in two's complement) instead of correctly terminating.
2. **Forgetting the final sign-correction step (`if a > 0x7FFFFFFF`) for results that should be negative.** Without it, a masked bit pattern that represents a negative 32-bit number gets returned as a large *positive* Python integer instead, since Python has no built-in notion of "this bit pattern should be interpreted as negative."
3. **Assuming XOR alone (without the AND-based carry) is sufficient to compute a sum.** XOR alone gives the sum *ignoring all carries* (equivalent to addition without carrying, which is actually the same as bitwise addition-without-carry, not real addition) - the AND-and-shift step is what makes this genuinely equivalent to real binary addition, not an optional refinement.
4. **Believing this algorithm's use of a `while` loop makes it slow.** Because a 32-bit carry chain can cascade through at most 32 positions before terminating, the loop is bounded by a small constant regardless of the actual input values - it's genuinely O(1) for fixed-width integers, not a hidden O(n) cost scaling with the numbers' magnitude.

## Summary

| Approach | Notes |
|---|---|
| Direct `+` | Trivially correct, but explicitly disallowed by the problem. |
| XOR/AND carry simulation with 32-bit masking | The standard, expected solution - the masking and final sign-conversion steps are specifically needed in Python due to its arbitrary-precision integers. |

**Key takeaway:** addition itself, at the hardware level, is exactly "XOR for the sum, AND-shifted-left for the carry, repeat until no carry remains" - this problem is really testing whether you understand what the `+` operator is *actually doing* underneath, and Python's lack of natural integer overflow (unlike most other languages) adds a genuinely extra layer of masking/sign-correction work that wouldn't be needed in a fixed-width-integer language.
