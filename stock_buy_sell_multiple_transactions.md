# Best Time to Buy and Sell Stock II (Multiple Transactions)

*LeetCode 122, greedy: collect every upward step, with values as comments*

## The problem

`prices[i]` is the price on day `i`. You may buy and sell **as many times as you like**, but you can hold **only one share at a time** (sell before you buy again). Return the maximum total profit.

`prices = [7, 1, 5, 3, 6, 4]` gives `7`:

- Buy at 1, sell at 5: **+4**
- Buy at 3, sell at 6: **+3**
- Total: **7**

With only one transaction (LC 121) the same array gives 5, so allowing more trades changes the answer.

## The idea: collect every upward step

Look at the price **day to day**:

- Price goes **up** from yesterday to today: you could have held through it, so add the gain.
- Price goes **down** or stays flat: skip it. You would not hold through a drop.

```
prices:   7   1   5   3   6   4
change:     -6  +4  -2  +3  -2
              ↓   ↑   ↓   ↑   ↓
add:           0   4   0   3   0   = 7
```

## Code

```python
def max_profit(prices):
    profit = 0                                  # profit = 0
    for i in range(1, len(prices)):
        if prices[i] > prices[i - 1]:           # price rose since yesterday
            profit += prices[i] - prices[i - 1]
    return profit

print(max_profit([7, 1, 5, 3, 6, 4]))   # 7
```

Shorter version, same logic:

```python
def max_profit(prices):
    return sum(max(0, prices[i] - prices[i - 1]) for i in range(1, len(prices)))
```

## Values as comments

`prices = [7, 1, 5, 3, 6, 4]`

```python
profit = 0                                      # profit = 0

# i = 1: prices[1] - prices[0] = 1 - 7 = -6
#   1 > 7 ? no -> skip                          profit = 0

# i = 2: prices[2] - prices[1] = 5 - 1 = 4
#   5 > 1 ? yes -> profit += 4                  profit = 4

# i = 3: prices[3] - prices[2] = 3 - 5 = -2
#   3 > 5 ? no -> skip                          profit = 4

# i = 4: prices[4] - prices[3] = 6 - 3 = 3
#   6 > 3 ? yes -> profit += 3                  profit = 7

# i = 5: prices[5] - prices[4] = 4 - 6 = -2
#   4 > 6 ? no -> skip                          profit = 7

# return 7
```

| i | Yesterday | Today | Change | Rose? | profit |
|---|---|---|---|---|---|
| 1 | 7 | 1 | -6 | no | 0 |
| 2 | 1 | 5 | +4 | **yes** | 4 |
| 3 | 5 | 3 | -2 | no | 4 |
| 4 | 3 | 6 | +3 | **yes** | 7 |
| 5 | 6 | 4 | -2 | no | 7 |

## Why adding every up-step is correct

Say the price climbs over three days: `1, 3, 6`.

- Hold once from day 0 to day 2: `6 - 1 = 5`
- Add each step: `(3 - 1) + (6 - 3) = 2 + 3 = 5`

They match, because the middle price cancels out. So buying at the bottom and selling at the top of a long climb equals collecting each daily gain. You never need to find the exact buy and sell days.

## Which days to buy and sell

The profit above does not say which days. Track the start of each climb:

```python
def max_profit_trades(prices):
    profit = 0
    trades = []
    i = 0
    while i < len(prices) - 1:
        while i < len(prices) - 1 and prices[i + 1] <= prices[i]:
            i += 1                               # walk down to a valley
        buy = i
        while i < len(prices) - 1 and prices[i + 1] > prices[i]:
            i += 1                               # walk up to a peak
        if i > buy:
            profit += prices[i] - prices[buy]
            trades.append((buy, i))
    return profit, trades

print(max_profit_trades([7, 1, 5, 3, 6, 4]))   # (7, [(1, 2), (3, 4)])
```

```python
# i = 0: prices[1]=1 <= prices[0]=7 ? yes -> i = 1        (valley at day 1)
# buy = 1
# prices[2]=5 > prices[1]=1 ? yes -> i = 2
# prices[3]=3 > prices[2]=5 ? no  -> stop                 (peak at day 2)
# profit += 5 - 1 = 4, trades = [(1, 2)]

# prices[3]=3 <= prices[2]=5 ? yes -> i = 3               (valley at day 3)
# buy = 3
# prices[4]=6 > prices[3]=3 ? yes -> i = 4
# prices[5]=4 > prices[4]=6 ? no  -> stop                 (peak at day 4)
# profit += 6 - 3 = 3, trades = [(1, 2), (3, 4)]

# i = 4 < 5 -> prices[5]=4 <= prices[4]=6 ? yes -> i = 5  (end of array)
# loop ends -> return (7, [(1, 2), (3, 4)])
```

Buy on day 1 and sell on day 2 (+4), then buy on day 3 and sell on day 4 (+3). The first version is shorter and is the one to give in an interview.

## More examples

| prices | Up-steps | Answer |
|---|---|---|
| `[1, 2, 3, 4, 5]` | +1 +1 +1 +1 | **4** (same as buy at 1, sell at 5) |
| `[7, 6, 4, 3, 1]` | none | **0** (never trade) |
| `[3, 3, 3]` | none | **0** |
| `[1, 5, 2, 6]` | +4, +4 | **8** (buy 1 sell 5, buy 2 sell 6) |

## Compare with the one-transaction version

| | LC 121 (one transaction) | LC 122 (multiple) |
|---|---|---|
| Idea | Track the cheapest price so far | Add every upward step |
| Tracks | `min_price` and `profit` | Only `profit` |
| `[7, 1, 5, 3, 6, 4]` | 5 | **7** |
| Time / Space | O(n) / O(1) | O(n) / O(1) |

## Edge cases

- **Empty or one-day array:** the loop does not run, profit is `0`.
- **Only falling prices:** nothing is added, profit is `0`.
- **Flat prices:** `>` is false, nothing is added.

## What to say in an interview

> "Since I can trade as often as I want, any price rise can be captured by buying the day before and selling the day after. So I add up every positive day-to-day difference. That is a greedy O(n) time, O(1) space solution, and it matches buying at each valley and selling at each peak."

## Practice

Find the best total profit by hand, then check with the code:

```python
prices = [3, 8, 2, 5, 9, 1]
prices = [10, 9, 8, 7]
prices = [2, 4, 1, 7, 5, 3, 6]
```
