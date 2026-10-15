# Find the Duplicate Number

*LeetCode 287, three approaches with values as comments*

## The problem

You get an array `nums` with `n + 1` numbers, and every number is between `1` and `n`. Exactly **one value is repeated** (it can appear more than twice). Find it.

The follow-up asks for this **without modifying the array** and with **O(1) extra space**.

`nums = [1, 3, 4, 2, 2]` gives `2`.

## Approach 1: Hash set (simplest, O(n) space)

Same "notebook" idea as Two Sum: the first number you see a second time is the duplicate.

```python
def find_duplicate(nums):
    seen = set()                    # seen = {}
    for num in nums:
        if num in seen:
            return num
        seen.add(num)

print(find_duplicate([1, 3, 4, 2, 2]))   # 2
```

```python
# num = 1: is 1 in {} ?            no -> seen = {1}
# num = 3: is 3 in {1} ?           no -> seen = {1, 3}
# num = 4: is 4 in {1, 3} ?        no -> seen = {1, 3, 4}
# num = 2: is 2 in {1, 3, 4} ?     no -> seen = {1, 3, 4, 2}
# num = 2: is 2 in {1, 3, 4, 2} ?  YES -> return 2
```

**Time:** O(n). **Space:** O(n). It works, but it breaks the O(1) space follow-up.

## Approach 2: Sorting (O(1) space, but changes the array)

Sort, then look for two equal neighbors.

```python
def find_duplicate(nums):
    nums.sort()                         # [1, 3, 4, 2, 2] -> [1, 2, 2, 3, 4]
    for i in range(1, len(nums)):
        if nums[i] == nums[i - 1]:      # i = 1: 2 == 1 ? no
            return nums[i]              # i = 2: 2 == 2 ? yes -> return 2
```

**Time:** O(n log n). It modifies the input, which the follow-up forbids.

## Approach 3: Floyd's cycle detection (the interview answer)

**O(n) time, O(1) space, no changes to the array.**

### The trick

Treat the array as a map: from index `i`, jump to index `nums[i]`.

`nums = [1, 3, 4, 2, 2]` gives:

```
index:   0  1  2  3  4
nums:    1  3  4  2  2

0 -> 1 -> 3 -> 2 -> 4 -> 2 -> 4 -> 2 ...
               ^_________|
               a cycle: 2 -> 4 -> 2
```

Two different indices (here `3` and `4`) both point to `2`. That is what the duplicate does: it creates a **loop**, and the duplicate value is the **entrance of that loop**. Every number is at least `1`, so nothing points back to index `0`, which means `0` is always outside the loop and a safe starting point.

So the problem becomes: **find where the loop starts.** That is the classic "linked list cycle II" problem.

### Code

```python
def find_duplicate(nums):
    # Phase 1: find a meeting point inside the loop
    slow = 0
    fast = 0
    while True:
        slow = nums[slow]               # moves 1 jump
        fast = nums[nums[fast]]         # moves 2 jumps
        if slow == fast:
            break

    # Phase 2: find the loop entrance (the duplicate)
    slow = 0                            # restart one pointer from the beginning
    while slow != fast:
        slow = nums[slow]               # both move 1 jump now
        fast = nums[fast]
    return slow

print(find_duplicate([1, 3, 4, 2, 2]))   # 2
```

### Values as comments

`nums = [1, 3, 4, 2, 2]`

```python
# ---------- Phase 1 ----------
# slow = 0, fast = 0

# Round 1: slow = nums[0] = 1,  fast = nums[nums[0]] = nums[1] = 3   -> 1 != 3
# Round 2: slow = nums[1] = 3,  fast = nums[nums[3]] = nums[2] = 4   -> 3 != 4
# Round 3: slow = nums[3] = 2,  fast = nums[nums[4]] = nums[2] = 4   -> 2 != 4
# Round 4: slow = nums[2] = 4,  fast = nums[nums[4]] = nums[2] = 4   -> 4 == 4, they meet at 4

# ---------- Phase 2 ----------
# slow = 0 (reset), fast = 4 (stays at the meeting point)

# Round 1: slow = nums[0] = 1,  fast = nums[4] = 2   -> 1 != 2
# Round 2: slow = nums[1] = 3,  fast = nums[2] = 4   -> 3 != 4
# Round 3: slow = nums[3] = 2,  fast = nums[4] = 2   -> 2 == 2, they meet at 2

# return 2
```

