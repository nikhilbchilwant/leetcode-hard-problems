# 878. Nth Magical Number

> Source: https://algo.monster/liteproblems/878

## Problem Statement

A **magical number** is defined as a positive integer that is divisible by either `a` or `b`.

You are given three integers:
- `n`: the position we're looking for
- `a`: first divisor
- `b`: second divisor

Your task is to find the `n`-th smallest magical number. Since this number can be very large, return the result modulo `10^9 + 7`.

For example, if `a = 2` and `b = 3`, the magical numbers in order would be: 2, 3, 4, 6, 8, 9, 10, 12, ... (numbers divisible by 2 or 3).

## Intuition

The key insight is that we need to find the n-th number in a sequence where numbers are divisible by either `a` or `b`. Instead of generating all magical numbers one by one (which would be too slow), we can use a counting approach.

For any given number `x`, we can quickly calculate how many magical numbers exist from 1 to `x`:
- Numbers divisible by `a`: `x // a`
- Numbers divisible by `b`: `x // b`
- But wait - we've double-counted numbers divisible by both `a` and `b`!

This is where the inclusion-exclusion principle comes in. Numbers divisible by both `a` and `b` are exactly those divisible by `lcm(a, b)`. So the correct count is: `x // a + x // b - x // lcm(a, b)`.

Now we have a way to count magical numbers up to any value `x`. To find the n-th magical number, we need to find the smallest `x` such that there are exactly `n` magical numbers from 1 to `x`. This is a perfect scenario for [binary search](/problems/binary_search_intro)!

