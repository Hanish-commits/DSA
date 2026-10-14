# Kadane's Algorithm: Complete Notes

## Contents

- Part 1: The Shopkeeper Story
- Part 2: What Problems Kadane's Algorithm Solves

---

# Part 1: The Shopkeeper Story

*Finding the best consecutive streak (Maximum Subarray, LeetCode 53)*

## The story

You run a small shop. Each day you either **earn** money (positive number) or **lose** money (negative number). You want the **best streak of consecutive days**, meaning the stretch with the biggest total profit.

Daily results: `[3, -4, 2, -1, 6, -2, -5, 4]`

## The one rule

At the start of each new day, ask:

> "Is my streak so far **worth keeping**, or should I **forget it and begin a new streak today**?"

- Streak total is **positive**: keep it, it gives you a head start.
- Streak total is **zero or negative**: forget it, it is only baggage.

Two things are tracked:

- `streak` is the total of the current streak.
- `best` is the biggest streak total ever seen (the trophy on the wall).

## Day by day

| Day | Result | Streak before | Decision | Streak after | Best |
|---|---|---|---|---|---|
| 1 | +3 | none | start | 3 | 3 |
| 2 | -4 | 3 (positive) | keep: 3 - 4 | -1 | 3 |
| 3 | +2 | -1 (negative) | forget, start fresh | 2 | 3 |
| 4 | -1 | 2 (positive) | keep: 2 - 1 | 1 | 3 |
| 5 | +6 | 1 (positive) | keep: 1 + 6 | 7 | **7** |
| 6 | -2 | 7 (positive) | keep: 7 - 2 | 5 | 7 |
| 7 | -5 | 5 (positive) | keep: 5 - 5 | 0 | 7 |
| 8 | +4 | 0 (adds nothing) | start fresh | 4 | 7 |

**Answer: 7.** The best streak is days 3 to 5: `[2, -1, 6]`, since 2 - 1 + 6 = 7.

## The code

```python
def kadane(days):
    streak = days[0]
    best = days[0]
    for d in days[1:]:
        streak = max(d, streak + d)   # keep streak, or start fresh today
        best = max(best, streak)      # remember the best streak ever
    return best

print(kadane([3, -4, 2, -1, 6, -2, -5, 4]))   # 7
```

`max(d, streak + d)` is the "keep or forget" question in one line: if `streak` is positive, `streak + d` is bigger (keep); if it is negative, `d` alone is bigger (start fresh).

## The code with values as comments

Using `days = [3, -4, 2, -1, 6, -2, -5, 4]`:

```python
def kadane(days):
    streak = days[0]        # streak = 3
    best = days[0]          # best = 3

    for d in days[1:]:      # d goes: -4, 2, -1, 6, -2, -5, 4

        # ---------- d = -4 ----------
        streak = max(d, streak + d)   # max(-4, 3 + -4) = max(-4, -1) = -1  (extend)
        best = max(best, streak)      # max(3, -1) = 3

        # ---------- d = 2 ----------
        # streak = max(2, -1 + 2) = max(2, 1) = 2     (start fresh, old streak was negative)
        # best   = max(3, 2) = 3

        # ---------- d = -1 ----------
        # streak = max(-1, 2 + -1) = max(-1, 1) = 1   (extend)
        # best   = max(3, 1) = 3

        # ---------- d = 6 ----------
        # streak = max(6, 1 + 6) = max(6, 7) = 7      (extend)
        # best   = max(3, 7) = 7                      (new record)

        # ---------- d = -2 ----------
        # streak = max(-2, 7 + -2) = max(-2, 5) = 5   (extend)
        # best   = max(7, 5) = 7

        # ---------- d = -5 ----------
        # streak = max(-5, 5 + -5) = max(-5, 0) = 0   (extend)
        # best   = max(7, 0) = 7

        # ---------- d = 4 ----------
        # streak = max(4, 0 + 4) = max(4, 4) = 4      (tie, either choice gives 4)
        # best   = max(7, 4) = 7

    return best             # 7

print(kadane([3, -4, 2, -1, 6, -2, -5, 4]))   # 7
```

### Same trace as a table

