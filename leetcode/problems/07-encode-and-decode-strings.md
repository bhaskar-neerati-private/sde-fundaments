# 7. Encode and Decode Strings

**LeetCode:** [#271 - Encode and Decode Strings](https://leetcode.com/problems/encode-and-decode-strings/) (Premium - problem statement below, article/solution write-ups are free) · **Topic:** [Arrays & Hashing](../topics/01-arrays-hashing.md) · **Difficulty:** Medium

## Problem statement

Design an algorithm to **encode** a list of strings into a single string, and **decode** that single string back into the original list of strings.

The tricky part: strings in the list can contain *any* characters, including things like commas, spaces, or even the delimiter characters you might naturally want to use to separate them. Your encode/decode pair must work correctly no matter what characters appear inside the strings.

**Example:**
```
Input: ["neet","code","love","you"]
Encoded: (some single string produced by your encode function)
Decoded: ["neet","code","love","you"]  (must exactly match the input)
```

## Applicable approaches

- **Naive — Simple Delimiter (join with a character like comma)** — broken, but essential to understand *why* it's broken before the fix makes sense.
- **Optimal — Length-Prefixed Encoding** — the standard, expected solution.

**Why this problem isn't really about data structures or time complexity:** unlike most problems in this topic, the interesting content here isn't "which algorithm is asymptotically faster" — both the naive and correct approaches are O(n). The real content is a **design/correctness** question: can your scheme be fed adversarial input (a string containing your own delimiter) and still round-trip correctly? This is worth calling out explicitly, because the standards you'd apply to a performance problem (compare Big-O) don't capture what actually matters here.

## Approach 1: Naive — Simple Delimiter (and why it's broken)

### Intuition

The first instinct: join the strings with a separator character, like a comma — `",".join(strs)` — and split on that same character to decode. This "feels" correct because it works on friendly test cases.

### Why it breaks

The scheme relies on an assumption: that the delimiter character never appears *inside* any of the actual strings. The problem statement explicitly tells you this assumption can't be trusted — strings can contain *any* character. If any string in the list contains the delimiter itself, decoding becomes genuinely ambiguous, not just awkward. For example, encoding `["hello,world", "foo"]` with a comma delimiter produces `"hello,world,foo"` — and decoding that by splitting on commas gives `["hello", "world", "foo"]`, which is **wrong**: we've lost the information about where the original strings actually started and ended, and there's no way to recover it after the fact, because the encoded string itself no longer contains that information.

```python
def encode(strs):
    return ",".join(strs)  # BROKEN if any string contains a comma

def decode(s):
    return s.split(",")    # BROKEN for the same reason
```

**There's no "safer" delimiter character that fixes this** — the problem guarantees strings can contain *any* character, so no single character is ever guaranteed absent. This illustrates a lesson worth generalizing beyond this problem: **a solution that passes the example test cases isn't necessarily correct** — you have to explicitly think about adversarial inputs (here: a string containing whatever separator you chose), not just typical ones.

---

## Approach 2: Optimal — Length-Prefixed Encoding

### Intuition

The delimiter approach fails because it tries to mark *where a string ends* using a character, and that character can be confused with real content. The fix reframes the problem entirely: instead of marking an *end*, record the **length** of each string *before* the string itself. A number is never ambiguous with string content in the way a delimiter character is, because once you know a length, you read *exactly that many characters* next, no matter what they are — you're never searching for a boundary character that could coincidentally also appear inside the data. This is a general, reusable idea: **prefix data with its length instead of trying to mark its end with a special character** — it's exactly how real-world formats like HTTP's `Content-Length` header work, for the same reason.

### Algorithm

**Encode:**
1. For each string `s` in the list, output: `len(s)` followed by a delimiter character (e.g. `#`) followed by `s` itself.
2. Concatenate all of these together into one final string.

**Decode:**
1. Walk through the encoded string with a pointer `i`, starting at 0.
2. At each step, read digits starting at `i` until hitting the delimiter `#` — that gives you the length of the next string.
3. Move the pointer past the `#`, then read exactly `length` characters — that's the next original string.
4. Move the pointer past those characters, and repeat from step 2 until the whole encoded string has been consumed.

### Python code
```python
def encode(strs):
    encoded = ""
    for s in strs:
        encoded += str(len(s)) + "#" + s
    return encoded

def decode(s):
    result = []
    i = 0
    while i < len(s):
        # find the delimiter '#' to know where the length-number ends
        j = i
        while s[j] != "#":
            j += 1
        length = int(s[i:j])

        # the actual string starts right after the '#' and runs for `length` characters
        start = j + 1
        result.append(s[start : start + length])

        # move i past this whole chunk (length digits + '#' + the string itself)
        i = start + length

    return result
```

### Line-by-line explanation (encode)

- `encoded = ""` — the final output string, built up piece by piece.
- `for s in strs: encoded += str(len(s)) + "#" + s` — for each string, append its length (as digits), a `#` delimiter, then the string's actual content, all glued together with no separator needed *between* chunks, because each chunk is fully self-describing (it announces its own length before its content).

### Line-by-line explanation (decode)

- `result = []` — the list of decoded strings we're rebuilding.
- `i = 0` — a pointer marking where the *next* length-prefixed chunk starts in the encoded string.
- `j = i; while s[j] != "#": j += 1` — scan forward from `i` until we find the `#` character — everything between `i` and `j` is the length number. **This is safe specifically because a `#` will always appear here by construction** (every chunk we wrote in `encode` includes exactly one `#` right after its length digits), and digits themselves never equal `#`, so this scan can't accidentally terminate early.
- `length = int(s[i:j])` — convert that digit substring into an actual integer, e.g. `"4"` → `4`.
- `start = j + 1` — the real string content starts right after the `#`.
- `result.append(s[start : start + length])` — slice out exactly `length` characters. **This is the crux of why the scheme is correct even for adversarial content**: we're not searching for an end marker inside this slice, we already know precisely how many characters to take — so it doesn't matter whether those characters happen to include digits, `#` symbols, or anything else.
- `i = start + length` — move the pointer past this entire chunk (length-digits + `#` + string content), ready to read the next chunk.
- Loop continues until `i` reaches the end of the encoded string.

### Dry run

`strs = ["ab", "cd"]`

**Encoding:**
- `"ab"` → `"2#ab"` (length 2, delimiter, content)
- `"cd"` → `"2#cd"`
- Concatenated: `"2#ab2#cd"`

**Decoding `"2#ab2#cd"`:**

| step | i | scan for '#' from i | j (found at) | length = s[i:j] | start = j+1 | slice s[start:start+length] | result so far | new i |
|---|---|---|---|---|---|---|---|---|
| 1 | 0 | "2" then "#" at index 1 | 1 | "2" → 2 | 2 | s[2:4] = "ab" | ["ab"] | 4 |
| 2 | 4 | "2" then "#" at index 5 | 5 | "2" → 2 | 6 | s[6:8] = "cd" | ["ab","cd"] | 8 |

`i = 8 = len(s)`, loop ends. Result: `["ab", "cd"]` ✅ exactly matches the original input.

**The adversarial dry run — this is the one that actually proves the design is correct**, not just the happy path above: encode `["ab#cd", "ef"]` (a string that *contains* a `#`), producing `"5#ab#cd2#ef"`. Decoding: at `i=0`, we scan for the *first* `#` (found at index 1) to get length `5`, then — critically — we **blindly take the next 5 characters** `s[2:7]` = `"ab#cd"`, without ever re-scanning for another `#` inside that slice. The fact that those 5 characters happen to contain a `#` themselves is irrelevant, because we never search inside the slice at all — we just count. Continuing: `i` moves to `7`, and decoding proceeds correctly to recover `"ef"` next.

### Time & space complexity

- **Time: O(n)** for encode, where n = total characters across all strings (a single pass building the output). **Decode is also O(n)**: the inner `while s[j] != "#"` loop looks like it could make this O(n²) at first glance (a loop inside a loop), but it isn't — `j` only ever moves forward, and across the *entire* decode call, the combined total distance `j` travels (across all iterations of the outer `while i < len(s)` loop) can't exceed `len(s)`, since `j` never resets backward. This "inner pointer only ever advances, never restarts" argument is the same reasoning used to show Sliding Window algorithms are O(n) despite their nested-loop appearance — worth recognizing as a recurring proof pattern.
- **Space: O(n)** — the encoded string itself is proportional to the total input size (plus a small constant per string for its length prefix and delimiter).

---

## Common mistakes & misconceptions

1. **Believing a "rare" or "unusual" delimiter character (like `\n` or a control character) is safe enough.** The problem's guarantee that strings can contain *any* character makes this reasoning invalid in principle, even if it happens to pass casual testing — the design needs to be correct by construction, not correct by the delimiter being unlikely to collide.
2. **Forgetting that the length must be read as digits, not as a fixed-width field.** Some encodings pad lengths to a fixed width (e.g. always 4 digits) — that's also a valid, correct design, but the code above specifically relies on scanning for `#` to find the length's *variable*-width end; mixing the two ideas (variable-length numbers without a terminator) would reintroduce the exact ambiguity this approach was built to avoid.
3. **Off-by-one in the slice boundaries during decode.** `s[start : start + length]` is easy to mistype as `s[start : start + length - 1]` or similar — a careful dry run (as above) is the reliable way to catch this, since the bug wouldn't crash, just silently truncate or extend the decoded string by one character.
4. **Assuming this problem is "testing hashing" because it's filed under the Arrays & Hashing topic.** It isn't — no hash map or set appears anywhere in the correct solution. It's grouped here because it's an array/string-processing problem of similar flavor and difficulty, not because hashing is the relevant tool.

## Summary

| Approach | Correct? | Time | Space | Notes |
|---|---|---|---|---|
| Simple delimiter (comma-join) | ❌ No | O(n) | O(n) | Breaks if any string contains the delimiter — same time complexity as the correct answer, but wrong. |
| Length-prefixed encoding | ✅ Yes | O(n) | O(n) | Self-describing chunks — the standard correct answer. |

**Key takeaway:** when you need to serialize variable-length data unambiguously, **prefix each piece with its own length** rather than relying on a delimiter character that might collide with the data itself — and always stress-test a serialization scheme against input containing your *own* delimiter/marker characters, since that's precisely the case a naive design fails on. This exact idea reappears later in Serialize and Deserialize Binary Tree.
