# 12. Best Time to Buy and Sell Stock

**LeetCode:** [#121 - Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) · **Topic:** [Sliding Window](../topics/03-sliding-window.md) · **Difficulty:** Easy

## Problem statement

Given an array `prices` where `prices[i]` is the stock price on day `i`, find the maximum profit from buying on one day and selling on a **later** day. If no profit is possible, return `0`. (You may only make one transaction: buy once, sell once.)

**Example:**
```
Input: prices = [7,1,5,3,6,4]
Output: 5
Explanation: Buy on day 1 (price 1), sell on day 4 (price 6): profit = 6 - 1 = 5.
```

## Applicable approaches

- **Brute Force** — check every pair of (buy day, sell day).
- **Optimal — One-Pass / Sliding Window (track minimum-so-far)** — the standard, expected O(n) solution.

## Approach 1: Brute Force

### Intuition

The most direct reading of the problem: try every possible pair of a buy day and a later sell day, compute the profit for each, and track the best one. Every valid transaction is a `(buy day, sell day)` pair with `buy < sell`, and this checks all of them, so it's correct by exhaustion — the entire question is whether we actually need to check all `O(n²)` such pairs, which the optimal approach answers with "no."

### Algorithm

1. For every pair `i < j`, compute `prices[j] - prices[i]`.
2. Track the maximum such value (but never let the answer go below 0, since we can choose not to transact).

### Python code
```python
def maxProfit(prices):
    n = len(prices)
    best = 0
    for i in range(n):
        for j in range(i + 1, n):
            profit = prices[j] - prices[i]
            best = max(best, profit)
    return best
```

### Line-by-line explanation

- `for i in range(n)` — every possible buy day.
- `for j in range(i + 1, n)` — every possible sell day *after* the buy day (the problem requires selling strictly later than buying).
- `profit = prices[j] - prices[i]` — profit from this specific buy/sell pair (can be negative if the price dropped, e.g. buying high and being forced to consider selling low within this exhaustive check).
- `best = max(best, profit)` — keep the best profit found; since `best` starts at 0, a negative profit never "wins," correctly representing "do nothing" as always an available option — this is a subtle but important detail: the brute force *does* consider transactions that lose money, but the `max` with an initial 0 silently discards them from ever becoming the answer.

### Time & space complexity

- **Time: O(n²)** — every pair of buy/sell days checked, since there's no way (yet) to know which buy days are worth pairing with which sell days without comparing them.
- **Space: O(1)**.

---

## Approach 2: Optimal — One-Pass, Track Minimum-So-Far

### Intuition

The brute force's waste: for every possible sell day, it re-checks *every* earlier day as a candidate buy day, even though only **one** of those earlier days could ever be optimal — the one with the lowest price so far. This is the key realization: **to maximize `sell_price - buy_price` for any fixed sell day, the best possible buy day is always the lowest price seen at any point *before* it** — there is never a reason to consider a higher earlier price when a lower one is available, because a lower buy price strictly increases the profit for the same sell price. So instead of checking every earlier day for every sell day (O(n) work repeated n times), we can walk through the prices once, always remembering the lowest price seen so far, and at each day, ask only one question: "if I sold today, having bought at the cheapest point so far, what would my profit be?"

This can be thought of as a degenerate sliding window: a window that only ever *grows* from its lowest-price anchor forward, never needing an explicit shrink step, because the "left edge" of the window is redefined implicitly every time a new minimum is found, rather than removed element-by-element the way a general sliding window's `left` pointer would be.

### Algorithm

1. Track `min_price_so_far`, initialized to infinity (so the very first real price is guaranteed to be lower).
2. Track `best_profit`, initialized to 0.
3. Walk through the prices in order:
   - Update `best_profit = max(best_profit, price - min_price_so_far)` — "if I sold today, having bought at the cheapest point so far, what's my profit?"
   - Update `min_price_so_far = min(min_price_so_far, price)` — if today's price is even lower than what we've seen, it becomes the new best possible buy point for future days.
4. Return `best_profit`.

### Python code
```python
def maxProfit(prices):
    min_price_so_far = float("inf")
    best_profit = 0

    for price in prices:
        best_profit = max(best_profit, price - min_price_so_far)
        min_price_so_far = min(min_price_so_far, price)

    return best_profit
```

### Line-by-line explanation

- `min_price_so_far = float("inf")` — starts as "infinitely high" so that the very first real price will always be lower and correctly become the initial minimum, without needing a special-cased first iteration.
- `best_profit = 0` — if no profitable transaction exists anywhere in the array, we correctly return 0 (never transacting is always a valid, zero-cost option).
- `for price in prices` — walk through the prices in order, left to right (day by day) — **the order matters here in a way it didn't for, say, Contains Duplicate**: "buy before sell" is a directional constraint, so this must be a single forward pass, not something order-independent.
- `best_profit = max(best_profit, price - min_price_so_far)` — **this line runs *before* updating `min_price_so_far` for the current day, and that ordering is the entire correctness argument.** It guarantees we're always computing "sell today, having bought on some *earlier* day" — never accidentally using today's own price as both the buy and sell price (which would trivially always give profit 0, and worse, would allow "buying and selling on the same day," which the problem doesn't permit).
- `min_price_so_far = min(min_price_so_far, price)` — now update the running minimum, so *future* days can consider buying at today's price if it turns out to be the best seen yet.

### Dry run

`prices = [7, 1, 5, 3, 6, 4]`

| price | best_profit = max(best_profit, price - min_price_so_far) | min_price_so_far = min(min_price_so_far, price) |
|---|---|---|
| 7 | max(0, 7 - inf) = 0 | min(inf, 7) = 7 |
| 1 | max(0, 1 - 7) = max(0, -6) = 0 | min(7, 1) = 1 |
| 5 | max(0, 5 - 1) = max(0, 4) = 4 | min(1, 5) = 1 |
| 3 | max(4, 3 - 1) = max(4, 2) = 4 | min(1, 3) = 1 |
| 6 | max(4, 6 - 1) = max(4, 5) = 5 | min(1, 6) = 1 |
| 4 | max(5, 4 - 1) = max(5, 3) = 5 | min(1, 4) = 1 |

Final `best_profit = 5` ✅ (correctly found: buy at price 1 on day index 1, sell at price 6 on day index 4). Notice `min_price_so_far` locks onto `1` after day index 1 and never updates again (since no later price is lower) — every subsequent profit calculation correctly uses that same anchor, which is exactly the "window that only grows from its lowest anchor" framing from the intuition.

### Time & space complexity

- **Time: O(n)** — a single pass through the prices, O(1) work per day.
- **Space: O(1)** — only two running variables, regardless of array size.

---

## Common mistakes & misconceptions

1. **Updating `min_price_so_far` *before* computing `best_profit` on the same iteration.** This would allow today's own price to be used as its own buy price (profit always 0 for that day, masking a potential earlier better answer only by coincidence of order, not because it's actually forbidden) — worse, in edge cases it can compute a nonsensical "buy today, sell today" comparison instead of the intended "buy on some earlier day." The order shown above (compute profit first, then update the minimum) is what enforces "sell after buy" correctly.
2. **Initializing `min_price_so_far` to `prices[0]` and starting the loop from index 1, versus initializing to infinity and starting from index 0.** Both are correct if done consistently, but mixing conventions (e.g. initializing to `prices[0]` *and* still iterating from index 0) double-processes the first day and can produce subtly wrong results in edge cases — pick one convention and verify it against a 1-element or 2-element input.
3. **Confusing this with "Maximum Subarray" (Kadane's algorithm, covered later in Greedy).** They look structurally similar (both are single-pass, both track a running best), but this problem tracks a running *minimum* to compute a *difference*, while Maximum Subarray tracks a running *sum* directly. Don't reuse Kadane's exact update rule here without re-deriving why a different quantity needs to be tracked.
4. **Assuming multiple transactions are allowed.** This specific version of the problem (`#121`) permits exactly **one** buy and **one** sell — a closely related but different LeetCode problem (`#122`, not in this list) allows unlimited transactions and needs a different (though related) greedy strategy. Don't transplant a solution from one variant to the other without checking the constraint has actually changed.

## Summary

| Approach | Time | Space | Notes |
|---|---|---|---|
| Brute Force | O(n²) | O(1) | Checks every buy/sell pair, most of which are provably suboptimal. |
| One-Pass (min-so-far) | O(n) | O(1) | The standard, expected optimal solution. |

**Key takeaway:** whenever a problem asks "for each position, what's the best result using some earlier position?", check whether you can track the single best *relevant* earlier value (here: the minimum price so far) as you go, instead of re-scanning all earlier positions each time. This "running best, updated after use" trick — with the specific ordering discipline of "use the old value before updating it" — turns many O(n²) brute forces into O(n), and it's the same underlying idea whether the "best earlier value" is a minimum (as here) or a maximum.
