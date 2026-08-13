# 73. Reverse Bits

**LeetCode:** [#190 - Reverse Bits](https://leetcode.com/problems/reverse-bits/) · **Topic:** [Bit Manipulation](../topics/18-bit-manipulation.md) · **Difficulty:** Easy

## Problem statement

Reverse the bits of a given 32-bit unsigned integer.

**Example:**
```
Input: n = 00000010100101000001111010011100 (binary)
Output:  00111001011110000010100101000000 (binary)
```

## Applicable approaches

- **Brute Force String Reversal** - convert to a binary string, reverse it, convert back.
- **Optimal - Bit Manipulation (Extract and Rebuild).** The standard, expected solution.
- **Bonus - Divide and Conquer (bit-swapping in chunks).** A clever, more advanced alternative worth knowing exists.

## Approach 1: Brute Force - String Reversal

### Intuition
Convert the number to its 32-character binary string representation, reverse that string, then convert it back to an integer.

### Python code
```python
def reverseBits(n):
    binary_str = format(n, '032b')  # pad to exactly 32 bits
    reversed_str = binary_str[::-1]
    return int(reversed_str, 2)
```

### Line-by-line explanation
- `format(n, '032b')` - converts `n` to a binary string, zero-padded to exactly 32 characters (essential, since a number like `5` would otherwise only produce `"101"`, losing the leading zeros that matter for a fixed-width reversal).
- `binary_str[::-1]` - reverses the string.
- `int(reversed_str, 2)` - parses the reversed binary string back into an integer (base 2).

### Time & space complexity
- **Time: O(32) = O(1)** for a fixed-width integer (or O(k) generally, k = bit width).
- **Space: O(32) = O(1)** for the string representations.

*(Simple and correct, but relies on string conversion utilities rather than demonstrating direct bit manipulation - the approach below is what's typically expected to show bitwise fluency.)*

---

## Approach 2: Optimal - Bit Manipulation (Extract and Rebuild)

### Intuition
Build the result one bit at a time: repeatedly take the **lowest** bit of `n`, and place it into the **highest** available position of the result (starting from the very top and working down) - directly simulating the "reversal" using shifts.

### Algorithm
1. Initialize `result = 0`.
2. For each of the 32 bit positions:
   - Shift `result` left by 1 (making room for a new bit at the bottom).
   - Extract `n`'s current lowest bit (`n & 1`), and OR it into `result`'s now-empty lowest position.
   - Shift `n` right by 1 (to expose the next bit for the next iteration).
3. Return `result`.

### Python code
```python
def reverseBits(n):
    result = 0
    for _ in range(32):
        result = (result << 1) | (n & 1)
        n >>= 1
    return result
```

### Line-by-line explanation
- `result = 0` - starts empty; will be built up bit by bit.
- `for _ in range(32):` - process exactly 32 bit positions (the fixed width).
- `result = (result << 1) | (n & 1)` - **shift `result` left by 1** (every bit already placed moves up one position, making room at the bottom), **then OR in** `n`'s current lowest bit (`n & 1` is `0` or `1`) into that newly-opened bottom position.
- `n >>= 1` - shift `n` right by 1, so the *next* bit we care about (originally the second-lowest, now the new lowest after this shift) is ready to be extracted on the next iteration.
- After all 32 iterations, `result` holds every one of `n`'s original bits, in reverse order (the original lowest bit ended up shifted all the way to the top, having been placed first and then pushed upward by every subsequent iteration's left-shift; the original highest bit was placed last, ending up at the very bottom, unshifted after being placed).

### Dry run (small example, using a 4-bit width for simplicity)
`n = 0b1011` (imagine we're reversing just 4 bits: `1,0,1,1`)

| iteration | n & 1 | result before | result = (result<<1)\|(n&1) | n after (n>>=1) |
|---|---|---|---|---|
| 1 | `1011 & 1 = 1` | `0000` | `(0000<<1)\|1 = 0001` | `0101` |
| 2 | `0101 & 1 = 1` | `0001` | `(0001<<1)\|1 = 0011` | `0010` |
| 3 | `0010 & 1 = 0` | `0011` | `(0011<<1)\|0 = 0110` | `0001` |
| 4 | `0001 & 1 = 1` | `0110` | `(0110<<1)\|1 = 1101` | `0000` |

Final `result = 1101`. Original `n = 1011` reversed is indeed `1101` ✅ (reading `1011` backward gives `1101`).

### Time & space complexity
- **Time: O(32) = O(1)** for a fixed 32-bit width.
- **Space: O(1)**.

---

## Bonus: Divide and Conquer (Bit-Swapping in Chunks)

### Intuition
A cleverer approach: swap bits in progressively larger groups - first swap every adjacent pair of individual bits, then swap every adjacent pair of 2-bit groups, then 4-bit groups, then 8-bit groups, then finally the two 16-bit halves. Each stage doubles the group size, and after `log2(32) = 5` stages, the entire 32-bit number is fully reversed. This uses precomputed bit masks and is more of a "clever trick to know exists" than something to derive from scratch, but it demonstrates that O(log n) (in terms of number of *operations*, not bits) reversal is possible via a divide-and-conquer bit-shuffling strategy.

### Time & space complexity
- **Time: O(log 32) = O(1)** operations (5 fixed stages, each O(1) work) - technically fewer discrete operations than the 32-iteration loop, though both are O(1) for a fixed bit-width in practice.
- **Space: O(1)**.

---

## Common mistakes & misconceptions

1. **Forgetting to zero-pad the binary string to exactly 32 characters in the string-reversal approach.** Python's `bin(n)` (without explicit width formatting) drops leading zeros - reversing an unpadded string would incorrectly treat the number as if it had fewer than 32 bits, misplacing every bit in the result.
2. **Getting the shift directions backward** - `result << 1` (making room at the bottom for a new bit) versus `n >> 1` (exposing the next bit to extract) serve two different, non-interchangeable purposes; swapping them would neither correctly build the result nor correctly consume the input.
3. **Running fewer or more than exactly 32 iterations.** Since this problem operates on a fixed-width 32-bit integer, running too few iterations would drop high-order bits of the reversed result, and running too many would incorrectly try to extract bits from `n` that have already been fully shifted out to 0 (harmless in this specific algorithm since `n & 1` on an exhausted `n` just contributes 0s, but conceptually worth being precise about).
4. **Assuming this same bit-by-bit extraction template generalizes trivially to signed integers without extra care.** This problem is specified as unsigned; reversing bits of a *signed* representation would require extra care about how the sign bit's position and meaning changes after reversal - a genuinely different concern from what's handled here.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| String reversal | O(1) (fixed width) | O(1) | Simple, leans on string conversion utilities. |
| Bit manipulation (extract & rebuild) | O(1) (32 iterations) | O(1) | The standard, expected solution demonstrating direct bit fluency. |
| Divide and conquer (chunk swapping) | O(1) (5 stages) | O(1) | A more advanced, clever alternative - good to know exists. |

**Key takeaway:** the "shift result left, OR in the next extracted bit, shift source right" loop is the standard template for building up any bit-by-bit transformation of a fixed-width integer - the same shape (extract from one end, place into the other, repeat) reappears in various bit manipulation and low-level encoding problems beyond this specific one.