| d | streak + d | d alone | streak = max | best |
|---|-----------|---------|--------------|------|
| (start) | | | 3 | 3 |
| -4 | 3 - 4 = -1 | -4 | **-1** | 3 |
| 2 | -1 + 2 = 1 | 2 | **2** (fresh) | 3 |
| -1 | 2 - 1 = 1 | -1 | **1** | 3 |
| 6 | 1 + 6 = 7 | 6 | **7** | **7** |
| -2 | 7 - 2 = 5 | -2 | **5** | 7 |
| -5 | 5 - 5 = 0 | -5 | **0** | 7 |
| 4 | 0 + 4 = 4 | 4 | **4** | 7 |

### How to read it

- `streak + d` means "keep my streak and add today."
- `d` alone means "forget the past and start fresh today."
- `max(...)` picks whichever is bigger. `d` alone only wins when `streak` is negative, as at `d = 2`.
- `best` never goes down. It only changes when `streak` beats the old record, which happens once, at `d = 6`.

## Why this story helps

Imagine telling a friend: "I had a terrible stretch and I'm down $1 overall, but tomorrow I'm starting a new streak." That is exactly what the algorithm does. A negative past can never help a future streak, so it gets dropped.

`best` is the trophy on your wall. Even if the streak later collapses to 0, the trophy still says 7.

## Complexity

- **Time:** O(n), one pass.
- **Space:** O(1), just two variables.

## Edge case: all-negative arrays

`[-3, -1, -2]` should return `-1` (the least bad single day). That is why `streak` and `best` start at `days[0]` and not `0`.

## Try it yourself

`days = [-1, 5, -2, 3, -8, 4]`

Write down `streak` and `best` after each day, then check: the answer is **6**, from `[5, -2, 3]`.

---

# Part 2: What Problems Kadane's Algorithm Solves

*A quick guide to recognizing when to use Kadane's*

## The pattern to look for

Kadane's fits any problem that asks for the best **contiguous** chunk of an array, where "best" means the largest (or smallest) total. If you see "subarray," "consecutive," or "streak" plus "maximum sum," think Kadane.

## Direct uses

| Problem | What it asks |
|---|---|
| **Maximum Subarray** (LC 53) | Largest sum of a contiguous chunk. This is the base problem. |
| **Minimum subarray sum** | Smallest sum of a chunk. Use `min` instead of `max`, or negate the array and run Kadane. |
| **Best Time to Buy and Sell Stock** (LC 121) | Max profit from one buy and one sell. Turn prices into daily changes (`prices[i] - prices[i-1]`), and the answer is the max subarray sum of those changes. |
| **Maximum Absolute Sum of Any Subarray** (LC 1749) | Run Kadane for the max and for the min, then take the larger absolute value. |
| **Profit/loss streaks** | Best run of days, best stretch of a game score, best hot streak of any daily numbers. |

## Variations that need a small twist

- **Maximum Product Subarray** (LC 152): the same "extend or restart" idea, but track both the max and min product so far, because two negatives multiply to a positive.
- **Maximum Sum Circular Subarray** (LC 918): the chunk may wrap around the end. Answer = max of (normal Kadane) and (total sum minus the minimum subarray). Watch the all-negative case.
- **Maximum Subarray Sum with One Deletion** (LC 1186): keep two states, one with no deletion and one with a deletion used.
- **Return the subarray itself, not just the sum:** track a `start` index. Reset `start = i` whenever you start fresh, and save `(start, i)` whenever `best` improves.
- **Largest sum rectangle in a 2D grid:** fix a pair of rows, squash the columns into a 1D array, and run Kadane on it.

## Problems that look similar but are NOT Kadane

- **Subarray sum equals K:** use a prefix sum with a hash map.
- **Longest subarray with a condition:** usually sliding window.
- **Non-contiguous picks** (choose any elements): usually DP, like House Robber.

## Quick test to decide

Ask yourself two questions:

1. Must the elements be **next to each other**?
2. Does a bad (negative) past ever **hurt** a future chunk, so you'd want to drop it?

If both are yes, Kadane is likely the tool.

## Practice order

Start with LC 53, then LC 121, then LC 918.