| Phase | Round | slow | fast |
|---|---|---|---|
| 1 | start | 0 | 0 |
| 1 | 1 | 1 | 3 |
| 1 | 2 | 3 | 4 |
| 1 | 3 | 2 | 4 |
| 1 | 4 | 4 | 4 (meet) |
| 2 | start | 0 | 4 |
| 2 | 1 | 1 | 2 |
| 2 | 2 | 3 | 4 |
| 2 | 3 | 2 | 2 (meet, **answer**) |

`fast = nums[nums[fast]]` is **two jumps** written as one expression: the inner `nums[fast]` is the first jump, and the outer `nums[...]` is the second.

### Why phase 2 works

Phase 1 only proves a loop exists, and the meeting point is somewhere inside it, not necessarily at the entrance. The math of the two speeds works out so that the distance from the start to the loop entrance equals the distance from the meeting point to the loop entrance (going around the loop). So if one pointer restarts at the beginning and both move one jump at a time, they land on the entrance together.

If you do not need to prove it in an interview, remember it as: **"find where they meet, then restart one pointer and walk both slowly."**

## Worked example: `nums = [3, 1, 3, 4, 2]`

```
index:   0  1  2  3  4
nums:    3  1  3  4  2
```

**Hash set**

```python
# num = 3: is 3 in {} ?       no  -> seen = {3}
# num = 1: is 1 in {3} ?      no  -> seen = {3, 1}
# num = 3: is 3 in {3, 1} ?   YES -> duplicate is 3
```

**Jump path** (start at index `0`, jump to `nums[current index]`)

| Step | At index | Value there | Jump to |
|---|---|---|---|
| 1 | 0 | 3 | index 3 |
| 2 | 3 | 4 | index 4 |
| 3 | 4 | 2 | index 2 |
| 4 | 2 | 3 | index 3 (already visited) |

```
0 -> 3 -> 4 -> 2 -> 3 -> 4 -> 2 ...
     ^___________|
     the loop is 3 -> 4 -> 2 -> 3
```

The loop starts at `3`, which is the duplicate. Index `0` and index `2` both hold the value `3`, so both jump to index `3`. A node with two ways in is exactly where a loop begins.

**Floyd's pointers**

```python
# ---------- Phase 1 ----------
# slow = 0, fast = 0

# Round 1: slow = nums[0] = 3,  fast = nums[nums[0]] = nums[3] = 4   -> 3 != 4
# Round 2: slow = nums[3] = 4,  fast = nums[nums[4]] = nums[2] = 3   -> 4 != 3
# Round 3: slow = nums[4] = 2,  fast = nums[nums[3]] = nums[4] = 2   -> 2 == 2, meet at 2

# ---------- Phase 2 ----------
# slow = 0 (reset), fast = 2

# Round 1: slow = nums[0] = 3,  fast = nums[2] = 3   -> 3 == 3, meet at 3

# return 3
```

| Round | slow (1 jump) | fast (2 jumps) | Meet? |
|---|---|---|---|
| 1 | 0 -> **3** | 0 -> 3 -> **4** | no |
| 2 | 3 -> **4** | 4 -> 2 -> **3** | no |
| 3 | 4 -> **2** | 3 -> 4 -> **2** | yes, at 2 |

All three methods agree on **3**.

## Comparison

| Approach | Time | Space | Modifies array? |
|---|---|---|---|
| Hash set | O(n) | O(n) | no |
| Sorting | O(n log n) | O(1) | yes |
| Floyd's cycle | O(n) | O(1) | no |

## What to say in an interview

> "A hash set gives O(n) time but O(n) space. Sorting gives O(1) space but changes the array. Because the values are in `1..n` and there are `n + 1` of them, I can treat the array as a linked list where `i` points to `nums[i]`. The duplicate is where two nodes point to the same place, which creates a cycle, so I use Floyd's algorithm to find the cycle entrance. That is O(n) time and O(1) space without modifying the input."

## Practice

Find the duplicate with the hash set, then write the jump path starting at index `0`.

```python
nums = [1, 4, 4, 2, 3]
nums = [2, 5, 9, 6, 9, 3, 8, 9, 7, 1]
nums = [2, 2, 2, 2, 2]
```
