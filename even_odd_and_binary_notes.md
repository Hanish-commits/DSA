# Even/Odd Checker and Binary

*Check even or odd with `%` and `&`, and see why it works through binary*

## 1. Even or odd with modulo (`%`)

A number is **even** if dividing by 2 leaves remainder `0`, and **odd** if it leaves `1`.

```python
def check_even_odd(n):
    if n % 2 == 0:
        return "Even"
    return "Odd"

print(check_even_odd(10))   # Even
print(check_even_odd(7))    # Odd
```

```python
# check_even_odd(10): 10 % 2 = 0 -> 0 == 0 ? yes -> "Even"
# check_even_odd(7):   7 % 2 = 1 -> 1 == 0 ? no  -> "Odd"
# check_even_odd(0):   0 % 2 = 0 -> "Even"        (zero is even)
# check_even_odd(-3): -3 % 2 = 1 -> "Odd"         (negatives work in Python)
```

Test for `== 0`, not `== 1`. In C or Java, `-3 % 2` is `-1`, so `== 1` would call -3 even.

## 2. Even or odd with bitwise AND (`&`)

`n & 1` keeps only the **last bit** of `n`: `0` for even, `1` for odd.

```python
def check_even_odd(n):
    return "Odd" if n & 1 else "Even"      # 1 is truthy, 0 is falsy

print(check_even_odd(10))   # Even
print(check_even_odd(7))    # Odd
```

```
n = 7                    n = 10
  0 0 0 0 0 1 1 1          0 0 0 0 1 0 1 0
& 0 0 0 0 0 0 0 1        & 0 0 0 0 0 0 0 1
  ---------------          ---------------
  0 0 0 0 0 0 0 1 = 1      0 0 0 0 0 0 0 0 = 0
  -> odd                   -> even
```

**Parentheses tip:** in Python, `&` binds tighter than `==`, so `n & 1 == 0` already means `(n & 1) == 0`. Still write the parentheses: in C, Java and JavaScript `==` binds tighter than `&`, and the same line there silently means `n & (1 == 0)`.

```python
(n & 1) == 0     # clear in every language
```

### Filtering a list

```python
nums = [1, 2, 3, 4, 5, 6]
evens = [x for x in nums if (x & 1) == 0]     # [2, 4, 6]
odds  = [x for x in nums if (x & 1) == 1]     # [1, 3, 5]
```

### Input from the user

```python
n = int(input("Enter a number: "))            # types 15 -> n = 15
print(n, "is", "Odd" if n & 1 else "Even")    # 15 & 1 = 1 -> 15 is Odd
```

| | `n % 2` | `n & 1` |
|---|---|---|
| Readability | Clearer for most people | Needs bit knowledge |
| Negatives | Works | Works |
| Best use | Everyday code | Interviews, low-level code |

## 3. Binary: the place-value boxes

Binary uses digits 0 and 1, and each place is **double** the one on its right. Each box holds `1` (use this value) or `0` (skip it). Add the values over the `1`s to get the number. 8 boxes cover 0 to 255.

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 128 │ 64  │ 32  │ 16  │  8  │  4  │  2  │  1  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  ?  │  ?  │  ?  │  ?  │  ?  │  ?  │  ?  │  ?  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

## 4. n to binary

### Method 1: place-value boxes

Go left to right and ask **"does this value fit in what is left?"** Yes means put `1` and subtract. No means put `0`.

`n = 25`:

| Box value | Left | Fits? | Bit | Left after |
|---|---|---|---|---|
| 128 | 25 | no | 0 | 25 |
| 64 | 25 | no | 0 | 25 |
| 32 | 25 | no | 0 | 25 |
| 16 | 25 | yes | **1** | 9 |
| 8 | 9 | yes | **1** | 1 |
| 4 | 1 | no | 0 | 1 |
| 2 | 1 | no | 0 | 1 |
| 1 | 1 | yes | **1** | 0 |

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 128 │ 64  │ 32  │ 16  │  8  │  4  │  2  │  1  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  0  │  1  │  1  │  0  │  0  │  1  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
                                             ^
                                             1's place (decides even/odd)
```

**25 = 16 + 8 + 1 = `00011001`**. Same way: 10 = 8 + 2 = `00001010`, and 45 = 32 + 8 + 4 + 1 = `00101101`.

### Method 2: divide by 2

Keep dividing by 2 and write each remainder. Stop at 0, then read the remainders **bottom to top**.

```
 25 ÷ 2 = 12   remainder 1
 12 ÷ 2 =  6   remainder 0
  6 ÷ 2 =  3   remainder 0
  3 ÷ 2 =  1   remainder 1
  1 ÷ 2 =  0   remainder 1

Read remainders bottom to top: 11001
```

The first remainder is the 1's place (last box), which is why you read upward. `11001` fills the 8 boxes as `00011001`.

| | Place-value boxes | Divide by 2 |
|---|---|---|
| Best for | Small numbers, mental math | Big numbers |
| Needs | Memorized powers of 2 | Nothing |
| Read answer | Left to right | Bottom to top |

### Binary back to n

Add the box values that sit over a `1`:

```
┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
│ 128 │ 64  │ 32  │ 16  │  8  │  4  │  2  │  1  │
├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
│  0  │  0  │  1  │  0  │  0  │  1  │  0  │  1  │
└─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
```

`00100101` = 32 + 4 + 1 = **37**

## 5. Why the last box decides even or odd

Every box except the last has an **even** value (128, 64, 32, 16, 8, 4, 2), and even numbers always add up to an even number. Only the last box (worth 1) can make the total odd. That is exactly the bit `n & 1` reads, and the bit that alternates 0, 1, 0, 1 in the table below.

## 6. Quick reference and practice

| n | binary | even/odd |
|---|---|---|
| 0 | `0000` | even |
| 1 | `0001` | odd |
| 2 | `0010` | even |
| 3 | `0011` | odd |
| 4 | `0100` | even |
| 5 | `0101` | odd |
| 6 | `0110` | even |
| 7 | `0111` | odd |
| 8 | `1000` | even |
| 9 | `1001` | odd |
| 10 | `1010` | even |
| 11 | `1011` | odd |
| 12 | `1100` | even |
| 13 | `1101` | odd |
| 14 | `1110` | even |
| 15 | `1111` | odd |

**Cheat sheet**

- Box values: **128, 64, 32, 16, 8, 4, 2, 1** (each is double the last)
- Even or odd: `n % 2 == 0` or `(n & 1) == 0`
- n to binary: boxes (subtract what fits) or divide by 2 (read remainders upward)
- Binary to n: add the box values over the `1`s

**Practice:** convert `6`, `13`, `20`, `37`, `100` to binary, then convert `00001001`, `00110011`, `01100100` back to decimal. State whether each is even or odd just by looking at the last box.
