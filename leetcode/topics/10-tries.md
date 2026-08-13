# Topic 10: Tries

## Core concepts / data structures

### Trie (pronounced "try," from re**trie**val)

**What it is:** a tree specialized for storing and searching **strings**, particularly efficient for **prefix**-based operations (like "does any word start with 'ap'?" or autocomplete). Each node represents **one character**, and a path from the root down through several nodes spells out a string, character by character. Nodes are shared between words that share a common prefix.

**The conceptual reason this beats a hash set for prefix questions:** a hash set (Arrays & Hashing topic) can answer "does this exact string exist?" in O(1), because hashing scatters entire strings into slots with no relationship between similar strings' locations — which is exactly why it *can't* answer "does anything starting with this prefix exist?" without scanning every stored string (there's no way to find "everything near this prefix" in a hash table, since similar keys aren't stored near each other). A trie inverts this: it deliberately keeps strings with shared prefixes **physically connected** in the structure, by literally sharing the path through those characters. That structural sharing is what makes prefix questions answerable by simply walking as far as the prefix goes — no scanning needed.

**Simple analogy:** think of a trie like the branching structure of an actual dictionary index, where "cat," "car," and "card" all share the same path for `c -> a`, and only diverge after that shared prefix (`t` vs `r`, and `r` further splits into an end-of-word vs. continuing to `d`). Instead of storing each full word separately, shared prefixes are stored **once** and reused by every word that starts with them.

### Node structure in Python
```python
class TrieNode:
    def __init__(self):
        self.children = {}       # character -> TrieNode
        self.is_end_of_word = False  # True if a word ends exactly at this node
```
- `children` is a dictionary mapping a single character to the next `TrieNode` in that direction. (Some implementations use a fixed-size array of 26 slots instead of a dict, when the alphabet is known and small — a common, slightly faster alternative, the same array-vs-dict tradeoff seen in Group Anagrams' character-count key.)
- `is_end_of_word` distinguishes "this path spells out a complete stored word" from "this path is just a prefix of some longer stored word(s)," which is essential — e.g. after inserting "card," walking `c -> a -> r` reaches a valid node, but that node should **not** be marked as a complete word unless "car" was *also* explicitly inserted. Without this flag, there would be no way to distinguish "a real word ends here" from "this is merely a waypoint on the way to a longer word."

### Core operations
```python
class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end_of_word = True

    def search(self, word):
        node = self._find_node(word)
        return node is not None and node.is_end_of_word

    def starts_with(self, prefix):
        return self._find_node(prefix) is not None

    def _find_node(self, s):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```
**Why `search` and `starts_with` differ by only one check, precisely:** walking down the trie following each character is identical for both operations — the only difference is that `search` additionally requires the final node to be explicitly marked `is_end_of_word` (a complete word was inserted there), while `starts_with` only cares that the path exists at all (it's a valid prefix of *something* stored, complete or not). This mirrors the trie's core design: the path-existence check and the "is this a complete word" check are genuinely separate questions, answered by separate pieces of information (the child-link structure vs. the flag).

## Common patterns / techniques in this topic

| Pattern | When it applies |
|---|---|
| **Basic insert / search / prefix-check** | The foundational trie operations, as shown above. |
| **Wildcard search** (e.g. `.` matches any character) | Use DFS/backtracking at the trie-traversal level: when a wildcard is encountered, try **every** child at that position instead of just one specific character — a direct application of the Backtracking topic's "try every choice, recurse" template to trie traversal. |
| **Storing extra data at each node or at word-ending nodes** | Some problems need more than "does this word exist" — e.g. storing a full word string at its ending node (instead of just a boolean), or a count of how many words pass through a given node, for prefix-counting problems. |
| **Trie + Backtracking combined** (e.g. word search on a grid, but searching for many words at once) | Build a trie of all target words first, then do a single combined grid search, following the trie's structure to prune impossible paths early — much faster than searching for each word separately, since shared prefixes across target words get explored only once. |

## Key terminology

- **Prefix** — the beginning portion of a string (e.g. "ca" is a prefix of "cat" and "car").
- **`is_end_of_word` marker** — distinguishes a "complete stored word ends here" node from a mere "valid prefix, but not a complete word" node.
- **Branching factor** — how many possible next characters a node could have (up to 26 for lowercase English letters, though usually far fewer are actually used at any given node — most nodes have only a handful of children in practice, which is part of why a dict-based `children` is often more space-efficient than a fixed 26-slot array).
- **Radix tree / Prefix tree** — alternative names you may see used interchangeably with "trie."

## Common beginner mistakes

1. **Forgetting the `is_end_of_word` flag entirely**, making `search("car")` incorrectly return `True` just because "car" happens to be a prefix of a *different* stored word like "card," when "car" itself was never actually inserted. This is exactly the failure mode the flag exists to prevent — without it, the trie can't distinguish "a real word" from "merely a waypoint."
2. **Confusing `search` (exact word match) with `starts_with`/prefix-check (partial match is enough)** — implementing one when the problem actually wants the other; always re-read the problem statement's exact wording, since these two operations differ by exactly one condition and it's easy to conflate them.
3. **Not handling wildcard characters correctly** in problems that need them (like Design Add and Search Words) — forgetting that a wildcard needs to try **all** children at that trie level, not just proceed as if it were a normal character, which requires branching (backtracking-style) rather than a simple deterministic walk.
4. **Rebuilding a trie from scratch when a persistent/reusable structure is needed.** If a problem does many searches against the same fixed dictionary, build the trie **once** up front, not per-query — this is the entire point of using a trie instead of just checking each word against the input directly; rebuilding it per-query would throw away the whole efficiency advantage.
5. **Using a trie when a simple hash set would do.** If you only ever need exact-match lookups (never prefix-based queries), a plain `set` of strings is simpler and just as fast for that specific need — reach for a trie specifically when prefix operations matter, per the "why a trie beats a hash set" explanation above; using one when it isn't needed adds complexity without benefit.

## How this compares to Hash Sets

A hash set answers "does this **exact** string exist?" in O(1) — but it cannot efficiently answer "does anything **starting with** this prefix exist?" without scanning every stored string, precisely because hashing deliberately scatters similar strings apart rather than keeping them together. A trie answers *both* kinds of questions in time proportional to the length of the string/prefix being checked (**not** the number of words stored), by sharing structure between words with common prefixes. Reach for a trie specifically when prefix-based questions matter; use a hash set when you only ever need exact matches — the two data structures are optimized for genuinely different questions, not just different implementations of the same question.

## Starter problems (medium, to warm up)

1. **Implement Trie (Prefix Tree)** (LeetCode #208) — in your Blind 75 list; build the exact core structure described above from scratch.
2. **Longest Common Prefix** (LeetCode #14) — not in Blind 75, but a good simple warm-up that can be solved with or without an actual trie, useful for building intuition about shared prefixes.

## What carries over from here

The trie's "shared prefix, branch on divergence" structure directly powers autocomplete and spell-check systems in the real world, and the technique of "build a trie of many target words, then search a grid/text once using it" (as in Word Search II) is a genuinely important optimization pattern when you'd otherwise need to repeat a search once per word — directly extending the Backtracking topic's grid-search techniques with the Trie topic's prefix-sharing power. The general skill of **designing a custom data structure with specific operations** (as opposed to just using arrays/hash maps directly) also reappears in the LRU Cache-style "design" problems common in the NeetCode 150 extension beyond Blind 75.
