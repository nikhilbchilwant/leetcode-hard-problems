# 2954. Count the Number of Infection Sequences

> Source: https://algo.monster/liteproblems/2954

## Problem Statement

You have a line of `n` people where some are initially infected (sick). The positions of initially infected people are given in the array `sick` (sorted in increasing order).

The infection spreads step by step: at each step, exactly one uninfected person who is adjacent to an infected person becomes infected. This process continues until everyone in the line is infected.

An **infection sequence** represents the order in which the uninfected people become infected (not including those who were initially infected).

Your task is to find the total number of different possible infection sequences, returning the answer modulo `10^9 + 7`.

For example, if you have people at positions 0, 1, 2, 3, 4 and positions 1 and 3 are initially sick, then positions 0, 2, and 4 need to get infected. The infection can spread in different orders:
- Position 2 could get infected first (from position 1 or 3), then position 0 (from position 1), then position 4 (from position 3)
- Position 0 could get infected first (from position 1), then position 2 (from position 1 or 3), then position 4 (from position 3)
- And several other possible sequences

## Intuition

Let's think about how the infection spreads. The initially sick people create "barriers" that divide the healthy people into separate segments. For instance, if we have 10 people and positions 3 and 7 are sick, we get three segments: [0,1,2], [4,5,6], and [8,9].

The key observation is that people in different segments can get infected in an interleaved manner. If segment A has 3 people to infect and segment B has 2 people, we need to arrange these 5 infections in some order - this is like arranging 5 items where 3 are of type A and 2 are of type B. The number of such arrangements is `5!/(3! × 2!)`, which is the multinomial coefficient.

For the general case with multiple segments having `nums[0], nums[1], ..., nums[k]` healthy people, the total number of ways to interleave their infections is:
`s! / (nums[0]! × nums[1]! × ... × nums[k]!)`
where `s` is the total number of healthy people.

But wait, there's more! Within each segment, the infection can spread in different ways. Consider a middle segment (not at the ends) with 4 healthy people. The infection can come from both sides. At each step, we choose whether to infect from the left or right end:
- First infection: choose left or right (2 choices)
- Second infection: choose left or right (2 choices)  
- Third infection: choose left or right (2 choices)
- Fourth infection: only one person left (no choice)

This gives us `2^3 = 2^(4-1)` ways for a segment of length 4. Generally, a middle segment of length `x` has `2^(x-1)` infection patterns.

However, the leftmost and rightmost segments are special - they can only be infected from one side (the side with the sick person), so they each have only 1 way to be infected.

Combining both ideas:
1. The multinomial coefficient for interleaving: `s! / ∏(nums[i]!)`
2. The infection patterns within middle segments: `∏(2^(nums[i]-1))` for middle segments only

The final answer is the product of these two values, taken modulo `10^9 + 7`.

**Learn more about [Math](/problems/math-basics) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation follows the mathematical formula we derived:

`(s! / ∏(nums[i]!)) × ∏(2^(nums[i]-1))` for middle segments

**Step 1: Precompute Factorials**
```python
mod = 10**9 + 7
mx = 10**5
fac = [1] * (mx + 1)
for i in range(2, mx + 1):
    fac[i] = fac[i - 1] * i % mod
```
We precompute factorials up to `10^5` since that's the maximum possible value of `n`. This allows us to quickly access `k!` for any `k`.

**Step 2: Calculate Segment Sizes**
```python
nums = [b - a - 1 for a, b in pairwise([-1] + sick + [n])]
```
This clever line calculates the size of each healthy segment. By adding `-1` at the start and `n` at the end to the `sick` array, we can use `pairwise` to get consecutive pairs and calculate the gaps between them. For example, if `sick = [3, 7]` and `n = 10`, we get pairs `(-1, 3)`, `(3, 7)`, `(7, 10)`, giving us segments of size `3-(-1)-1 = 3`, `7-3-1 = 3`, `10-7-1 = 2`.

**Step 3: Calculate the Multinomial Coefficient**
```python
s = sum(nums)
ans = fac[s]
for x in nums:
    if x:
        ans = ans * pow(fac[x], mod - 2, mod) % mod
```
We start with `s!` (total permutations) and divide by each `nums[i]!`. Since we're working modulo a prime, division is done using the modular multiplicative inverse: `a/b ≡ a × b^(-1) ≡ a × b^(mod-2)` (by Fermat's Little Theorem).

**Step 4: Account for Internal Segment Choices**
```python
for x in nums[1:-1]:
    if x > 1:
        ans = ans * pow(2, x - 1, mod) % mod
```
For each middle segment (excluding first and last segments, hence `nums[1:-1]`), we multiply by `2^(x-1)` to account for the different ways infection can spread within that segment. We only do this when `x > 1` because a segment of size 0 or 1 has only one way to be infected.