The search space can be bounded by observing that the n-th magical number cannot exceed `n * min(a, b)` (in the worst case, we'd take every multiple of the smaller number). The solution uses `n * min(a, b)` as the upper bound.

Binary searching with the counting function as the feasibility test, we can efficiently find the exact position where we have `n` magical numbers, giving us our answer.

**Learn more about [Math](/problems/math-basics) and [Binary Search](/problems/binary_search_intro) patterns.**

## Solution Approach

The solution implements a [binary search](/problems/binary_search_intro) on the answer, using a counting function to test each candidate value.

**Step 1: Calculate the Least Common Multiple (LCM)**
```python
lcm_value = (a * b) // math.gcd(a, b)
```
We need the LCM to apply the inclusion-exclusion principle correctly.

**Step 2: Define the Search Range**
```python
left, right = 1, n * min(a, b)
```
The multiples `min(a, b), 2·min(a, b), ..., n·min(a, b)` are already `n` magical numbers, so the n-th magical number is at most `n * min(a, b)`. This gives a safe search range from 1 to `right`.

**Step 3: [Binary Search](/problems/binary_search_intro) with a Feasibility Check**
```python
def feasible(x):
    count = x // a + x // b - x // lcm_value
    return count >= n
```
- `x // a`: count of numbers in `[1, x]` divisible by `a`
- `x // b`: count of numbers in `[1, x]` divisible by `b`
- `x // lcm_value`: count of numbers divisible by both (subtracted to remove double-counting)

`feasible(x)` is `False` for small `x` and `True` from the answer onward. The loop `while left <= right` tests `mid`; when `feasible(mid)` holds it records `first_true_index = mid` and searches left with `right = mid - 1`, otherwise it moves right with `left = mid + 1`. When the loop ends, `first_true_index` is the smallest `x` with at least `n` magical numbers in `[1, x]`. That `x` must itself be magical, because the count only increases at magical numbers.

**Step 4: Apply Modulo**
```python
return first_true_index % MOD
```
Since the answer can be very large, we return it modulo `10^9 + 7` as required.

## Example Walkthrough

Let's find the 4th magical number where `a = 2` and `b = 3`.

**Step 1: Calculate LCM**
- `lcm(2, 3) = 6` (since 2 and 3 are coprime)

**Step 2: Set up search range**
- `left = 1`, `right = 4 * min(2, 3) = 8`

**Step 3: Binary search process**

The counting function for any value `x` is: `x // 2 + x // 3 - x // 6`

Counts for several values:
- `x = 1`: `1//2 + 1//3 - 1//6 = 0 + 0 - 0 = 0` magical numbers
- `x = 2`: `2//2 + 2//3 - 2//6 = 1 + 0 - 0 = 1` magical number (just 2)
- `x = 3`: `3//2 + 3//3 - 3//6 = 1 + 1 - 0 = 2` magical numbers (2, 3)
- `x = 4`: `4//2 + 4//3 - 4//6 = 2 + 1 - 0 = 3` magical numbers (2, 3, 4)
- `x = 5`: `5//2 + 5//3 - 5//6 = 2 + 1 - 0 = 3` magical numbers (2, 3, 4)
- `x = 6`: `6//2 + 6//3 - 6//6 = 3 + 2 - 1 = 4` magical numbers (2, 3, 4, 6)

The binary search looks for the first `x` where the count reaches 4:
- `left = 1, right = 8`: `mid = 4`, count 3 < 4, so `left = 5`
- `left = 5, right = 8`: `mid = 6`, count 4 ≥ 4, so `first_true_index = 6`, `right = 5`
- `left = 5, right = 5`: `mid = 5`, count 3 < 4, so `left = 6`
- `left = 6 > right = 5`, so the loop ends with `first_true_index = 6`

**Step 4: Return result**
- The 4th magical number is 6
- Apply modulo: `6 % (10^9 + 7) = 6`

**Verification**: The magical numbers in order are 2, 3, 4, 6... and indeed the 4th one is 6.

The key insight is that we never generated the actual sequence - we just counted how many magical numbers exist up to each point and used binary search to find where that count equals n.

## Python Solution

```python
import math

class Solution:
    def nthMagicalNumber(self, n: int, a: int, b: int) -> int:
        """
        Find the nth magical number.
        A magical number is a positive integer that is divisible by either a or b.

        Args:
            n: The position of the magical number to find
            a: First divisor
            b: Second divisor

        Returns:
            The nth magical number modulo 10^9 + 7
        """
        MOD = 10**9 + 7

        # Calculate the least common multiple of a and b
        lcm_value = (a * b) // math.gcd(a, b)

        def feasible(x: int) -> bool:
            """Check if there are at least n magical numbers <= x."""
            count = x // a + x // b - x // lcm_value
            return count >= n

        # Binary search for the nth magical number
        # Search space: [1, n * min(a, b)]
        left, right = 1, n * min(a, b)
        first_true_index = -1

        while left <= right:
            mid = (left + right) // 2
            if feasible(mid):
                first_true_index = mid
                right = mid - 1
            else:
                left = mid + 1

        return first_true_index % MOD
```

## Time and Space Complexity

**Time Complexity:** `O(log(n * min(a, b)) + log(min(a, b)))`

The binary search runs over the range `[1, n * min(a, b)]`, so it performs `O(log(n * min(a, b)))` iterations. Each iteration evaluates `feasible`, which does three integer divisions and two additions or subtractions, all `O(1)`. The LCM is computed once before the search using `math.gcd`, which takes `O(log(min(a, b)))` time.

**Space Complexity:** `O(1)`

The algorithm uses only a constant number of variables (`MOD`, `lcm_value`, `left`, `right`, `mid`, `first_true_index`), and no data structures that grow with the input.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Search Range That Is Too Small**
The upper bound must be at least the answer. Using `n` or `max(a, b)` as the upper bound misses the answer for most inputs.

**Incorrect approach:**
```python
left, right = 1, n  # WRONG: the n-th magical number can be much larger than n
```

For `n = 4, a = 2, b = 3`, the answer is `6`, but the search never looks past `4`. With this bound the loop never finds a feasible value, and `first_true_index` stays `-1`.

**Correct approach:** Use `right = n * min(a, b)`, which is always at least the answer, because the first `n` multiples of `min(a, b)` are all magical:
```python
left, right = 1, n * min(a, b)
first_true_index = -1

while left <= right:
    mid = (left + right) // 2
    if count(mid) >= n:
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1

return first_true_index % MOD
```

The `while left < right` form with `right = mid` on a feasible value and `return left` is also a correct way to write the same search.

### 2. **Incorrect LCM Calculation Leading to Integer Overflow**
The naive formula `lcm = (a * b) // gcd(a, b)` can cause integer overflow during the multiplication step, especially in languages like Java/C++.

**Solution:**
```python
# Option 1: Use built-in function (Python 3.9+)
lcm_value = math.lcm(a, b)

# Option 2: Divide first to prevent overflow
gcd_value = math.gcd(a, b)
lcm_value = (a // gcd_value) * b
```

### 3. **Forgetting the Modulo Operation**
The problem explicitly asks for the result modulo `10^9 + 7`, but it's easy to forget this step since the binary search itself doesn't require it.

**Solution:**
Always apply the modulo operation before returning:
```python
MOD = 10**9 + 7
return first_true_index % MOD
```

### 4. **Edge Cases with first_true_index**
When using the template, always initialize `first_true_index = -1` and handle the case where no valid answer is found (though for this problem, an answer is guaranteed to exist).

