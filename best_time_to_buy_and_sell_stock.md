# Best Time to Buy and Sell Stock (One Transaction)

*LeetCode 121, single-pass tracking, with values as comments*

## The problem

`prices[i]` is the price of a stock on day `i`. You may **buy once and sell once**, and the sell must come **after** the buy. Return the maximum profit, or `0` if no profit is possible.

`prices = [7, 1, 5, 3, 6, 4]` gives `5` (buy at 1 on day 1, sell at 6 on day 4).

## Approach 1: Brute force (O(n²))

Try every buy day and every later sell day.

```python
def max_profit_brute(prices):
    best = 0
    for i in range(len(prices)):
        for j in range(i + 1, len(prices)):      # sell day is AFTER buy day
            best = max(best, prices[j] - prices[i])
    return best
```

Correct but slow for large inputs. State it first in an interview, then improve it.

## Approach 2: Single pass (O(n) time, O(1) space)

For each day, **pretend you sell today**. The best day to have bought is the **cheapest price seen so far**. Track two things:

- `min_price`: the lowest price seen so far (the best buy)
- `profit`: the best profit seen so far

```python
def max_profit(prices):
    min_price = prices[0]               # cheapest price so far
    profit = 0                          # best profit so far

    for price in prices[1:]:
        if price < min_price:
            min_price = price           # found a cheaper day to buy
        else:
            profit = max(profit, price - min_price)    # try selling today

    return profit

print(max_profit([7, 1, 5, 3, 6, 4]))   # 5
```

### Values as comments

`prices = [7, 1, 5, 3, 6, 4]`

```python
min_price = 7                           # min_price = 7
profit = 0                              # profit = 0

# price = 1
#   1 < 7 ? yes -> new cheapest day
#   min_price = 1                       (profit stays 0)

# price = 5
#   5 < 1 ? no -> try selling today
#   profit = max(0, 5 - 1) = max(0, 4) = 4

# price = 3
#   3 < 1 ? no
#   profit = max(4, 3 - 1) = max(4, 2) = 4

# price = 6
#   6 < 1 ? no
#   profit = max(4, 6 - 1) = max(4, 5) = 5      <- best

# price = 4
#   4 < 1 ? no
#   profit = max(5, 4 - 1) = max(5, 3) = 5

# return 5
```

| Day | Price | Cheapest so far | Profit if sold today | Best profit |
|---|---|---|---|---|
| 0 | 7 | 7 | none | 0 |
| 1 | 1 | **1** | none (new low) | 0 |
| 2 | 5 | 1 | 5 - 1 = 4 | 4 |
| 3 | 3 | 1 | 3 - 1 = 2 | 4 |
| 4 | 6 | 1 | 6 - 1 = **5** | **5** |
| 5 | 4 | 1 | 4 - 1 = 3 | 5 |

### When prices only fall

`prices = [7, 6, 4, 3, 1]`

```python
# 6 < 7 -> min_price = 6
# 4 < 6 -> min_price = 4
# 3 < 4 -> min_price = 3
# 1 < 3 -> min_price = 1
# profit never updated -> return 0
```

Every day is a new low, so there is never a day to sell for a gain. The answer is `0` (don't trade).

## Which days to buy and sell

Track the **index** of the cheapest day and save the pair whenever profit improves:

```python
def max_profit_days(prices):
    min_price = prices[0]
    min_day = 0
    profit = 0
    buy_day = sell_day = 0

    for i in range(1, len(prices)):
        if prices[i] < min_price:
            min_price = prices[i]
            min_day = i
        elif prices[i] - min_price > profit:
            profit = prices[i] - min_price
            buy_day, sell_day = min_day, i

    return profit, buy_day, sell_day

print(max_profit_days([7, 1, 5, 3, 6, 4]))   # (5, 1, 4)
```

## Why not just take the min and max of the array?

The sell must come **after** the buy. In `[7, 1, 5, 3, 6, 4]` the max is `7` on day 0, before the min. Taking `7 - 1` would mean selling before you bought. That is why the code only compares today's price against lows from **earlier** days.

## Link to Kadane's algorithm

Turn prices into daily changes, and the answer is the **maximum subarray sum** of those changes:

```
prices:   7   1   5   3   6   4
changes:    -6  +4  -2  +3  -2
```

The best streak is `+4, -2, +3 = 5`, the same answer. Buying on day 1 and selling on day 4 is the same as holding through those daily changes.

## "Single-pass tracking" is the same question

It is the name of the technique, not a different problem: one loop, keeping a running minimum and a running best profit.

## Other stock problems (do not mix them up)

| Problem | Transactions allowed | Approach |
|---|---|---|
| **LC 121 (this file)** | Exactly one buy and one sell | Single pass with `min_price` and `profit` |
| LC 122 | Unlimited | Add every upward step: `profit += max(0, prices[i] - prices[i-1])` |
| LC 123 | At most 2 | DP with states for the 1st and 2nd buy and sell |
| LC 188 | At most k | DP over k transactions |
| LC 309 | Unlimited, with a cooldown day after selling | DP with hold, sold and rest states |

Quick check with `prices = [7, 1, 5, 3, 6, 4]`:

- **One transaction (121):** buy at 1, sell at 6, profit **5**
- **Unlimited (122):** buy at 1, sell at 5 (+4), then buy at 3, sell at 6 (+3), profit **7**

## Edge cases

- **One day** (`[5]`): the loop doesn't run, profit is `0`.
- **Empty array:** add `if not prices: return 0` first.
- **Flat prices** (`[3, 3, 3]`): profit stays `0`.
- **Price rises every day** (`[1, 2, 3, 4]`): buy day 0, sell day 3, profit `3`.

## Complexity

| Approach | Time | Space |
|---|---|---|
| Brute force | O(n²) | O(1) |
| Single pass | **O(n)** | **O(1)** |

## What to say in an interview

> "Brute force tries every buy and sell pair in O(n²). Instead, for each day I assume I sell today, and the best buy is the cheapest price before it. So I keep a running minimum and a running best profit in one pass: O(n) time, O(1) space."

## Practice

Find the best profit, and the buy and sell days, by hand:

```python
prices = [3, 8, 2, 5, 9, 1]
prices = [10, 9, 8, 7]
prices = [2, 4, 1, 7, 5, 3, 6]
```
