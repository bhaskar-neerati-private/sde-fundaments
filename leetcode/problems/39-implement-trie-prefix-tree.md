# 39. Implement Trie (Prefix Tree)

**LeetCode:** [#208 - Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) · **Topic:** [Tries](../topics/10-tries.md) · **Difficulty:** Medium

## Problem statement

Implement a `Trie` class with:
- `insert(word)` — inserts a string into the trie.
- `search(word)` — returns `True` if the exact word was previously inserted.
- `startsWith(prefix)` — returns `True` if any previously inserted word starts with this prefix.

**Example:**
```
trie.insert("apple")
trie.search("apple")   -> True
trie.search("app")     -> False (never explicitly inserted)
trie.startsWith("app") -> True  (a prefix of "apple")
trie.insert("app")
trie.search("app")     -> True (now it has been)
```

## Applicable approaches

- **Array-based children (26 fixed slots, for lowercase English letters)**.
- **Hash-map-based children (dict, works for any character set)**.

Both approaches implement the exact same trie structure and algorithm from the topic overview — the difference is purely in how each node stores its children, and this problem is a good place to build both, since it's directly asking you to construct the core data structure this whole topic is built around.

## Approach 1: Array-Based Children (26 Slots)

### Intuition

Since this problem's constraints guarantee lowercase English letters only, each node can use a **fixed-size array of 26 slots** (one per letter), indexed via `ord(ch) - ord('a')` — the same character-to-index mapping trick used in Group Anagrams' character-count approach. This avoids hash map overhead entirely (direct array indexing is cheaper than hashing), at the cost of some wasted space for unused letter slots at nodes with few children.

### Python code
```python
class TrieNode:
    def __init__(self):
        self.children = [None] * 26
        self.is_end_of_word = False

class Trie:
    def __init__(self):
        self.root = TrieNode()

    def insert(self, word):
        node = self.root
        for ch in word:
            index = ord(ch) - ord('a')
            if node.children[index] is None:
                node.children[index] = TrieNode()
            node = node.children[index]
        node.is_end_of_word = True

    def search(self, word):
        node = self._find(word)
        return node is not None and node.is_end_of_word

    def startsWith(self, prefix):
        return self._find(prefix) is not None

    def _find(self, s):
        node = self.root
        for ch in s:
            index = ord(ch) - ord('a')
            if node.children[index] is None:
                return None
            node = node.children[index]
        return node
```

### Line-by-line explanation

- `self.children = [None] * 26` — a fixed array with one slot per lowercase letter, initially all empty (`None`).
- `index = ord(ch) - ord('a')` — maps `'a'`→0, `'b'`→1, ..., `'z'`→25 (same trick as Group Anagrams).
- `insert`: walk character by character, creating a new `TrieNode` in the appropriate slot whenever one doesn't already exist, then move into it; after the full word, mark the final node `is_end_of_word = True`.
- `_find(s)`: shared helper that walks as far as possible following `s`'s characters; returns the final node reached, or `None` the moment any character's slot is empty (meaning no stored word/prefix matches this far) — reusing one helper for both `search` and `startsWith` is exactly the topic overview's point that the two operations share the same walk, differing only in the final check.
- `search`: uses `_find`, but additionally requires the found node to be marked as a genuine word ending (not just a valid prefix path).
- `startsWith`: only needs `_find` to succeed at all — doesn't care whether the final node is a complete word or just a prefix.

### Dry run

`insert("app")`: root → create child 'a' → create child 'p' → create child 'p' → mark this last node `is_end_of_word = True`.

`search("app")`: `_find("app")` walks root→a→p→p successfully (all three characters found matching slots created during insert), reaches the node marked `is_end_of_word=True` → `search` returns `True`.

`search("ap")`: `_find("ap")` walks root→a→p successfully, reaches a node that exists (it was created as an intermediate step while inserting "app") but its `is_end_of_word` is `False` (only "app" was explicitly marked, not "ap") → `search` returns `False` — this is exactly the scenario the topic overview's mistake #1 warns about, correctly handled here because the flag is checked.

`startsWith("ap")`: `_find("ap")` succeeds (node exists) → returns `True`, regardless of the `is_end_of_word` flag — this is exactly the difference between the two operations.

### Time & space complexity

- **Time: O(L) per operation**, where L is the length of the word/prefix involved — each operation does one pass through the string's characters, with O(1) work per character (direct array indexing).
- **Space: O(N · L)** in the worst case for all insertions combined (N words, average length L), though shared prefixes reduce this in practice — each `TrieNode` itself uses O(26) space for its children array regardless of how many are actually filled, which is the main space trade-off of this approach.

---

## Approach 2: Hash-Map-Based Children

### Intuition

Using a `dict` instead of a fixed 26-slot array works for **any** character set (not just lowercase English letters), and avoids allocating unused space for letters that never actually appear as children at a given node — at the cost of slightly slower access (hash map operations vs. direct array indexing) and per-entry dictionary overhead. This is the same array-vs-dict tradeoff the topic overview flags: fixed-size and fast for a known small alphabet, versus flexible and more memory-proportional for a large or unknown alphabet.

### Python code
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False

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
        node = self._find(word)
        return node is not None and node.is_end_of_word

    def startsWith(self, prefix):
        return self._find(prefix) is not None

    def _find(self, s):
        node = self.root
        for ch in s:
            if ch not in node.children:
                return None
            node = node.children[ch]
        return node
```

### Line-by-line explanation

Structurally identical to Approach 1 — the only difference is `if ch not in node.children` / `node.children[ch]` (dict operations) instead of computing an array index. Every other line's logic and purpose is exactly the same as explained above.

### Time & space complexity

- **Time: O(L) per operation** — same as the array version, though with a somewhat larger constant factor per character due to hashing overhead (still O(1) average per character, just not as fast in absolute terms as direct array indexing).
- **Space: O(N · L)** overall, but typically **more space-efficient per node** than the fixed-array version when the alphabet is large or most nodes only actually have a few children — no wasted slots for unused letters.

---

## Common mistakes & misconceptions

1. **Forgetting to mark `is_end_of_word = True` after the insert loop.** Without it, no word would ever be found by `search`, even immediately after inserting it — a very common first-attempt bug, since it's easy to focus on building the child-link structure and forget the flag that actually marks completion.
2. **Checking `is_end_of_word` inside `startsWith`.** This would incorrectly make `startsWith` behave like `search` — it should only check that the path exists, per the topic overview's explicit distinction between the two operations.
3. **Using the array-based approach on input that isn't guaranteed lowercase English letters**, causing `ord(ch) - ord('a')` to produce an out-of-range or negative index for uppercase letters, digits, or other characters — always verify the problem's actual character-set constraints before committing to the fixed-array optimization.
4. **Creating a new `TrieNode` even when one already exists at that position.** `if node.children[index] is None:` (or `if ch not in node.children:`) is essential — without it, re-inserting a word (or inserting a second word sharing a prefix with the first) would overwrite and discard the existing subtree, losing everything previously built there.

## Summary

| Approach | Time (per op) | Space | Notes |
|---|---|---|---|
| Array-based children (26 slots) | O(L) | O(N·L), fixed 26 slots/node | Fastest in practice for a small, known alphabet (lowercase letters); some wasted space per node. |
| Hash-map-based children | O(L) | O(N·L), only used slots stored | More flexible (any character set), slightly slower per-character access, less wasted space. |

**Key takeaway:** both approaches implement the exact same trie *logic* — the choice between an array and a hash map for storing children is purely an implementation detail trading off speed vs. flexibility vs. memory, and it's worth being comfortable with both, since interviewers sometimes specifically ask "how would you change this if the alphabet were much larger?" as a direct follow-up.
