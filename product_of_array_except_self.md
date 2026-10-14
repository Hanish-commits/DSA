# Product of Array Except Self: Prefix and Suffix Products

*LeetCode 238, explained step by step with values as comments*

## The problem

Given `nums`, return an array where `answer[i]` is the product of **every number except `nums[i]`**. Division is not allowed, and it should run in O(n).

`nums = [1, 2, 3, 4]` gives `[24, 12, 8, 6]`.

- Skip `1`: 2 x 3 x 4 = **24**
- Skip `2`: 1 x 3 x 4 = **12**
- Skip `3`: 1 x 2 x 4 = **8**
- Skip `4`: 1 x 2 x 3 = **6**

The challenge is doing this fast, without redoing the multiplication from scratch for every number.

## The idea: left side times right side

When you skip a number, everything else is either **to its left** or **to its right**. So:

> answer[i] = (product of everything to the left of i) x (product of everything to the right of i)

If a side is empty, use **1**, because multiplying by 1 changes nothing.

- **Prefix product** at `i`: product of all elements before `i`.
- **Suffix product** at `i`: product of all elements after `i`.

## Step 1: Do it by hand

```
index:    0   1   2   3
nums:     1   2   3   4

prefix:   1   1   2   6      (product of everything to the left)
suffix:  24  12   4   1      (product of everything to the right)

answer:  24  12   8   6      (prefix x suffix)
```

| i | left product | right product | answer |
|---|---|---|---|
| 0 | 1 (nothing) | 2 x 3 x 4 = 24 | 1 x 24 = **24** |
| 1 | 1 | 3 x 4 = 12 | 1 x 12 = **12** |
| 2 | 1 x 2 = 2 | 4 | 2 x 4 = **8** |
| 3 | 1 x 2 x 3 = 6 | 1 (nothing) | 6 x 1 = **6** |

Each left product is the **previous left product x the previous number**, so you never start over. The right products work the same way, walking from right to left.

## Version 1: prefix and suffix arrays (easier to understand)

```python
def product_except_self(nums):
    n = len(nums)                       # n = 4
    prefix = [1] * n                    # prefix = [1, 1, 1, 1]
    suffix = [1] * n                    # suffix = [1, 1, 1, 1]

    # prefix[i] = product of nums[0 .. i-1]
    for i in range(1, n):
        prefix[i] = prefix[i - 1] * nums[i - 1]
        # i = 1: prefix[1] = prefix[0] * nums[0] = 1 * 1 = 1
        # i = 2: prefix[2] = prefix[1] * nums[1] = 1 * 2 = 2
        # i = 3: prefix[3] = prefix[2] * nums[2] = 2 * 3 = 6
        # prefix = [1, 1, 2, 6]

    # suffix[i] = product of nums[i+1 .. n-1]
    for i in range(n - 2, -1, -1):
        suffix[i] = suffix[i + 1] * nums[i + 1]
        # i = 2: suffix[2] = suffix[3] * nums[3] = 1 * 4  = 4
        # i = 1: suffix[1] = suffix[2] * nums[2] = 4 * 3  = 12
        # i = 0: suffix[0] = suffix[1] * nums[1] = 12 * 2 = 24
        # suffix = [24, 12, 4, 1]

    return [prefix[i] * suffix[i] for i in range(n)]
    # [1*24, 1*12, 2*4, 6*1] = [24, 12, 8, 6]

print(product_except_self([1, 2, 3, 4]))   # [24, 12, 8, 6]
```

**Time:** O(n). **Space:** O(n) for the two extra arrays.

## Version 2: O(1) extra space (the interview answer)

Skip the separate arrays. Store the prefix products directly in `answer`, then multiply the suffix products in using one running variable.

```python
def product_except_self(nums):
    n = len(nums)                       # n = 4
    answer = [1] * n                    # answer = [1, 1, 1, 1]

    # Pass 1, left to right: answer[i] = product of everything LEFT of i
    prefix = 1                          # prefix = 1
    for i in range(n):
        answer[i] = prefix
        prefix *= nums[i]
        # i = 0: answer[0] = 1,  prefix = 1 * 1 = 1
        # i = 1: answer[1] = 1,  prefix = 1 * 2 = 2
        # i = 2: answer[2] = 2,  prefix = 2 * 3 = 6
        # i = 3: answer[3] = 6,  prefix = 6 * 4 = 24
    # answer = [1, 1, 2, 6]

    # Pass 2, right to left: multiply in the product of everything RIGHT of i
    suffix = 1                          # suffix = 1
    for i in range(n - 1, -1, -1):
        answer[i] *= suffix
        suffix *= nums[i]
        # i = 3: answer[3] = 6 * 1  = 6,   suffix = 1 * 4  = 4
        # i = 2: answer[2] = 2 * 4  = 8,   suffix = 4 * 3  = 12
        # i = 1: answer[1] = 1 * 12 = 12,  suffix = 12 * 2 = 24
        # i = 0: answer[0] = 1 * 24 = 24,  suffix = 24 * 1 = 24
    # answer = [24, 12, 8, 6]

    return answer

print(product_except_self([1, 2, 3, 4]))   # [24, 12, 8, 6]
```

**Time:** O(n). **Space:** O(1) extra, because the output array does not count.

## Understanding the second loop (`answer[i] *= suffix`)

Before the second loop, `answer` already holds the **left products**:

```
nums:    [1, 2, 3, 4]
answer:  [1, 1, 2, 6]     <- left product of each position
```

`suffix` is one number: a running right product. It starts at `1` (nothing to the right of the last spot). Walking from right to left, at each spot:

1. **Use** it: `answer[i] *= suffix`
2. **Update** it: `suffix *= nums[i]`

The order matters. Multiply first, then update, because a spot's own number must **not** be part of its own right product.

| i | answer[i] before (left product) | suffix (right product) | answer[i] after | suffix after |
|---|---|---|---|---|
| 3 | 6 | 1 | 6 x 1 = **6** | 1 x 4 = 4 |
| 2 | 2 | 4 | 2 x 4 = **8** | 4 x 3 = 12 |
| 1 | 1 | 12 | 1 x 12 = **12** | 12 x 2 = 24 |
| 0 | 1 | 24 | 1 x 24 = **24** | 24 x 1 = 24 |

`answer[i] *= suffix` is shorthand for `answer[i] = answer[i] * suffix`.

The picture to remember: `answer` starts as the **left** half of each result, and `suffix` carries the **right** half as you walk backwards. Multiplying them gives the full answer at each spot.

## Why not just divide?

You could compute the total product and divide by `nums[i]`, but:

- The problem forbids division.
- It breaks on zeros: dividing by `0` crashes, and a single zero changes the answer for every other position.

Prefix and suffix never divide, so zeros need no special cases. Check `nums = [1, 0, 3, 4]`:

```
prefix:  1   1   0   0
suffix:  0  12   4   1
answer:  0  12   0   0
```

Only the position holding the zero keeps a non-zero product (12).

## Things to remember

- Answer at `i` = (everything to the left) x (everything to the right).
- Build the left side going forward, the right side going backward.
- Empty side = `1`, not `0`.
- Version 1 is easier to explain. Version 2 is the one to give when the interviewer asks for O(1) extra space.
- The same prefix and suffix idea appears in trapping rain water and prefix-sum problems.
