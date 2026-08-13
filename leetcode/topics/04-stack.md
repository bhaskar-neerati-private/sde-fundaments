# Topic 4: Stack

## Core concepts / data structures

### Stack

**What it is:** a collection where you can only add ("push") or remove ("pop") from **one end**, called the "top." This means the **last** item added is always the **first** one removed — "Last In, First Out," usually shortened to **LIFO**.

**Simple explanation:** think of a stack of plates. You can only take a plate off the top (the one you most recently put down), and you can only add a new plate to the top too. You can't grab a plate from the middle or bottom without first removing everything above it. That's a stack.

**Why "last in, first out" is the actual point, not a trivia fact:** the LIFO property isn't just a description of behavior — it's what makes a stack the right tool whenever a problem's correctness depends on **undoing things in the exact reverse order they were done**, or matching each new item against the **most recently unresolved** item. If a problem's structure requires "the most recent open thing must be the next thing closed," a stack isn't just convenient, it's *structurally* the correct model of the problem — the data structure's ordering guarantee directly mirrors the problem's ordering requirement.

**Python implementation:** Python doesn't have a dedicated "Stack" type — a plain `list` works perfectly, using `.append(x)` to push and `.pop()` to pop. Both are O(1) because they operate on the *end* of the list, which (per the Arrays & Hashing topic's mental model) is the one position where Python's array-backed list can add/remove without shifting every other element — there's no need to renumber anything when the change happens at the boundary.

```python
stack = []
stack.append(1)   # stack: [1]
stack.append(2)   # stack: [1, 2]
stack.append(3)   # stack: [1, 2, 3]
top = stack.pop()  # removes and returns 3 (the most recently added) -> stack: [1, 2]
peek = stack[-1]   # look at the top without removing it -> 2
```

### What stacks are naturally good at

A stack is the right tool whenever a problem involves **matching something with the most recent unmatched thing that came before it**, or **needing to "undo" the most recent step before going further**. Concrete signals that a stack might help:
- Matching pairs (like opening/closing brackets) where the *most recently opened* one must be the *next* one closed.
- "Look back to the nearest previous element that satisfies some condition" (the **monotonic stack** pattern).
- Simulating recursion without actually recursing (an "explicit stack" instead of relying on the call stack) — used in iterative DFS, for example.
- Undo/redo functionality, or reversing the order of anything.

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Matching pairs** | Valid Parentheses-style problems: push an "opening" token, and when a "closing" token appears, check it against what's on top of the stack. |
| **Monotonic stack** | "For each element, find the nearest previous/next element that's larger/smaller" (e.g. Daily Temperatures, Next Greater Element — not in Blind 75, but a very common stack pattern elsewhere). The stack is kept in increasing or decreasing order by popping off anything that would break that order before pushing the new element. |
| **Expression evaluation** | Evaluating math expressions (especially in postfix/RPN form, or implementing a calculator) — operators act on whatever's most recently on the stack. |
| **Simulating recursion (explicit stack)** | Converting a recursive algorithm (like DFS) into an iterative one, when recursion depth might be a concern, by manually managing a stack instead of relying on function calls. |

## Key terminology

- **LIFO** — Last In, First Out; the defining property of a stack, and the property that makes it the correct model for "undo most-recent-first" problems.
- **Push / Pop** — add to the top / remove from the top.
- **Peek / Top** — look at the top element without removing it.
- **Monotonic stack** — a stack that's kept either strictly increasing or strictly decreasing from bottom to top, by popping elements that would violate that order before pushing a new one.
- **Call stack** — the stack the *programming language itself* uses to track function calls (including recursive ones). Understanding "explicit stack" problems is easier once you realize recursion is secretly automatic stack management done for you by the language runtime — every recursive call pushes a frame, every `return` pops one, in the exact LIFO order.

## Common beginner mistakes

1. **Forgetting to check if the stack is empty before popping or peeking.** If a "closing" token appears with nothing open to match it, that's invalid input — popping from an empty stack (or `stack[-1]` on an empty list) crashes or behaves unexpectedly if not checked first. Always check `len(stack) > 0` (or use Python's short-circuiting `and`/`or`) before accessing the top.
2. **Checking only "is the stack non-empty" without checking that the popped item is actually the *correct* match.** For bracket-matching, you need to compare the closing bracket to the *type* of the opening bracket that should match it — a stack that's merely non-empty doesn't guarantee the top is the right type.
3. **Not checking that the stack is completely empty at the very end.** A string like `"((("` has three opening brackets and no closing ones — the loop finishes without ever detecting an error mid-scan, but the stack isn't empty at the end, meaning something was left unmatched. Always check `len(stack) == 0` after the loop as part of the final validity verdict.
4. **Using a stack when the problem actually needs a queue.** Stacks are LIFO; queues (see the Trees/Graphs topics for BFS) are FIFO (First In, First Out). Reaching for a stack when a problem needs "process things in the order they arrived" produces results in the wrong order, not a crash — a silent logic bug, not an exception.

## Starter problems (easy, to warm up)

1. **Valid Parentheses** (LeetCode #20) — the canonical stack-matching problem. Also in your Blind 75 list.
2. **Baseball Game** (LeetCode #682) — not in Blind 75, but a good simple warm-up for "the stack represents history, and operations act on the most recent entries."
3. **Min Stack** (LeetCode #155) — not in Blind 75, but an excellent exercise in designing a stack that also supports O(1) minimum-tracking, a common follow-up question after Valid Parentheses.

## How this compares to Arrays & Hashing / Two Pointers

Where Two Pointers uses two indices moving through a fixed array, a Stack is a genuinely different data structure you build up and tear down as you go — it's for when you need to remember an **ordered history of unmatched/unprocessed things**, not just track a position or a window. If a problem needs "the most recent X that hasn't been resolved yet," that's a strong signal for a stack, where hashing (Arrays & Hashing) would only help you answer "have I seen this value at all," with no sense of *order* or *recency* — a hash set can tell you a bracket type exists, but never which one is the *most recent* unmatched one.

## What carries over from here

The "explicit stack instead of recursion" idea reappears directly when you learn **iterative DFS** in the Trees and Graphs topics (a stack-based DFS visits nodes in nearly the same order recursive DFS would, precisely because recursion *is* an implicit stack — see the call stack definition above). The general "match the current thing against the most recent unresolved thing" idea also reappears in Backtracking, where you're implicitly using the call stack to "undo" choices that didn't pan out, in the exact reverse order they were made.
