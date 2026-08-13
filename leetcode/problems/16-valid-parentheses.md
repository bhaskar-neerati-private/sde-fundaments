# 16. Valid Parentheses

**LeetCode:** [#20 - Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) · **Topic:** [Stack](../topics/04-stack.md) · **Difficulty:** Easy

## Problem statement

Given a string `s` containing just the characters `'('`, `')'`, `'{'`, `'}'`, `'['`, `']'`, determine if the input string is valid. A string is valid if:
- Every opening bracket has a matching closing bracket of the **same type**.
- Brackets close in the **correct order** (the most recently opened bracket must be the next one closed).

**Example:**
```
Input: s = "()[]{}"
Output: true

Input: s = "(]"
Output: false

Input: s = "([)]"
Output: false  (wrong order - the '(' should close before the '[' does)
```

## Applicable approaches

- **Brute Force — Repeatedly remove matched pairs.**
- **Optimal — Stack.** The standard, expected O(n) solution.

## Approach 1: Brute Force — Repeatedly Remove Matched Pairs

### Intuition

A valid string can always be reduced to empty by repeatedly finding and deleting any adjacent matched pair (like `"()"`, `"[]"`, `"{}"`) — if you can't fully reduce it to nothing this way, it was invalid. This is a correct, if inefficient, way to check validity, because every genuinely valid string is built entirely out of such nested matched pairs, and removing any innermost pair never destroys the validity of what remains around it.

### Algorithm

1. Repeat: scan the string for any occurrence of `"()"`, `"[]"`, or `"{}"` and remove it.
2. Stop when no more such pairs can be found.
3. If the string is now empty, it was valid; otherwise, it wasn't.

### Python code
```python
def isValid(s):
    while "()" in s or "[]" in s or "{}" in s:
        s = s.replace("()", "").replace("[]", "").replace("{}", "")
    return s == ""
```

### Line-by-line explanation

- The `while` loop keeps removing matched adjacent pairs as long as any exist.
- `.replace("()", "")` etc. — removes every occurrence of that exact pair in one pass; each `.replace()` call itself scans the whole (currently-shrinking) string.
- Once no more removable pairs exist, if anything is left over, it was invalid — either mismatched types created a permanently un-removable fragment, or wrong ordering (like `"([)]"`) left brackets that can never become adjacent to their match no matter how many inner pairs get removed.

### Time & space complexity

- **Time: O(n²)** — each pass through the `while` loop can remove at most a constant fraction of remaining pairs in the worst case, and each `.replace()` call is itself O(n); this can require many passes over a shrinking string, and each `.replace()` also allocates a new string.
- **Space: O(n)** — `.replace()` creates a new string each time, rather than modifying in place (Python strings are immutable).

*(This approach is correct but inefficient — shown mainly to build intuition; the stack approach below is what's actually expected and is the standard interview answer.)*

---

## Approach 2: Optimal — Stack

### Intuition

Every time we see an **opening** bracket, we don't yet know what it's going to match with — we just need to remember it, with the *most recently opened, still-unmatched* one always being the next candidate for matching. Every time we see a **closing** bracket, it must match that exact most-recent unmatched opener — not any arbitrary earlier one. This is precisely the LIFO guarantee a stack provides (see the topic overview): pushing every opener and popping on every closer naturally enforces "closes in the reverse order they were opened," which is exactly the rule the problem states in words. This is a case where the data structure isn't just a convenient tool — its ordering guarantee *is* the problem's correctness rule, restated.

### Algorithm

1. Create an empty stack, and a mapping from closing bracket → its matching opening bracket (for easy lookup).
2. Walk through the string one character at a time:
   - If it's an **opening** bracket (`(`, `[`, `{`), push it onto the stack.
   - If it's a **closing** bracket: check if the stack is non-empty **and** the top of the stack matches this closing bracket's expected opener. If either check fails, the string is invalid — return `False` immediately. Otherwise, pop the stack (that opening bracket has now been matched).
3. After processing the whole string, the string is valid **only if the stack is completely empty** (every opening bracket found its match; nothing was left unmatched).

### Python code
```python
def isValid(s):
    pairs = {')': '(', ']': '[', '}': '{'}
    stack = []

    for ch in s:
        if ch in pairs:  # it's a closing bracket
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()
        else:  # it's an opening bracket
            stack.append(ch)

    return len(stack) == 0
```

### Line-by-line explanation

- `pairs = {')': '(', ']': '[', '}': '{'}` — maps each closing bracket to the opening bracket it should match.
- `stack = []` — holds unmatched opening brackets, most recent on top.
- `if ch in pairs:` — `ch` is a closing bracket, since `pairs`'s keys are exactly the three closing brackets.
- `if not stack or stack[-1] != pairs[ch]:` — **two distinct failure conditions checked in one line, and it's worth being able to name both**: `not stack` catches a closing bracket with nothing open to match it at all (e.g. `")"` as the very first or an extra character); `stack[-1] != pairs[ch]` catches a **type mismatch** — the stack isn't empty, but its top is the wrong kind of bracket (e.g. stack has `'('` on top but we see `']'`). Python's short-circuit evaluation of `or` guarantees `stack[-1]` is never evaluated when `stack` is empty, so this is also safe from an index error.
- `return False` — either failure means the string is invalid; stop immediately, no need to keep scanning.
- `stack.pop()` — the top of the stack correctly matched this closing bracket — remove it, it's resolved.
- `else: stack.append(ch)` — it's an opening bracket (not a key in `pairs`), so remember it for later matching.
- `return len(stack) == 0` — after processing everything, valid only if nothing was left unmatched (catches inputs like `"((("`, which never trigger a `False` inside the loop — every character pushed successfully — but leave brackets open at the end).

### Dry run

`s = "([)]"` (expected `False` — wrong order)

| ch | is closing? | check | action | stack after |
|---|---|---|---|---|
| ( | no | - | push | ['('] |
| [ | no | - | push | ['(', '['] |
| ) | yes | `pairs[')']='('`; `stack[-1]='['` → `'[' != '('` → **mismatch** | return `False` | - |

Correctly detects the wrong-order case: the `')'` needed to match the most recent opener `'['`, but `'['` ≠ `'('` — even though a `'('` genuinely exists earlier in the stack, it is *not* the most recent one, and matching against it instead would violate the LIFO/nesting rule.

**A passing dry run:** `s = "()[]{}"`

| ch | action | stack after |
|---|---|---|
| ( | push | ['('] |
| ) | matches top '(' → pop | [] |
| [ | push | ['['] |
| ] | matches top '[' → pop | [] |
| { | push | ['{'] |
| } | matches top '{' → pop | [] |

Loop finishes, `len(stack) == 0` → return `True` ✅

### Time & space complexity

- **Time: O(n)** — one pass through the string, each push/pop is O(1).
- **Space: O(n)** — worst case (all opening brackets, e.g. `"((((("`), the stack holds every character.

---

## Common mistakes & misconceptions

1. **Checking `stack[-1] != pairs[ch]` without first checking `not stack`.** If you reverse the order or omit the empty-stack check, `stack[-1]` on an empty list raises an `IndexError` — the check order in `if not stack or stack[-1] != pairs[ch]` matters both for correctness *and* to avoid a crash, relying on short-circuit evaluation.
2. **Forgetting the final `len(stack) == 0` check.** Without it, an input like `"((("` would incorrectly return `True` (or whatever the function's default fallthrough is) — the loop body never triggers a `False` for unmatched *openers*, only for bad *closers*, so leftover openers must be caught separately at the end.
3. **Mapping openers to closers instead of closers to openers.** It's tempting to write `pairs = {'(': ')', ...}` and then check `pairs[stack[-1]] == ch` — this works too, but it's easy to get the direction backward under time pressure (checking `pairs[ch] == stack[-1]` vs `ch == pairs[stack[-1]]`); pick one direction and be consistent, and verify it against a concrete example rather than trusting memory.
4. **Assuming a stack is overkill for "such a simple problem."** This is a common instinct for beginners who haven't yet internalized that LIFO ordering *is* the problem's rule, not an implementation detail — a solution using only counters (e.g. counting total opens vs. closes) would incorrectly accept `"([)]"` as valid, since it has equal counts of every bracket type but wrong nesting order; counting alone cannot detect ordering violations, only a stack can.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Repeated pair removal | O(n²) | O(n) | Correct but slow; illustrates the idea without a stack. |
| Stack | O(n) | O(n) | The standard, expected optimal solution. |

**Key takeaway:** "the most recently opened thing must be the next thing closed" is *the* defining signal for reaching for a stack — and it's worth recognizing that a plain count-based check (opens == closes) is a common, tempting, but insufficient substitute, since it verifies quantity without verifying order. The push-on-open, pop-and-check-on-close pattern here is worth having ready to write from memory, since it reappears (sometimes disguised) in expression-parsing and nested-structure problems generally.
