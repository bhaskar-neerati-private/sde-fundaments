# Topic 18: Bit Manipulation

## Core concepts / data structures

### Binary representation
**What it is:** every integer, at the hardware level, is stored as a sequence of **bits** (binary digits, each either 0 or 1). Bit manipulation problems work directly with this binary representation, using special operators to inspect or change individual bits, instead of treating the number as an opaque "quantity" the way arithmetic normally does.

**Simple explanation:** think of an integer as a row of light switches, each either on (1) or off (0), where each switch's position represents a power of 2 (the rightmost switch is worth 1, the next is worth 2, then 4, then 8, and so on). Bit manipulation operations let you flip, check, or combine these switches directly and extremely fast (these are among the cheapest operations a CPU can perform).

### The core bitwise operators
| Operator | Symbol | What it does |
|---|---|---|
| AND | `&` | Result bit is 1 only if **both** input bits are 1. |
| OR | `\|` | Result bit is 1 if **either** input bit is 1. |
| XOR | `^` | Result bit is 1 if the input bits are **different** (exactly one is 1). |
| NOT | `~` | Flips every bit (1 becomes 0, 0 becomes 1). |
| Left shift | `<<` | Shifts all bits left, filling with 0s - equivalent to multiplying by `2^k` for a shift of `k`. |
| Right shift | `>>` | Shifts all bits right - equivalent to (integer) dividing by `2^k`. |

### Useful, commonly memorized bit tricks
- **Check if the lowest bit is set (odd/even check):** `n & 1` - is `1` if `n` is odd, `0` if even.
- **Check if a number is a power of 2:** `n > 0 and (n & (n - 1)) == 0` - see the explanation in the "n & (n-1)" trick below.
- **The `n & (n - 1)` trick - clears the lowest set bit:** subtracting 1 from `n` flips every bit starting from the lowest set `1` bit (turning it to 0) through the end (turning trailing 0s into 1s); ANDing this with the original `n` clears exactly that lowest set bit, leaving everything else unchanged. This single trick is the basis for efficiently counting set bits (see Number of 1 Bits) and for the power-of-2 check above (a power of 2 has exactly one set bit, so clearing it leaves 0).
- **XOR's "cancel out" property:** `x ^ x = 0` and `x ^ 0 = x`, and XOR is commutative/associative - this means XOR-ing a list of numbers where everything appears in pairs **except one** leaves just that unpaired number (all the pairs cancel each other out to 0, leaving only the unique one) - a classic trick for "find the single/missing number" problems.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **`n & (n-1)` to clear the lowest set bit** | Counting set bits, checking powers of 2. |
| **XOR to cancel out pairs, isolate an unpaired/missing value** | Finding a value that appears once among duplicates, or a missing number in a known range. |
| **Shifting to extract or build up bits one at a time** | Reversing bits, converting between representations. |
| **Simulating addition manually with XOR and AND** | Implementing addition without using the `+` operator (Sum of Two Integers) - XOR gives the "sum ignoring carries," AND (shifted left by 1) gives "where a carry is generated," and repeating this until no carry remains reproduces binary addition from first principles. |
| **DP building on smaller already-computed bit counts** | Counting bits for every number in a range efficiently, by relating each number's bit count to a smaller previously-computed one (e.g. `i` and `i >> 1`, or `i` and `i & (i-1)`). |

## Key terminology

- **Bit** - a single binary digit (0 or 1).
- **Set bit** - a bit with value 1.
- **LSB / MSB** - least significant bit (rightmost, worth 1) / most significant bit (leftmost, worth the highest power of 2 for the number's size).
- **Two's complement** - the standard way negative numbers are represented in binary on most systems; important context for understanding shifts and NOT on negative numbers, though most of the problems in this specific list work with non-negative integers where this is less of a direct concern.
- **Overflow / bit-width** - a fixed number of bits (commonly 32) can only represent a limited range of integers; some problems (like Sum of Two Integers in Python specifically) require extra care to correctly simulate fixed-width overflow behavior, since Python's own integers don't naturally overflow the way many other languages' do.

## Common beginner mistakes

1. **Confusing `&` (bitwise AND) with `and` (logical AND)**, or `|` with `or` - these look similar but operate completely differently (bitwise operators work on individual bits of numbers; logical operators work on truthy/falsy values as a whole).
2. **Forgetting operator precedence** - bitwise operators in Python have lower precedence than comparison operators, so `if n & 1 == 1` can silently parse as `if n & (1 == 1)` in some contexts - always use parentheses liberally around bitwise expressions when combined with comparisons: `if (n & 1) == 1`.
3. **Not accounting for Python's arbitrary-precision integers when simulating fixed-width behavior** - Python integers can grow arbitrarily large and don't naturally "wrap around" the way a 32-bit integer would in many other languages, which specifically causes issues for problems like Sum of Two Integers that rely on realistic overflow behavior - this typically requires explicit masking (`& 0xFFFFFFFF`) and sign-correction logic.
4. **Off-by-one in shift amounts**, especially when building up a result bit by bit (e.g. reversing bits) - forgetting exactly which position a given bit should end up in after 32 iterations.

## How this compares to other topics

Bit manipulation problems are less about a shared algorithmic strategy (like DP's memoization or Graphs' traversal) and more about a specific, low-level toolkit of operator tricks - many of which are worth simply memorizing as named tricks (like `n & (n-1)`), the same way you'd memorize a useful math identity, since deriving them from scratch under interview pressure is much harder than recognizing "this is the power-of-2 check" or "this is the missing-number XOR trick" on sight.

## Starter problems (easy, to warm up)

1. **Number of 1 Bits** (LeetCode #191) - in your Blind 75 list; the cleanest introduction to the `n & (n-1)` trick.
2. **Single Number** (LeetCode #136) - not in Blind 75 (though a close cousin, Missing Number, is), but the purest possible introduction to XOR's cancel-out property.

## What carries over from here

There isn't a "next topic" this specifically feeds into within this curriculum, but the general comfort with binary representation reinforces understanding of how integers, hashing, and even some Dynamic Programming state-compression tricks (representing a set of "used" items as bits of a single integer, a technique sometimes called "bitmask DP," common in harder problems beyond this list) work under the hood.
