# LeetCode Interview Prep

Recovered from the old project location (`C:\Users\lenovo\Desktop\devProjects\sde_fundamentals`, before the move to `misc\sde_fundamentals`). Consolidates the teaching-style rules and the problem order agreed on 2026-07-31.

## Goal

Prep for coding interviews at product-based companies. User can write Python well (with online references) but wants structured, pattern-based LeetCode practice with deep conceptual understanding - not random problem-solving.

## Session flow

- User pings whenever he's free for the day, asking for "a problem."
- The next problem is picked from the fixed pre-agreed order below (not random), one pattern at a time.

## Teaching rules (staged, confirm-per-step)

When teaching a problem, follow this flow and **stop for User's confirmation at each stage** before moving to the next:

1. Explain the **original LeetCode problem** - always state its **LeetCode problem number** first - then the statement, constraints, and examples in simple English → wait for confirmation.
2. Explain the **brute-force solution** → wait for confirmation.
3. Explain the **optimized solution** (can be broken into multiple incremental steps) → wait for confirmation.
4. Move to the next problem. **No mind map / recap diagram** (dropped 2026-08-01 after problem #1, same call as the general curriculum).

Additional rules:
- Use fresh, everyday-life analogies chosen per concept - don't force one analogy across the whole explanation; pick whatever fits each specific idea best.
- Include short Python code examples wherever they clarify the idea.
- Keep language simple/plain English throughout - avoid unnecessary jargon.

## Phase 1 target: Blind 75 (accepted minimum baseline)

Reviewed 2026-08-01 against NeetCode's own Blind 75 tracker (the standard reference most people use to work this list) and cross-checked problem names against 3+ independent listings. Order below is NeetCode's own sequencing - foundational array/string patterns first, DP/greedy/math last, since later categories lean on earlier ones.

### Arrays & Hashing (8)

1. ~~Two Sum (#1) ✅~~
    - 🎥 [Two Sum - Leetcode 1 - HashMap - Python](https://www.youtube.com/watch?v=KLlXCFG5TnA) (NeetCode)
    - 📖 [NeetCode: Two Sum - full writeup](https://neetcode.io/solutions/two-sum)
2. ~~Contains Duplicate (#217) ✅~~
    - 🎥 [Contains Duplicate - Leetcode 217 - Python](https://www.youtube.com/watch?v=3OamzN90kPg) (NeetCode)
    - 📖 [NeetCode: Contains Duplicate - full writeup](https://neetcode.io/solutions/contains-duplicate)
3. ~~Valid Anagram (#242) ✅~~
    - 🎥 [Valid Anagram - Leetcode 242 - Python](https://www.youtube.com/watch?v=9UtInBqnCgA) (NeetCode)
    - 📖 [NeetCode: Valid Anagram - full writeup](https://neetcode.io/solutions/valid-anagram)
4. ~~Group Anagrams (#49) ✅~~
    - 🎥 [Group Anagrams - Categorize Strings by Count - Leetcode 49](https://www.youtube.com/watch?v=vzdNOK2oB2E) (NeetCode)
    - 📖 [NeetCode: Group Anagrams - full writeup](https://neetcode.io/solutions/group-anagrams)
5. ~~Top K Frequent Elements (#347) ✅~~
    - 🎥 [Top K Frequent Elements - Bucket Sort - Leetcode 347 - Python](https://www.youtube.com/watch?v=YPTqKIgVk-k) (NeetCode)
    - 📖 [NeetCode: Top K Frequent Elements - full writeup](https://neetcode.io/solutions/top-k-frequent-elements)
6. ~~Product of Array Except Self (#238) ✅~~
    - 🎥 [Product of Array Except Self - Leetcode 238 - Python](https://www.youtube.com/watch?v=bNvIQI2wAjk) (NeetCode)
    - 📖 [NeetCode: Product of Array Except Self - full writeup](https://neetcode.io/solutions/product-of-array-except-self)
7. ~~Encode and Decode Strings (#271) ✅~~
    - 🎥 [Encode and Decode Strings - Leetcode 271 - Python](https://www.youtube.com/watch?v=B1k_sxOSgv8) (NeetCode)
    - 📖 [NeetCode: Encode and Decode Strings - full writeup](https://neetcode.io/solutions/encode-and-decode-strings)
8. ~~Longest Consecutive Sequence (#128) ✅~~
    - 🎥 [Longest Consecutive Sequence - Leetcode 128 - Hashmaps & Sets (Python)](https://www.youtube.com/watch?v=joIEdeOGqjQ) (NeetCode)
    - 📖 [NeetCode: Longest Consecutive Sequence - full writeup](https://neetcode.io/solutions/longest-consecutive-sequence)
### Two Pointers (3)


9. ~~Valid Palindrome (#125) ✅~~
    - 🎥 [Valid Palindrome - Leetcode 125 - Python](https://www.youtube.com/watch?v=jJXJ16kPFWg) (NeetCode)
    - 📖 [NeetCode: brute-force cleaned-string build vs. optimized two-pointer O(1) space](https://neetcode.io/solutions/valid-palindrome)
10. ~~3Sum (#15) ✅~~
    - 🎥 [3Sum - Leetcode 15 - Python](https://www.youtube.com/watch?v=jzZsG8n2R9A) (NeetCode)
    - 📖 [NeetCode: brute-force O(n³) → hashmap O(n²) → two-pointer O(n²), with duplicate-handling pitfalls](https://neetcode.io/solutions/3sum)
11. ~~Container With Most Water (#11) ✅~~
    - 🎥 [Container with Most Water - Leetcode 11 - Python](https://www.youtube.com/watch?v=UuiTKBwPgAo) (NeetCode)
    - 📖 [NeetCode: brute-force O(n²) all-pairs vs. optimized two-pointer O(n)](https://neetcode.io/solutions/container-with-most-water)
### Sliding Window (4)


12. ~~Best Time to Buy and Sell Stock (#121) ✅~~
    - 🎥 [Sliding Window: Best Time to Buy and Sell Stock - Leetcode 121 - Python](https://www.youtube.com/watch?v=1pkOgXD63yU) (NeetCode)
    - 📖 [NeetCode: brute-force O(n²) pair check vs. two-pointer/DP O(n) tracking min-so-far](https://neetcode.io/solutions/best-time-to-buy-and-sell-stock)
13. ~~Longest Substring Without Repeating Characters (#3) ✅~~
    - 🎥 [Longest Substring Without Repeating Characters - Leetcode 3 - Python](https://www.youtube.com/watch?v=wiGpQwVHdE0) (NeetCode)
    - 📖 [NeetCode: brute-force O(n·m) → sliding window O(n) → optimal sliding window with hashmap index jumps](https://neetcode.io/solutions/longest-substring-without-repeating-characters)
14. ~~Longest Repeating Character Replacement (#424) ✅~~
    - 🎥 [Longest Repeating Character Replacement - Leetcode 424 - Python](https://www.youtube.com/watch?v=gqXU1UyA8pk) (NeetCode)
    - 📖 [NeetCode: brute-force O(n²) → beginner sliding window → optimal single-pass sliding window O(n)](https://neetcode.io/solutions/longest-repeating-character-replacement)
15. Minimum Window Substring (#76)
    - 🎥 [Minimum Window Substring - Airbnb Interview Question - Leetcode 76](https://www.youtube.com/watch?v=jSto0O4AJbM) (NeetCode)
    - 📖 [NeetCode: brute-force O(n²) substring check vs. optimized sliding window O(n+m) with frequency maps](https://neetcode.io/solutions/minimum-window-substring)
### Stack (1)


16. Valid Parentheses (#20)
    - 🎥 [Valid Parentheses - Stack - Leetcode 20 - Python](https://www.youtube.com/watch?v=WTzjTskDFMg) (NeetCode)
    - 📖 [NeetCode: brute-force repeated pair-removal vs. optimized stack-based O(n)](https://neetcode.io/solutions/valid-parentheses)
### Binary Search (2)


17. Find Minimum in Rotated Sorted Array (#153)
    - 🎥 [Find Minimum in Rotated Sorted Array - Binary Search - Leetcode 153 - Python](https://www.youtube.com/watch?v=nIVW4P8b1VA) (NeetCode)
    - 📖 [NeetCode: brute-force O(n) linear scan vs. binary search O(log n)](https://neetcode.io/solutions/find-minimum-in-rotated-sorted-array)
18. Search in Rotated Sorted Array (#33)
    - 🎥 [Search in rotated sorted array - Leetcode 33 - Python](https://www.youtube.com/watch?v=U8XENwh8Oy8) (NeetCode)
    - 📖 [NeetCode: brute-force O(n) linear search vs. binary search O(log n) pivot-based elimination](https://neetcode.io/solutions/search-in-rotated-sorted-array)
### Linked List (6)


19. Reverse Linked List (#206)
    - 🎥 [Reverse Linked List - Iterative AND Recursive - Leetcode 206 - Python](https://www.youtube.com/watch?v=G0_I-ZF0S38) (NeetCode)
    - 📖 [NeetCode: brute-force array reversal vs. optimized iterative/recursive approaches](https://neetcode.io/solutions/reverse-linked-list)
20. Merge Two Sorted Lists (#21)
    - 🎥 [Merge Two Sorted Lists - Leetcode 21 - Python](https://www.youtube.com/watch?v=XIdigk956u0) (NeetCode)
    - 📖 [NeetCode: brute-force merge-into-array-and-sort vs. recursive/iterative dummy-node approaches](https://neetcode.io/solutions/merge-two-sorted-lists)
21. Linked List Cycle (#141)
    - 🎥 [Linked List Cycle - Floyd's Tortoise and Hare - Leetcode 141 - Python](https://www.youtube.com/watch?v=gBTe7lFR3vc) (NeetCode)
    - 📖 [NeetCode: brute-force hash-set vs. optimized fast/slow pointer (Floyd's) approach](https://neetcode.io/solutions/linked-list-cycle)
22. Reorder List (#143)
    - 🎥 [Linkedin Interview Question - Reorder List - Leetcode 143 - Python](https://www.youtube.com/watch?v=S5bfdUTrKLM) (NeetCode)
    - 📖 [NeetCode: brute-force array + two pointers vs. optimized find-middle/reverse/merge approach](https://neetcode.io/solutions/reorder-list)
23. Remove Nth Node From End of List (#19)
    - 🎥 [Remove Nth Node from End of List - Oracle Interview Question - Leetcode 19](https://www.youtube.com/watch?v=XVuQxVej6y8) (NeetCode)
    - 📖 [NeetCode: brute-force array-store vs. two-pass and optimized one-pass two-pointer approaches](https://neetcode.io/solutions/remove-nth-node-from-end-of-list)
24. Merge K Sorted Lists (#23)
    - 🎥 [Merge K Sorted Lists - Leetcode 23 - Python](https://www.youtube.com/watch?v=q5a5OiGbT6Q) (NeetCode)
    - 📖 [NeetCode: brute-force sort, sequential pairwise merge, heap-based, and divide-and-conquer approaches](https://neetcode.io/solutions/merge-k-sorted-lists)
### Trees (11)


25. Invert Binary Tree (#226)
    - 🎥 [Invert Binary Tree - Depth First Search - Leetcode 226](https://www.youtube.com/watch?v=OnSn2XEQ4MY) (NeetCode)
    - 📖 [NeetCode: BFS, recursive DFS, and iterative DFS approaches](https://neetcode.io/solutions/invert-binary-tree)
26. Maximum Depth of Binary Tree (#104)
    - 🎥 [Maximum Depth of Binary Tree - 3 Solutions - Leetcode 104 - Python](https://www.youtube.com/watch?v=hTM3phVI6YQ) (NeetCode)
    - 📖 [NeetCode: recursive DFS, iterative DFS (stack), and BFS approaches](https://neetcode.io/solutions/maximum-depth-of-binary-tree)
27. Same Tree (#100)
    - 🎥 [Same Tree - Leetcode 100 - Python](https://www.youtube.com/watch?v=vRbbcKXCxOw) (NeetCode)
    - 📖 [NeetCode: recursive DFS, iterative DFS, and BFS level-order comparison](https://neetcode.io/solutions/same-tree)
28. Subtree of Another Tree (#572)
    - 🎥 [Subtree of Another Tree - Leetcode 572 - Python](https://www.youtube.com/watch?v=E36O5SWp-LE) (NeetCode)
    - 📖 [NeetCode: brute-force DFS vs. optimized serialization + string-match approach](https://neetcode.io/solutions/subtree-of-another-tree)
29. Binary Tree Level Order Traversal (#102)
    - 🎥 [Binary Tree Level Order Traversal - BFS - Leetcode 102](https://www.youtube.com/watch?v=6ZnyEApgFYg) (NeetCode)
    - 📖 [NeetCode: DFS-with-depth approach and the standard BFS/queue approach](https://neetcode.io/solutions/binary-tree-level-order-traversal)
30. Construct Binary Tree from Preorder and Inorder Traversal (#105)
    - 🎥 [Construct Binary Tree from Inorder and Preorder Traversal - Leetcode 105 - Python](https://www.youtube.com/watch?v=ihj4IQGZ2zc) (NeetCode)
    - 📖 [NeetCode: basic O(n²) DFS through hash-map-optimized and limit-based optimal DFS](https://neetcode.io/solutions/construct-binary-tree-from-preorder-and-inorder-traversal)
31. Binary Tree Maximum Path Sum (#124)
    - 🎥 [Binary Tree Maximum Path Sum - DFS - Leetcode 124 - Python](https://www.youtube.com/watch?v=Hr5cWUld4vU) (NeetCode)
    - 📖 [NeetCode: brute-force DFS vs. single-pass optimal DFS](https://neetcode.io/solutions/binary-tree-maximum-path-sum)
32. Serialize and Deserialize Binary Tree (#297)
    - 🎥 [Serialize and Deserialize Binary Tree - Preorder Traversal - Leetcode 297 - Python](https://www.youtube.com/watch?v=u4JAi2JJhI8) (NeetCode)
    - 📖 [NeetCode: DFS/preorder serialization and BFS/level-order serialization approaches](https://neetcode.io/solutions/serialize-and-deserialize-binary-tree)
33. Lowest Common Ancestor of a BST (#235)
    - 🎥 [Lowest Common Ancestor of a Binary Search Tree - Leetcode 235 - Python](https://www.youtube.com/watch?v=gs2LMfuOR9k) (NeetCode)
    - 📖 [NeetCode: recursive and iterative BST-property-based approaches](https://neetcode.io/solutions/lowest-common-ancestor-of-a-binary-search-tree)
34. Validate Binary Search Tree (#98)
    - 🎥 [Validate Binary Search Tree - Depth First Search - Leetcode 98](https://www.youtube.com/watch?v=s6ATEkipzow) (NeetCode)
    - 📖 [NeetCode: brute-force O(n²) subtree checks vs. optimal O(n) DFS with value-range bounds](https://neetcode.io/solutions/validate-binary-search-tree)
35. Kth Smallest Element in a BST (#230)
    - 🎥 [Kth Smallest Element in a BST - Leetcode 230 - Python](https://www.youtube.com/watch?v=5LUXSvjmGCw) (NeetCode)
    - 📖 [NeetCode: brute-force sort, inorder traversal, optimal DFS with early stop, and Morris traversal](https://neetcode.io/solutions/kth-smallest-element-in-a-bst)
### Heap / Priority Queue (1)


36. Find Median from Data Stream (#295)
    - 🎥 [Find Median from Data Stream - Heap & Priority Queue - Leetcode 295](https://www.youtube.com/watch?v=itmhHWaHupI) (NeetCode)
    - 📖 [NeetCode: brute-force sorting vs. optimized two-heap (max-heap/min-heap) approach](https://neetcode.io/solutions/find-median-from-data-stream)
### Backtracking (2)


37. Combination Sum (#39)
    - 🎥 [Combination Sum - Backtracking - Leetcode 39 - Python](https://www.youtube.com/watch?v=GBKI9VSKdGg) (NeetCode)
    - 📖 [NeetCode: basic include/skip recursion vs. optimized sorted-array backtracking with pruning](https://neetcode.io/solutions/combination-sum)
38. Word Search (#79)
    - 🎥 [Word Search - Backtracking - Leetcode 79 - Python](https://www.youtube.com/watch?v=pfiQ_PS1g8E) (NeetCode)
    - 📖 [NeetCode: hash-set vs. visited-array vs. optimal in-place cell-marking approaches](https://neetcode.io/solutions/word-search)
### Tries (3)


39. Implement Trie (Prefix Tree) (#208)
    - 🎥 [Implement Trie (Prefix Tree) - Leetcode 208](https://www.youtube.com/watch?v=oobqoCJlHA0) (NeetCode)
    - 📖 [NeetCode: array-based (26 children) vs. hash-map-based Trie node implementations](https://neetcode.io/solutions/implement-trie-prefix-tree)
40. Design Add and Search Words Data Structure (#211)
    - 🎥 [Design Add and Search Words Data Structure - Leetcode 211 - Python](https://www.youtube.com/watch?v=BTf05gs_8iU) (NeetCode)
    - 📖 [NeetCode: brute-force check-every-word vs. optimized Trie + DFS wildcard approach](https://neetcode.io/solutions/design-add-and-search-words-data-structure)
41. Word Search II (#212)
    - 🎥 [Word Search II - Backtracking Trie - Leetcode 212 - Python](https://www.youtube.com/watch?v=asbcE9mZz_U) (NeetCode)
    - 📖 [NeetCode: brute-force per-word backtracking vs. optimized shared-Trie backtracking with pruning](https://neetcode.io/solutions/word-search-ii)
### Graphs (6)


42. Number of Islands (#200)
    - 🎥 [NUMBER OF ISLANDS - Leetcode 200 - Python](https://www.youtube.com/watch?v=pV2kpPD66nE) (NeetCode)
    - 📖 [NeetCode: DFS, BFS, and Union-Find approaches](https://neetcode.io/solutions/number-of-islands)
43. Clone Graph (#133)
    - 🎥 [Clone Graph - Depth First Search - Leetcode 133](https://www.youtube.com/watch?v=mQeF6bN8hMk) (NeetCode)
    - 📖 [NeetCode: DFS (with hash map) and BFS approaches](https://neetcode.io/solutions/clone-graph)
44. Pacific Atlantic Water Flow (#417)
    - 🎥 [Pacific Atlantic Water Flow - Leetcode 417 - Python](https://www.youtube.com/watch?v=s-VkcjHqkGI) (NeetCode)
    - 📖 [NeetCode: brute-force backtracking vs. optimized reverse-flow DFS/BFS](https://neetcode.io/solutions/pacific-atlantic-water-flow)
45. Course Schedule (#207)
    - 🎥 [Course Schedule - Graph Adjacency List - Leetcode 207](https://www.youtube.com/watch?v=EgI5nU9etnU) (NeetCode)
    - 📖 [NeetCode: DFS cycle detection and Kahn's (BFS) topological sort](https://neetcode.io/solutions/course-schedule)
46. Graph Valid Tree (#261, LeetCode Premium - video/article are still free)
    - 🎥 [Graph Valid Tree - Leetcode 261 - Python](https://www.youtube.com/watch?v=bXsUuownnoQ) (NeetCode)
    - 📖 [NeetCode: DFS cycle detection, BFS, and Union-Find approaches](https://neetcode.io/solutions/graph-valid-tree)
47. Number of Connected Components in an Undirected Graph (#323, LeetCode Premium - video/article are still free)
    - 🎥 [Number of Connected Components in an Undirected Graph - Union Find - Leetcode 323 - Python](https://www.youtube.com/watch?v=8f1XPm4WOUc) (NeetCode)
    - 📖 [NeetCode: DFS, BFS, and Disjoint Set Union (with path compression)](https://neetcode.io/solutions/number-of-connected-components-in-an-undirected-graph)
### Advanced Graphs (1)


48. Alien Dictionary (#269, LeetCode Premium - video/article are still free)
    - 🎥 [Alien Dictionary - Topological Sort - Leetcode 269 - Python](https://www.youtube.com/watch?v=6kTZYvNNyps) (NeetCode)
    - 📖 [NeetCode: post-order DFS topological sort and Kahn's BFS algorithm](https://neetcode.io/solutions/alien-dictionary)
### 1-D Dynamic Programming (10)


49. Climbing Stairs (#70)
    - 🎥 [Climbing Stairs - Dynamic Programming - Leetcode 70 - Python](https://www.youtube.com/watch?v=Y0lT9Fck7qI) (NeetCode)
    - 📖 [NeetCode: recursion through memoization, tabulation, space-optimized DP, and Binet's formula](https://neetcode.io/solutions/climbing-stairs)
50. Coin Change (#322)
    - 🎥 [Coin Change - Dynamic Programming Bottom Up - Leetcode 322](https://www.youtube.com/watch?v=H9bfqozjoqs) (NeetCode)
    - 📖 [NeetCode: recursion, top-down memoization, bottom-up tabulation, and BFS approaches](https://neetcode.io/solutions/coin-change)
51. Longest Increasing Subsequence (#300)
    - 🎥 [Longest Increasing Subsequence - Dynamic Programming - Leetcode 300](https://www.youtube.com/watch?v=cjWnW0hdF1Y) (NeetCode)
    - 📖 [NeetCode: recursion through memoization, bottom-up DP, up to O(n log n) binary search](https://neetcode.io/solutions/longest-increasing-subsequence)
52. Word Break (#139)
    - 🎥 [Word Break - Dynamic Programming - Leetcode 139 - Python](https://www.youtube.com/watch?v=Sx9NNgInc3A) (NeetCode)
    - 📖 [NeetCode: brute-force recursion, memoization, bottom-up DP, and Trie-based optimization](https://neetcode.io/solutions/word-break)
53. Combination Sum IV (#377)
    - 🎥 [Combination Sum IV - Dynamic Programming - Leetcode 377 - Python](https://www.youtube.com/watch?v=dw2nMCxG0ik) (NeetCode)
    - 📖 [NeetCode: recursion, top-down memoization, bottom-up DP (counting problem - not #39)](https://neetcode.io/solutions/combination-sum-iv)
54. House Robber (#198)
    - 🎥 [House Robber - Leetcode 198 - Python Dynamic Programming](https://www.youtube.com/watch?v=73r3KWiEvyk) (NeetCode)
    - 📖 [NeetCode: pure recursion, top-down memoization, bottom-up DP, space-optimized](https://neetcode.io/solutions/house-robber)
55. House Robber II (#213)
    - 🎥 [House Robber II - Dynamic Programming - Leetcode 213](https://www.youtube.com/watch?v=rWAJCfYYOvM) (NeetCode)
    - 📖 [NeetCode: reduces the circular problem to two House Robber I passes, with full DP progression](https://neetcode.io/solutions/house-robber-ii)
56. Decode Ways (#91)
    - 🎥 [Decode Ways - Dynamic Programming - Leetcode 91 - Python](https://www.youtube.com/watch?v=6aEyTjOwlJU) (NeetCode)
    - 📖 [NeetCode: brute-force recursion, top-down memoization, bottom-up DP, space-optimized DP](https://neetcode.io/solutions/decode-ways)
57. Maximum Product Subarray (#152)
    - 🎥 [Maximum Product Subarray - Dynamic Programming - Leetcode 152](https://www.youtube.com/watch?v=lXVy6YWFcRM) (NeetCode)
    - 📖 [NeetCode: brute force, sliding window, running max/min (Kadane's-style), prefix & suffix approaches](https://neetcode.io/solutions/maximum-product-subarray)
58. Partition Equal Subset Sum (#416)
    - 🎥 [Partition Equal Subset Sum - Dynamic Programming - Leetcode 416 - Python](https://www.youtube.com/watch?v=IsvocB5BJhw) (NeetCode)
    - 📖 [NeetCode: recursion through top-down/bottom-up/space-optimized DP, hash set, and bitset optimization](https://neetcode.io/solutions/partition-equal-subset-sum)
### 2-D Dynamic Programming (2)


59. Longest Common Subsequence (#1143)
    - 🎥 [Longest Common Subsequence - Dynamic Programming - Leetcode 1143](https://www.youtube.com/watch?v=Ua0GhsJSlWM) (NeetCode)
    - 📖 [NeetCode: recursion, top-down memoization, bottom-up 2D DP, space-optimized DP](https://neetcode.io/solutions/longest-common-subsequence)
60. Unique Paths (#62)
    - 🎥 [Unique Paths - Dynamic Programming - Leetcode 62](https://www.youtube.com/watch?v=IlEsdxuD4lY) (NeetCode)
    - 📖 [NeetCode: pure recursive brute force through DP variants up to a combinatorics closed-form solution](https://neetcode.io/solutions/unique-paths)
### Greedy (2)


61. Maximum Subarray (#53)
    - 🎥 [Maximum Subarray - Amazon Coding Interview Question - Leetcode 53 - Python](https://www.youtube.com/watch?v=5WZl3MMT0Eg) (NeetCode)
    - 📖 [NeetCode: brute force, recursion, DP variants, Kadane's algorithm, and divide & conquer](https://neetcode.io/solutions/maximum-subarray)
62. Jump Game (#55)
    - 🎥 [Jump Game - Greedy - Leetcode 55](https://www.youtube.com/watch?v=Yan0cv2cLy8) (NeetCode)
    - 📖 [NeetCode: brute-force recursion, top-down DP, bottom-up DP, and the O(n)/O(1) greedy solution](https://neetcode.io/solutions/jump-game)
### Intervals (5)


63. Insert Interval (#57)
    - 🎥 [Insert Interval - Leetcode 57 - Python](https://www.youtube.com/watch?v=A8NUOmlwOlM) (NeetCode)
    - 📖 [NeetCode: linear search, binary search, and greedy approaches](https://neetcode.io/solutions/insert-interval)
64. Merge Intervals (#56)
    - 🎥 [Merge Intervals - Sorting - Leetcode 56](https://www.youtube.com/watch?v=44H3cEC2fFM) (NeetCode)
    - 📖 [NeetCode: sorting, sweep line, and greedy approaches](https://neetcode.io/solutions/merge-intervals)
65. Non-overlapping Intervals (#435)
    - 🎥 [Non-Overlapping Intervals - Leetcode 435 - Python](https://www.youtube.com/watch?v=nONCGxWoUfM) (NeetCode)
    - 📖 [NeetCode: recursion, DP, and greedy approaches](https://neetcode.io/solutions/non-overlapping-intervals)
66. Meeting Rooms (#252, LeetCode Premium - video/article are still free)
    - 🎥 [Meeting Rooms - Leetcode 252 - Python](https://www.youtube.com/watch?v=PaJxqZVPhbg) (NeetCode)
    - 📖 [NeetCode: brute force pairwise vs. sort-and-check-adjacent](https://neetcode.io/solutions/meeting-rooms)
67. Meeting Rooms II (#253, LeetCode Premium - video/article are still free)
    - 🎥 [Meeting Rooms II - Leetcode 253 - Python](https://www.youtube.com/watch?v=FdzJmTCVyJU) (NeetCode)
    - 📖 [NeetCode: min-heap, sweep line, two-pointers, greedy event-counting](https://neetcode.io/solutions/meeting-rooms-ii)
### Math & Geometry (3)


68. Rotate Image (#48)
    - 🎥 [Rotate Image - Matrix - Leetcode 48](https://www.youtube.com/watch?v=fMSJSS7eO1w) (NeetCode)
    - 📖 [NeetCode: brute force new-matrix copy vs. in-place rotation vs. reverse-and-transpose](https://neetcode.io/solutions/rotate-image)
69. Spiral Matrix (#54)
    - 🎥 [Spiral Matrix - Leetcode 54 - Python](https://www.youtube.com/watch?v=ln5VV0E6lPk) (NeetCode)
    - 📖 [NeetCode: recursive ring peeling vs. iterative shrinking boundaries](https://neetcode.io/solutions/spiral-matrix)
70. Set Matrix Zeroes (#73)
    - 🎥 [Set Matrix Zeroes - In-place - Leetcode 73](https://www.youtube.com/watch?v=T41rL0L3Pnw) (NeetCode)
    - 📖 [NeetCode: brute force copy, marker arrays, O(1) in-place marker approach](https://neetcode.io/solutions/set-matrix-zeroes)
### Bit Manipulation (5)


71. Number of 1 Bits (#191)
    - 🎥 [Number of 1 Bits - Leetcode 191 - Python](https://www.youtube.com/watch?v=5Km3utixwZs) (NeetCode)
    - 📖 [NeetCode: bit-mask check all 32 positions vs. optimal n & (n-1) trick](https://neetcode.io/solutions/number-of-1-bits)
72. Counting Bits (#338)
    - 🎥 [Counting Bits - Dynamic Programming - Leetcode 338 - Python](https://www.youtube.com/watch?v=RyBM56RIWrM) (NeetCode)
    - 📖 [NeetCode: per-number bit checking vs. optimal O(n) DP](https://neetcode.io/solutions/counting-bits)
73. Reverse Bits (#190)
    - 🎥 [Reverse Bits - Binary - Leetcode 190 - Python](https://www.youtube.com/watch?v=UcoN6UjAI64) (NeetCode)
    - 📖 [NeetCode: brute-force string reversal vs. bit manipulation vs. divide-and-conquer](https://neetcode.io/solutions/reverse-bits)
74. Sum of Two Integers (#371)
    - 🎥 [Sum Of Two Integers - Leetcode 371 - Blind 75 Explained - Binary - Python](https://www.youtube.com/watch?v=_pUidg9gQyA) (NeetCode)
    - 📖 [NeetCode: naive +/- vs. 32-bit-loop XOR/AND carry vs. optimal iterative XOR-carry](https://neetcode.io/solutions/sum-of-two-integers)
75. Missing Number (#268)
    - 🎥 [Missing Number - Blind 75 - Leetcode 268 - Python](https://www.youtube.com/watch?v=WnPLSRLSANE) (NeetCode)
    - 📖 [NeetCode: sorting, hash set, XOR bit trick, sum-formula math](https://neetcode.io/solutions/missing-number)

Total: 75 problems across 18 categories.

\* One 1-D DP slot varies slightly by source - some Blind 75 mirrors swap in a different 10th problem here instead of Partition Equal Subset Sum. Not worth resolving precisely now; if it comes up, treat it as a bonus problem rather than a blocker.

Two categories (Stack, Heap/PQ) are single-problem - don't skip them, they're still load-bearing patterns, just less heavily tested in Blind 75 specifically.

## Phase 2 target: NeetCode 150 ("thorough" benchmark)

Once all 75 above are solved, extend to the remaining ~75 problems that take this list from Blind 75 to NeetCode 150 (adds depth to Tries, Backtracking, Intervals, Bit Manipulation, a full Math & Geometry section, and more Advanced Graphs coverage - see the earlier review for examples like LRU Cache, Meeting Rooms follow-ups, Design Twitter). Don't enumerate this list yet - revisit and pull the exact NeetCode 150 diff once Phase 1 is actually done, so it reflects whatever's current at that time.

## Status

As of 2026-08-05: Blind 75 order confirmed and locked in as Phase 1. **13/75 done** - Arrays & Hashing (8/8) and Two Pointers (3/3) categories complete, plus Best Time to Buy and Sell Stock (#121) and Longest Substring Without Repeating Characters (#3) in Sliding Window. Next up: Longest Repeating Character Replacement (Sliding Window, #3 in that category). Track progress here (mark categories/problems done) as sessions happen - don't let this drift out of sync within a single conversation.