**Key Implementation Details:**
- **Modular Arithmetic**: All operations are done modulo `10^9 + 7` to prevent overflow
- **Modular Inverse**: Using Fermat's Little Theorem, `a^(-1) ≡ a^(p-2) (mod p)` when `p` is prime
- **Fast Power**: The `pow(base, exp, mod)` function efficiently computes `base^exp % mod`

The time complexity is `O(n)` for preprocessing factorials and `O(k)` for the main computation where `k` is the number of segments, giving us `O(n)` overall.

## Example Walkthrough

Let's walk through a concrete example to illustrate the solution approach.

**Example:** `n = 7` people at positions `[0, 1, 2, 3, 4, 5, 6]`, with `sick = [1, 4]` (positions 1 and 4 are initially infected).

**Step 1: Identify the healthy segments**

The sick people at positions 1 and 4 divide the line into segments:
- Segment 0: Position `[0]` - leftmost segment (1 person)
- Segment 1: Positions `[2, 3]` - middle segment (2 people)  
- Segment 2: Positions `[5, 6]` - rightmost segment (2 people)

Using the formula: `nums = [b - a - 1 for a, b in pairwise([-1] + sick + [n])]`
- Pairs: `(-1, 1)`, `(1, 4)`, `(4, 7)`
- Segment sizes: `[1-(-1)-1 = 1, 4-1-1 = 2, 7-4-1 = 2]`
- So `nums = [1, 2, 2]`

**Step 2: Calculate the multinomial coefficient**

Total healthy people: `s = 1 + 2 + 2 = 5`

The multinomial coefficient represents how we can interleave the infections:
- `5! / (1! × 2! × 2!) = 120 / (1 × 2 × 2) = 30`

This means there are 30 ways to arrange 5 infections where 1 comes from segment 0, 2 from segment 1, and 2 from segment 2.

**Step 3: Account for internal choices in middle segments**

- Segment 0 (leftmost): Can only be infected from the right (position 1), so 1 way
- Segment 1 (middle): Has 2 people, can be infected from both sides
  - Can infect position 2 first (from position 1), then position 3 (from position 4)
  - OR infect position 3 first (from position 4), then position 2 (from position 1)
  - Total: `2^(2-1) = 2` ways
- Segment 2 (rightmost): Can only be infected from the left (position 4), so 1 way

**Step 4: Combine the results**

Total infection sequences = (multinomial coefficient) × (internal choices for middle segments)
= `30 × 2 = 60`

**Verification with a few sequences:**
Here are some of the 60 possible infection sequences (showing who gets infected at each step):
1. `[0, 2, 3, 5, 6]` - leftmost first, then middle left-to-right, then rightmost
2. `[0, 3, 2, 5, 6]` - leftmost first, then middle right-to-left, then rightmost
3. `[2, 0, 5, 3, 6]` - middle-left, leftmost, rightmost-left, middle-right, rightmost-right
4. `[5, 2, 6, 0, 3]` - rightmost-left, middle-left, rightmost-right, leftmost, middle-right

Each represents a valid infection sequence where at every step, exactly one healthy person adjacent to an infected person becomes infected.

The final answer would be `60 % (10^9 + 7) = 60`.

## Python Solution

```python
from typing import List

# Constants for modular arithmetic
MOD = 10**9 + 7
MAX_N = 10**5

# Precompute factorials for combinatorial calculations
factorial = [1] * (MAX_N + 1)
for i in range(2, MAX_N + 1):
    factorial[i] = factorial[i - 1] * i % MOD


class Solution:
    def numberOfSequence(self, n: int, sick: List[int]) -> int:
        """
        Calculate the number of valid infection sequences.
      
        Args:
            n: Total number of positions
            sick: List of initially sick positions (0-indexed)
      
        Returns:
            Number of valid sequences modulo 10^9 + 7
        """
        # Calculate gaps between consecutive sick positions
        # Add boundaries at -1 and n to handle edge cases
        extended_sick = [-1] + sick + [n]
        gaps = []
        for i in range(len(extended_sick) - 1):
            gap_size = extended_sick[i + 1] - extended_sick[i] - 1
            gaps.append(gap_size)
      
        # Total number of healthy positions to be infected
        total_healthy = sum(gaps)
      
        # Start with total permutations of all healthy positions
        result = factorial[total_healthy]
      
        # Divide by factorial of each gap size (multinomial coefficient)
        # This accounts for the ordering within each gap
        for gap_size in gaps:
            if gap_size > 0:
                # Compute modular inverse using Fermat's little theorem
                inverse = pow(factorial[gap_size], MOD - 2, MOD)
                result = result * inverse % MOD
      
        # For internal gaps (not at boundaries), multiply by 2^(gap_size-1)
        # This accounts for the bidirectional spread within each gap
        for gap_size in gaps[1:-1]:  # Exclude first and last gaps
            if gap_size > 1:
                result = result * pow(2, gap_size - 1, MOD) % MOD
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(m)`, where `m` is the length of the array `sick`.

