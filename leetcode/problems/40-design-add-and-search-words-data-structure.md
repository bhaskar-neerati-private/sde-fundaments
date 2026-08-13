# 40. Design Add and Search Words Data Structure

**LeetCode:** [#211 - Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) · **Topic:** [Tries](../topics/10-tries.md) · **Difficulty:** Medium

## Problem statement

Design a data structure supporting:
- `addWord(word)` — adds a word.
- `search(word)` — returns `True` if there's a previously added word matching `word`, where `word` may contain `'.'` as a **wildcard** that matches any single letter.

**Example:**
```
addWord("bad")
addWord("dad")
addWord("mad")
search("pad") -> False
search("bad") -> True
search(".ad") -> True  (matches "bad", "dad", or "mad")
search("b..") -> True  (matches "bad")
```

## Applicable approaches

- **Brute Force — Check every added word against the pattern directly.**
- **Optimal — Trie + DFS Wildcard Search.** The standard, expected solution.

## Approach 1: Brute Force — Check Every Word Directly

### Intuition

Store all added words in a plain list. For `search`, compare the query against every stored word, character by character, treating `'.'` as "matches anything." This is a direct, literal implementation of the matching rule, but — like every brute force in this curriculum that "checks everything" — it completely ignores any structural sharing between stored words, redoing comparisons from scratch for every word even when many words share long common prefixes.

### Python code
```python
class WordDictionary:
    def __init__(self):
        self.words = []

    def addWord(self, word):
        self.words.append(word)

    def search(self, word):
        for stored in self.words:
            if len(stored) != len(word):
                continue
            if all(w == '.' or w == s for w, s in zip(word, stored)):
                return True
        return False
```

### Time & space complexity

- **Time: `addWord` is O(1)`; `search` is O(N · L)`** where N = number of stored words, L = word length — checks every stored word individually, doing O(L) comparison work per word.
- **Space: O(N · L)** for storing all words.

*(Simple, but doesn't exploit shared prefixes between words at all — the trie-based approach below is significantly better when there are many stored words with common prefixes, which is exactly the situation a trie is designed for, per the topic overview.)*

---

## Approach 2: Optimal — Trie + DFS Wildcard Search

### Intuition

Store all added words in a **trie** (exactly like Implement Trie), which naturally shares structure between words with common prefixes. The interesting part is `search`: for a **normal** character, we simply follow that specific child in the trie, same as a regular trie search. But for a **wildcard** (`'.'`), since it could match *any* letter, we don't know in advance which child to follow — so we need to try **all** of the current node's children, recursively continuing the search from each one, and succeed if *any* of those branches eventually leads to a full match. This is a direct application of the Backtracking topic's "try every choice, recurse, and succeed if any works" template, layered on top of the trie structure — the wildcard is precisely the point where the search stops being a deterministic walk and becomes an exploration.

### Algorithm

1. `addWord`: identical to a normal trie insert.
2. `search(word)`: use a recursive helper `dfs(node, index)`:
   - If `index == len(word)`: we've consumed the whole query — success only if the current node is marked `is_end_of_word`.
   - If `word[index]` is a normal letter: check if that letter exists as a child of `node`; if not, fail; if so, recurse into that one child with `index + 1`.
   - If `word[index]` is `'.'`: try **every** child of `node` — recurse into each one with `index + 1`; succeed if **any** of them eventually leads to a match.

### Python code
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.is_end_of_word = False

class WordDictionary:
    def __init__(self):
        self.root = TrieNode()

    def addWord(self, word):
        node = self.root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.is_end_of_word = True

    def search(self, word):
        def dfs(node, index):
            if index == len(word):
                return node.is_end_of_word

            ch = word[index]
            if ch == '.':
                for child in node.children.values():
                    if dfs(child, index + 1):
                        return True
                return False
            else:
                if ch not in node.children:
                    return False
                return dfs(node.children[ch], index + 1)

        return dfs(self.root, 0)
```

### Line-by-line explanation

- `addWord` — identical to a plain trie insert (see Implement Trie).
- `dfs(node, index)` — can `word[index:]` be matched starting from `node`?
- `if index == len(word): return node.is_end_of_word` — consumed the entire query string; success only if this exact path corresponds to a **complete** stored word (not just a valid prefix), matching the same distinction Implement Trie's `search` relies on.
- `ch = word[index]` — the character (or wildcard) we're currently trying to match.
- `if ch == '.':` — wildcard case: since it could represent *any* single letter, we don't know which child to follow, so we must **try them all**.
  - `for child in node.children.values(): if dfs(child, index + 1): return True` — recursively check every possible branch; the moment *any* branch succeeds, the whole search succeeds (we don't need to check the rest) — Python's early `return` inside the loop achieves the same short-circuit effect as `or`-chaining, just spelled out over multiple children instead of a fixed 4 directions like Word Search.
  - `return False` — only reached if every single child was tried and none led to a match.
- `else:` — normal character case, works exactly like a regular trie search: if this specific character doesn't exist as a child, fail immediately; otherwise, follow that one specific child — no branching needed, since a normal character has exactly one possible next step.
- `return dfs(self.root, 0)` — kick off the search from the top of the trie.

### Dry run

Trie contains "bad", "dad", "mad" (each inserted normally). `search(".ad")`

- `dfs(root, 0)`: `word[0] = '.'` → wildcard → try every child of root: `'b'`, `'d'`, `'m'` (the first letters of the three stored words).
  - Try child `'b'`: `dfs(node_b, 1)`: `word[1] = 'a'` (normal) → is `'a'` a child of `node_b`? Yes (from "bad") → `dfs(node_ba, 2)`.
    - `dfs(node_ba, 2)`: `word[2] = 'd'` (normal) → is `'d'` a child? Yes → `dfs(node_bad, 3)`.
      - `dfs(node_bad, 3)`: `index == len(word) == 3` → return `node_bad.is_end_of_word` → `True` (since "bad" was fully inserted).
    - This returns `True` all the way back up through the wildcard's `for` loop → `dfs(root, 0)` returns `True` immediately, without even needing to try children `'d'` or `'m'`.

Final: `search(".ad")` → `True` ✅ (found via the "bad" branch; the search short-circuited without needing to also check "dad" or "mad," though either would have worked too, since the `for` loop stops at the first success).

**A failing dry run:** `search("pad")` — `word[0]='p'` (normal, not wildcard) → is `'p'` a child of root? No (only `'b'`, `'d'`, `'m'` were inserted as first letters) → `dfs` returns `False` immediately, without exploring any deeper — a normal character that doesn't exist as a child fails in O(1), no branching required.

### Time & space complexity

- **Time: `addWord` is O(L)`.** **`search` is O(26^d · L)** in the *absolute* worst case, where d = number of wildcard characters in the query (each wildcard could branch into up to 26 possibilities, and this branching can compound across multiple wildcards). In practice, this is far better than it sounds, because most tries have far fewer than 26 actual children at any given node, and normal (non-wildcard) characters never branch at all — the true cost depends heavily on how many wildcards appear and how "bushy" the trie actually is at those points, so this worst case is a loose upper bound, not a typical-case estimate.
- **Space: O(N · L)** for the trie itself (same as a plain trie), plus O(L) recursion depth per search call.

---

## Common mistakes & misconceptions

1. **Treating `'.'` as if it were a literal character to look up in `node.children`.** `'.'` will almost never actually be a key in `children` (since real words don't contain it) — the wildcard needs its own branching logic, entirely separate from the normal-character lookup path.
2. **Not short-circuiting the wildcard's child loop.** Continuing to try every remaining child even after one has already succeeded wastes time (though it doesn't affect correctness) — the `return True` inside the loop, immediately upon success, is what makes this efficient in the common case.
3. **Believing this approach's worst-case complexity means it's impractical.** The O(26^d) bound is a true worst case, but real inputs (especially with few wildcards, or a trie that doesn't have 26 children at every node) perform far better — don't over-index on the worst-case formula when reasoning about whether this approach is "good enough" for typical use.
4. **Forgetting the base case checks node validity, not word validity.** `if index == len(word): return node.is_end_of_word` — a common slip is checking something about `word` itself here instead of `node`, but by this point `word` has already been fully consumed; what matters now is only whether the *trie position* we've arrived at represents a complete stored word.

## Summary

| Approach | addWord | search | Notes |
|---|---|---|---|
| Brute force (list of words) | O(1) | O(N·L) | Simple, but no benefit from shared prefixes. |
| Trie + DFS wildcard search | O(L) | up to O(26^d · L) worst case, typically much better | The standard, expected solution — efficient for the common case of few/no wildcards. |

**Key takeaway:** wildcard matching in a trie is a great example of **combining two techniques**: the trie structure for efficient prefix-sharing, and backtracking/DFS ("try every possibility, recurse, and succeed if any branch works") specifically to handle the wildcard's ambiguity. Recognizing when a "search with unknowns" problem needs this branch-and-explore approach — versus a plain deterministic trie walk — is the key skill this problem tests.
