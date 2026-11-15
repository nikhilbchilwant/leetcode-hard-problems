# 483. Smallest Good Base

> Source: https://algo.monster/liteproblems/483

## Problem Statement

Given an integer `n` as a string, find the smallest base `k` (where `k ≥ 2`) such that when `n` is represented in base `k`, all its digits are 1's.

In other words, we need to find the smallest integer `k` where:
- `n = 1 + k + k² + k³ + ... + k^m` for some integer `m ≥ 0`
- This means `n` in base `k` would be written as "111...1" (all 1's)

For example:
- If `n = 13`, the smallest good base is `3` because `13` in base `3` equals `111` (since `1×3² + 1×3¹ + 1×3⁰ = 9 + 3 + 1 = 13`)
- If `n = 4681`, the smallest good base is `8` because `4681 = 1 + 8 + 8² + 8³ + 8⁴ = 1 + 8 + 64 + 512 + 4096 = 4681`

## Intuition

The key insight is recognizing that we're looking for a base `k` where `n = 1 + k + k² + ... + k^m`. This is a geometric series with sum formula: `n = (k^(m+1) - 1) / (k - 1)`.

Since we want the smallest base, we should start by checking if `n` can be represented with more digits (larger `m`), as having more 1's typically requires a smaller base. For instance, `111` in base 2 is smaller than `11` in base 6, even though both equal 7.

The maximum number of digits possible is around 63 because even in base 2 (the smallest valid base), we can't exceed the range of a 64-bit integer. So we iterate `m` from 63 down to 2.

For each fixed number of digits `m`, we need to find if there exists a base `k` such that `1 + k + k² + ... + k^m = n`. As `k` increases, this sum grows monotonically, making binary search perfect for finding the exact `k` value.

The search range for `k` is from 2 (minimum valid base) to `n-1` (maximum meaningful base, since in base `n` or higher, `n` would be represented as a single digit).

Why return `str(num - 1)` at the end? If no valid base is found for `m ≥ 2`, then we need `m = 1`, meaning `n = 1 + k`. This gives us `k = n - 1`, which always works as `n` in base `n-1` equals `11` (one `(n-1)` plus one unit).

This approach guarantees finding the smallest good base by systematically checking from the maximum possible number of digits down to the minimum.

## Solution Approach

The implementation uses binary search within a loop to systematically find the smallest good base, following the standard binary search template.

**Helper Function `calculate_sum(base, num_terms)`:**
This function calculates the geometric series sum `1 + base + base² + ... + base^num_terms`. It initializes `power = 1` (current power of base) and `total = 1` (sum starting with the first term). Then it iterates `num_terms` times, multiplying `power` by `base` to get the next power and adding it to the sum.

**Main Algorithm:**
1. Convert the string `n` to an integer `target` for numerical operations.

2. Iterate through possible digit counts `m` from 63 down to 2:
   - We start from 63 because that's approximately the maximum number of bits needed to represent large integers
   - We check larger `m` values first because more digits generally means a smaller base

3. For each `m`, use binary search with the standard template to find the base `k`:
   - Initialize: `left = 2`, `right = target - 1`, `first_true_index = -1`
   - The feasible condition is `calculate_sum(mid, m) >= target`
   - Binary search loop: `while left <= right`
     - Calculate midpoint: `mid = (left + right) // 2`
     - If `feasible(mid)`: store `first_true_index = mid`, then search for smaller bases (`right = mid - 1`)
     - Otherwise: search larger bases (`left = mid + 1`)

4. After binary search, check if `first_true_index != -1` and `calculate_sum(first_true_index, m) == target`:
   - If yes, we found an exact match! Return `str(first_true_index)` as the smallest good base for this `m`
   - If no exact match, continue to the next smaller `m`

5. If no base is found for any `m >= 2`, return `str(target - 1)`:
   - This handles the case where `m = 1`, meaning `n = 1 + k`
   - Therefore `k = n - 1`, and `n` in base `(n-1)` equals `11`

The algorithm efficiently finds the smallest good base by using the monotonic property of the geometric series and using binary search to quickly locate valid bases for each possible digit count.

## Example Walkthrough

Let's trace through finding the smallest good base for `n = "13"` using the standard binary search template.

**Step 1:** Convert string to integer: `target = 13`

**Step 2:** Try different digit counts `m` from 63 down to 2.

For `m = 63, 62, ..., 4`:
- When we calculate `calculate_sum(2, m)` for these values, we get sums much larger than 13
- Binary search finds no exact match for any of these `m` values
- So we continue to smaller `m` values

**Step 3:** When `m = 3` (looking for 4 digits):
- Initialize: `left = 2`, `right = 12`, `first_true_index = -1`
- Feasible condition: `calculate_sum(mid, 3) >= 13`

- First iteration: `mid = 7`
  - `calculate_sum(7, 3) = 1 + 7 + 49 + 343 = 400 >= 13` ✓ (feasible)
  - Update: `first_true_index = 7`, `right = 6`
- Second iteration: `mid = 4`
  - `calculate_sum(4, 3) = 1 + 4 + 16 + 64 = 85 >= 13` ✓ (feasible)
  - Update: `first_true_index = 4`, `right = 3`
- Third iteration: `mid = 2`
  - `calculate_sum(2, 3) = 1 + 2 + 4 + 8 = 15 >= 13` ✓ (feasible)
  - Update: `first_true_index = 2`, `right = 1`
- Loop ends: `left > right`
- Check: `calculate_sum(2, 3) = 15 ≠ 13`, so no exact match for `m = 3`

**Step 4:** When `m = 2` (looking for 3 digits):
- Initialize: `left = 2`, `right = 12`, `first_true_index = -1`
- Feasible condition: `calculate_sum(mid, 2) >= 13`

- First iteration: `mid = 7`
  - `calculate_sum(7, 2) = 1 + 7 + 49 = 57 >= 13` ✓ (feasible)
  - Update: `first_true_index = 7`, `right = 6`
- Second iteration: `mid = 4`
  - `calculate_sum(4, 2) = 1 + 4 + 16 = 21 >= 13` ✓ (feasible)
  - Update: `first_true_index = 4`, `right = 3`
- Third iteration: `mid = 2`
  - `calculate_sum(2, 2) = 1 + 2 + 4 = 7 < 13` ✗ (not feasible)
  - Update: `left = 3`
- Fourth iteration: `mid = 3`
  - `calculate_sum(3, 2) = 1 + 3 + 9 = 13 >= 13` ✓ (feasible)
  - Update: `first_true_index = 3`, `right = 2`
- Loop ends: `left > right`
- Check: `calculate_sum(3, 2) = 13 == 13` ✓ Exact match!

**Step 5:** Return `"3"`

Indeed, 13 in base 3 equals `111` because `1×3² + 1×3¹ + 1×3⁰ = 9 + 3 + 1 = 13`.

## Python Solution

```python
class Solution:
    def smallestGoodBase(self, n: str) -> str:
        def calculate_sum(base: int, num_terms: int) -> int:
            """
            Calculate the sum: 1 + base + base^2 + ... + base^num_terms
            """
            power = 1
            total = 1
            for _ in range(num_terms):
                power *= base
                total += power
            return total

        def feasible(base: int, num_terms: int, target: int) -> bool:
            """Check if sum of powers >= target (feasible condition)"""
            return calculate_sum(base, num_terms) >= target

        target = int(n)

        # Try different digit counts from largest to smallest
        for num_terms in range(63, 1, -1):
            # Binary search template: find first base where sum >= target
            left, right = 2, target - 1
            first_true_index = -1

            while left <= right:
                mid = (left + right) // 2
                if feasible(mid, num_terms, target):
                    first_true_index = mid
                    right = mid - 1  # Search for smaller base
                else:
                    left = mid + 1

            # Check if we found an exact match
            if first_true_index != -1 and calculate_sum(first_true_index, num_terms) == target:
                return str(first_true_index)

        # Fallback: m = 1 case, n = 1 + (n-1)
        return str(target - 1)
```

## Time and Space Complexity

**Time Complexity:** `O(log³(n))`

The outer loop tries lengths `m` from 63 down to 2. Lengths above `log₂(n)` cannot work, so the useful part of this loop is `O(log(n))` iterations. For each `m`, a binary search over `[2, n-1]` takes `O(log(n))` iterations. Each iteration calls `calculate_sum`, which performs `m` multiplications and additions, `O(log(n))` work. The total is `O(log(n) × log(n) × log(n)) = O(log³(n))`. Since `n ≤ 10^18`, all three factors are at most about 63, so the running time is small in practice.

**Space Complexity:** `O(1)`

The algorithm uses only a constant number of variables: `target`, `num_terms`, `left`, `right`, `mid`, `first_true_index`, and `power` and `total` inside `calculate_sum`. No data structure grows with the input. (In Python, intermediate sums for large `m` can be big integers with more digits than `n`, which this analysis treats as fixed-size values.)

## Common Pitfalls

### 1. **Mixing Binary Search Template Variants**
The loop `while left < right` with `right = mid` and the loop `while left <= right` with `right = mid - 1` are both valid ways to find the first feasible base. Combining parts of them is a bug.

**Incorrect (mixed template):**
```python
while left < right:
    mid = (left + right) // 2
    if calculate_sum(mid, num_terms) >= target:
        right = mid - 1  # Can step past the answer
    else:
        left = mid + 1
return left
```

For `n = 40` (which is `1111` in base 3) and `m = 3`, the range starts as `[2, 39]`. The midpoints 20, 10, and 5 are all feasible, so `right` drops to 4. Then `mid = 3` is feasible (sum exactly 40), and `right = mid - 1 = 2` skips it. The loop ends with `left = 2`, whose sum is 15, so the exact-match check fails for `m = 3`. No other length matches, and the function returns `"39"` instead of `"3"`.

**Correct (standard template):**
```python
first_true_index = -1
while left <= right:
    mid = (left + right) // 2
    if feasible(mid, num_terms, target):
        first_true_index = mid
        right = mid - 1
    else:
        left = mid + 1
```

The standard template with `first_true_index` clearly tracks whether a valid answer was found and handles edge cases consistently.

### 2. **Integer Overflow in Power Calculation**
When `base` and `num_terms` are large, computing `base^num_terms` can exceed the maximum integer value, causing incorrect results or runtime errors.

**Solution:**
Add overflow checking to prevent invalid calculations:
```python
def calculate_sum(base: int, num_terms: int) -> int:
    power = 1
    total = 1
    for _ in range(num_terms):
        if power > (10**18) // base:
            return float('inf')
        power *= base
        total += power
    return total
```

### 3. **Not Checking for Exact Match**
The binary search finds the first base where `sum >= target`, but we need an **exact match** where `sum == target`. Forgetting this check leads to incorrect results.

**Solution:**
After binary search completes, always verify:
```python
if first_true_index != -1 and calculate_sum(first_true_index, num_terms) == target:
    return str(first_true_index)
```

### 4. **Edge Case: n = "3"**
The code returns `str(target - 1)` when no base is found for `m >= 2`. This handles `m = 1` case where `n = 1 + k`, giving `k = n - 1`. For n = "3", this correctly returns "2" since 3 in base 2 is "11".

