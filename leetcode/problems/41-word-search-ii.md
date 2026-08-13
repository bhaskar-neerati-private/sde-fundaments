# 41. Word Search II

**LeetCode:** [#212 - Word Search II](https://leetcode.com/problems/word-search-ii/) · **Topic:** [Tries](../topics/10-tries.md) · **Difficulty:** Hard

## Problem statement

Given an `m x n` grid of characters `board` and a list of strings `words`, return **all words from the list** that can be found in the grid (same adjacency rule as Word Search: up/down/left/right, no reusing a cell within one path).

**Example:**
```
board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]]
words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```

## Applicable approaches

- **Brute Force — Run Word Search's single-word search once per word in the list.**
- **Optimal — Build a Trie of all words, then do ONE combined grid search using it.** The standard, expected solution.

## Approach 1: Brute Force — Repeat Word Search per Word

### Intuition

We already know how to check if a *single* word exists in the grid (Word Search, problem 38). The most direct extension to multiple words: just run that exact algorithm once for each word in the list. This is correct, but it treats every word as a completely independent search, even when two target words share a long common prefix (like "eat" and "eaten") — in that case, the algorithm redundantly re-explores the exact same grid paths for the shared prefix, once per word that needs it.

### Python code
```python
def findWords(board, words):
    def exist(word):
        rows, cols = len(board), len(board[0])

        def dfs(r, c, index):
            if index == len(word):
                return True
            if (r < 0 or r >= rows or c < 0 or c >= cols
                    or board[r][c] != word[index]):
                return False
            temp = board[r][c]
            board[r][c] = "#"
            found = (dfs(r+1,c,index+1) or dfs(r-1,c,index+1) or
                     dfs(r,c+1,index+1) or dfs(r,c-1,index+1))
            board[r][c] = temp
            return found

        for r in range(rows):
            for c in range(cols):
                if dfs(r, c, 0):
                    return True
        return False

    return [word for word in words if exist(word)]
```

### Time & space complexity

- **Time: O(W · m · n · 4^L)** where W = number of words, m·n = grid size, L = max word length — the entire grid may be searched from scratch **once per word**, with no sharing of work between searches for words that share a common prefix (e.g. searching for "eat" and "eaten" separately re-explores the exact same "e-a-t" paths in the grid twice, doing genuinely duplicate work).
- **Space: O(L)** per search (recursion depth).

*(Correct, but wasteful whenever multiple target words share prefixes — which is exactly the situation a trie is designed to exploit, per the topic overview's "Trie + Backtracking combined" pattern.)*

---

## Approach 2: Optimal — Trie of All Words + One Combined Grid Search

### Intuition

Instead of searching the grid once per word, build a **trie containing all the target words first**. Then, do a **single** DFS exploration of the grid, but instead of following a fixed target string character by character, follow the **trie's structure**: at each grid cell, only continue exploring if the current character exists as a child in the trie at our current position — this means the search for "eat" and "eaten" (which share the prefix "eat") naturally reuses the same grid exploration up through "eat," splitting only where the words actually diverge, and stopping immediately at any grid path that doesn't match *any* stored word's prefix, no matter how many words we're searching for at once. This directly fixes Approach 1's specific inefficiency: instead of re-walking a shared prefix's grid path once per word that needs it, we walk it exactly once, and the trie structure tells us which words are still "in play" at every point along that shared path.

### Algorithm

1. Build a trie from all words in the list, with each trie node additionally storing the **complete word** (not just `is_end_of_word = True`) at nodes where a word ends — this makes it trivial to know exactly *which* word was found upon reaching such a node.
2. For each starting cell in the grid, DFS: at each step, check if the current grid cell's character exists as a child in the **current trie node** (not from the trie's root every time — we carry the trie position forward as the grid search proceeds). If not, stop this path — it can't match any stored word's prefix.
3. If it does exist, move into that trie child, and if that trie node has a complete word stored, add it to the results (and, as an optimization, remove the word from that trie node to avoid adding duplicates if the same word is reachable via multiple grid paths).
4. Recurse into the grid's 4 neighboring cells, continuing to follow the trie from this new position.
5. Backtrack (unmark the cell) after exploring, same discipline as Word Search.

### Python code
```python
class TrieNode:
    def __init__(self):
        self.children = {}
        self.word = None  # stores the complete word if one ends here, else None

def findWords(board, words):
    root = TrieNode()
    for word in words:
        node = root
        for ch in word:
            if ch not in node.children:
                node.children[ch] = TrieNode()
            node = node.children[ch]
        node.word = word

    rows, cols = len(board), len(board[0])
    result = []

    def dfs(r, c, node):
        ch = board[r][c]
        if ch not in node.children:
            return

        next_node = node.children[ch]
        if next_node.word is not None:
            result.append(next_node.word)
            next_node.word = None  # avoid duplicate additions

        board[r][c] = "#"  # mark visited

        for dr, dc in [(1,0), (-1,0), (0,1), (0,-1)]:
            nr, nc = r + dr, c + dc
            if 0 <= nr < rows and 0 <= nc < cols and board[nr][nc] != "#":
                dfs(nr, nc, next_node)

        board[r][c] = ch  # backtrack

    for r in range(rows):
        for c in range(cols):
            dfs(r, c, root)

    return result
```

### Line-by-line explanation

- `TrieNode.word` — instead of a simple `is_end_of_word` boolean, we store the **actual word string** at the ending node — this lets us directly know which word to record without needing to reconstruct it from the grid path taken, a small but genuinely useful deviation from the standard trie template.
- Building the trie — identical to a normal trie insert, just setting `node.word = word` at the end instead of a boolean flag.
- `dfs(r, c, node)` — explores the grid starting from cell `(r, c)`, while simultaneously tracking our current position `node` **within the trie** (not restarting from the trie's root each call — this is the crucial difference from the brute force, letting shared prefixes reuse the same exploration instead of redoing it).
- `ch = board[r][c]; if ch not in node.children: return` — if the current grid cell's letter isn't a valid next step from wherever we currently are in the trie, this path can't lead to *any* remaining target word — stop immediately. **This is the key pruning power**: a single failed check here prunes the search for **every** word that would have needed this same failed prefix, not just one word.
- `next_node = node.children[ch]` — advance our trie position.
- `if next_node.word is not None: result.append(next_node.word); next_node.word = None` — we've found a complete word ending exactly here — record it, and clear the marker so we don't accidentally add the same word again if a different grid path also happens to spell it out.
- `board[r][c] = "#"` then explore all 4 neighbors, then `board[r][c] = ch` — identical in-place-marking backtracking discipline as in Word Search, just now passing `next_node` (our updated trie position) into each recursive call instead of an incrementing string index, since the trie itself is now tracking "how far along some word(s) we are" instead of a single fixed target string.
- Outer double loop — start the combined search from every grid cell, same as before, but note: **it's still just one full grid traversal in total**, not one per word, which is the entire point of this optimization.

### Dry run (conceptual, abbreviated)

`words = ["eat", "eaten"]` (sharing the prefix "eat")

Trie structure: `root -> e -> a -> t (word="eat") -> e -> n (word="eaten")`.

When the grid search follows a path spelling "e-a-t" and reaches the trie node for "eat," it records "eat" **and simultaneously continues** exploring further from that *same* trie node (since it also has a child `'e'` leading toward "eaten") — no separate, repeated grid exploration was needed to *also* check for "eaten" — the shared "eat" prefix's grid exploration was done exactly once, benefiting both target words. Contrast this with Approach 1, which would have walked the grid's "e-a-t" path twice: once while checking "eat" in isolation, and again while checking "eaten" in isolation.

### Time & space complexity

- **Time: O(m · n · 4^L)** where L = max word length — importantly, this is the cost of **one combined search**, not multiplied by the number of words, because the trie-following check prunes any path that doesn't match *some* stored word's prefix, and shared prefixes across words reuse the same explored paths. (The exact worst case is more nuanced and depends on the trie's shape, but the key improvement over brute force is the removal of the "×W" factor for shared-prefix words — this is a direct parallel to how the Two Heaps median-finder eliminates a repeated-work factor by restructuring what's tracked.)
- **Space: O(total characters across all words)** for the trie, plus O(L) recursion depth for the grid search.

---

## Common mistakes & misconceptions

1. **Restarting from the trie's root on every grid step instead of carrying the current trie node forward.** This would silently degrade the algorithm back toward Approach 1's redundancy — if `dfs` always started matching from `root` instead of continuing from `next_node`, the trie would provide no actual sharing benefit; the entire optimization *is* carrying trie position as state through the grid recursion.
2. **Forgetting to clear `next_node.word` after recording it.** Without this, if the same word were reachable via two different grid paths (e.g. a repeated pattern in the board), it could be added to `result` twice — the clearing step is a real correctness fix, not just tidiness.
3. **Using `is_end_of_word` (boolean) instead of storing the actual word string at trie nodes.** This works too, but then requires separately reconstructing the matched word from the grid path (e.g. accumulating characters as you go) — storing the word directly on the trie node, as shown here, is simpler and avoids that extra bookkeeping.
4. **Not pruning search starting points.** Some further-optimized versions also remove a trie branch entirely once every word under it has been found (so future grid cells hitting that dead branch fail even faster), which this solution doesn't do — worth knowing as a further possible optimization, though not necessary for a correct, reasonably efficient answer.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Repeat Word Search per word | O(W · m·n · 4^L) | O(L) | Simple, but re-explores shared prefixes redundantly for every word. |
| Trie of all words + one combined search | O(m·n · 4^L) (roughly, sharing across words) | O(total word length) | The standard, expected optimal solution. |

**Key takeaway:** whenever you need to search for **many** related strings against the same data (a grid, a text, etc.), check whether building a trie of the target strings first lets you do **one combined search** instead of one search per target — this is the single biggest lesson of this problem, extending Word Search's backtracking with the Trie topic's prefix-sharing power, and it's a direct, concrete illustration of the topic overview's "Trie + Backtracking combined" pattern.