The algorithm performs the following operations:
- Creating the `nums` array using `pairwise` iteration: `O(m)` 
- Computing the sum of `nums`: `O(m)`
- First loop iterating through `nums` for modular inverse calculations: `O(m)`, where each `pow(fac[x], mod - 2, mod)` operation takes `O(log(mod))` time
- Second loop iterating through `nums[1:-1]` for power calculations: `O(m)`, where each `pow(2, x - 1, mod)` operation takes `O(log(x))` time

Since `mod` is a constant (`10^9 + 7`) and `x` is bounded by `n`, the overall time complexity is `O(m)`.

**Space Complexity:** `O(m)` (excluding the preprocessing factorial array)

The algorithm uses:
- `nums` array which stores `m + 1` elements: `O(m)`
- A few scalar variables (`ans`, `s`): `O(1)`

The preprocessing factorial array `fac` of size `mx + 1` is not counted as it's computed outside the solution class. Therefore, the space complexity is `O(m)`.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Modular Division**
One of the most common mistakes is attempting direct division in modular arithmetic:
```python
# WRONG - This doesn't work with modular arithmetic
result = (factorial[total] // factorial[gap_size]) % MOD
```

**Why it fails:** Division doesn't distribute over modulo operation. `(a/b) % p ≠ (a % p) / (b % p)`.

**Solution:** Use modular multiplicative inverse with Fermat's Little Theorem:
```python
# CORRECT - Using modular inverse
inverse = pow(factorial[gap_size], MOD - 2, MOD)
result = result * inverse % MOD
```

### 2. **Off-by-One Errors in Gap Calculation**
A frequent mistake is incorrectly calculating the gap sizes between sick positions:
```python
# WRONG - Forgets to subtract 1 for the sick positions themselves
gaps = [extended_sick[i + 1] - extended_sick[i] for i in range(len(extended_sick) - 1)]

# WRONG - Incorrect boundary handling
gaps = [sick[i + 1] - sick[i] - 1 for i in range(len(sick) - 1)]  # Misses edge gaps
```

**Solution:** Always subtract 1 to exclude the sick positions and properly handle boundaries:
```python
# CORRECT
extended_sick = [-1] + sick + [n]
gaps = [extended_sick[i + 1] - extended_sick[i] - 1 for i in range(len(extended_sick) - 1)]
```

### 3. **Applying 2^(gap-1) to Boundary Segments**
A critical error is multiplying by `2^(gap-1)` for ALL segments, including those at the boundaries:
```python
# WRONG - Applies to all gaps including boundaries
for gap_size in gaps:
    if gap_size > 1:
        result = result * pow(2, gap_size - 1, MOD) % MOD
```

**Why it fails:** Boundary segments (leftmost and rightmost) can only be infected from one direction, not both.

**Solution:** Only apply to internal segments:
```python
# CORRECT - Skip first and last gaps
for gap_size in gaps[1:-1]:
    if gap_size > 1:
        result = result * pow(2, gap_size - 1, MOD) % MOD
```

### 4. **Integer Overflow Without Modulo**
Forgetting to apply modulo at each step can cause overflow:
```python
# WRONG - Can overflow before taking modulo
result = factorial[total]
for gap_size in gaps:
    result = result * pow(factorial[gap_size], MOD - 2, MOD)  # Missing % MOD
result = result % MOD  # Too late!
```

**Solution:** Apply modulo after every multiplication:
```python
# CORRECT
result = factorial[total]
for gap_size in gaps:
    result = result * inverse % MOD  # Apply modulo immediately
```

### 5. **Handling Empty Gaps Incorrectly**
When two sick positions are adjacent (gap size = 0), special care is needed:
```python
# WRONG - Division by factorial[0] without checking
for gap_size in gaps:
    result = result * pow(factorial[gap_size], MOD - 2, MOD) % MOD
```

**Solution:** Skip gaps of size 0:
```python
# CORRECT
for gap_size in gaps:
    if gap_size > 0:  # Only process non-empty gaps
        result = result * pow(factorial[gap_size], MOD - 2, MOD) % MOD
```

