# DSA Notes

A collection of interview-style notes and Python implementations for common data structures and algorithm patterns.

## Topics covered

- [Kadane's Algorithm](./Kadane's_Algorithm.md)  
  Maximum subarray, streak logic, and when Kadane is the right tool.

- [Two Pointers and Hashing](./Two_pointers_and_Hashing.md)  
  Pair-sum patterns, two-pointer strategies, and hash-based solutions.

- [Find the Duplicate Number](./find_the_duplicate_number.md)  
  Multiple approaches, including Floyd's cycle detection for O(1) extra space.

- [Product of Array Except Self](./product_of_array_except_self.md)  
  Prefix/suffix product approach and the O(1) extra-space optimization.

- [Best Time to Buy and Sell Stock](./best_time_to_buy_and_sell_stock.md)  
  Single-transaction stock profit problem solved with a minimum-price scan.

- [Best Time to Buy and Sell Stock II](./stock_buy_sell_multiple_transactions.md)  
  Greedy approach for maximizing profit across multiple transactions.

- [Even/Odd Checker and Binary](./even_odd_and_binary_notes.md)  
  Bitwise and modulo checks, plus an intuitive look at binary place values.

## What this repo is for

This repository is meant to make classic DSA ideas easier to understand by combining:

- plain-English explanations
- worked examples with values shown step by step
- Python code
- interview-style reasoning and trade-offs

## How to use it

Open each file in order of the topic you want to study, or use the list above as a quick map.

## Example

```python
# Kadane's Algorithm example

def kadane(nums):
    streak = nums[0]
    best = nums[0]
    for x in nums[1:]:
        streak = max(x, streak + x)
        best = max(best, streak)
    return best

print(kadane([3, -4, 2, -1, 6, -2, -5, 4]))  # 7
```

## Notes

These are learning notes for interview preparation and algorithm understanding, not a full competitive-programming library.
