# 829. Consecutive Numbers Sum

> Source: https://algo.monster/liteproblems/829

## Problem Statement

Given an integer `n`, you need to find how many different ways you can express `n` as a sum of consecutive positive integers.

For example, if `n = 5`, it can be written as:
- `5` (just the number itself)
- `2 + 3` (two consecutive numbers)

So there are 2 ways to write 5 as consecutive positive integers.

The problem asks you to count all possible ways to represent `n` as a sum of one or more consecutive positive integers. The consecutive integers must be positive (greater than 0) and must form an unbroken sequence like `a, a+1, a+2, ...` where `a` is the starting number.

The key insight is that any sum of consecutive integers forms an arithmetic sequence. If we have `k` consecutive numbers starting from `a`, the sum would be:
- `a + (a+1) + (a+2) + ... + (a+k-1) = n`

Using the arithmetic series formula, this sum equals `(2a + k - 1) × k / 2 = n`, which can be rearranged to `2n = (2a + k - 1) × k`.

From this equation, we can derive conditions that must be satisfied:
1. `k` must divide `2n` evenly (since the right side must equal `2n`)
2. Since `a ≥ 1`, we have `2n = (2a + k - 1) × k ≥ k × (k + 1)`, giving us an upper bound for `k`
3. From `2n/k = 2a + k - 1`, we get `2a = 2n/k - k + 1`, which must be even (since `2a` is even), meaning `2n/k - k + 1` must be even

## Intuition

When we think about writing a number as consecutive positive integers, we're essentially looking for arithmetic sequences that sum to `n`. Let's start with a simple example to build our intuition.

If we want to express `n` as `k` consecutive numbers starting from `a`, we'd have:
`a + (a+1) + (a+2) + ... + (a+k-1) = n`

Using the arithmetic series formula, this becomes:
`sum = (first term + last term) × number of terms / 2`
`n = (a + (a+k-1)) × k / 2`
`n = (2a + k - 1) × k / 2`

Multiplying both sides by 2:
`2n = (2a + k - 1) × k`

Now comes the key insight: we can rearrange this to solve for `a`:
`2n = 2ak + k² - k`
`2n = 2ak + k(k - 1)`
`2a = 2n/k - k + 1`

For `a` to be a positive integer, two things must happen:
1. `k` must divide `2n` evenly (otherwise `2n/k` wouldn't be an integer)
2. `2a` must be a positive even number, which means `2n/k - k + 1` must be even

Also, since `a` must be at least 1 (we need positive integers), we have:
`2a ≥ 2`
`2n/k - k + 1 ≥ 2`
`2n/k ≥ k + 1`
`2n ≥ k(k + 1)`

This gives us an upper bound for `k`: we only need to check values where `k(k + 1) ≤ 2n`.

The beautiful part of this approach is that instead of trying different starting points `a` and lengths `k` (which would be inefficient), we can iterate through possible lengths `k` and directly check if they lead to a valid starting point `a`. This transforms a potentially complex search problem into a simple iteration with mathematical checks.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

Based on our mathematical derivation, we implement the solution by checking each possible value of `k` (the number of consecutive integers) and verifying if it leads to a valid representation.

The implementation follows these steps:

1. **Double the value of n**: We start by computing `doubled_n = n << 1` (which is `n × 2`) since all our conditions involve `2n`.

2. **Initialize variables**: Set `count = 0` to count valid ways, and `sequence_length = 1` (called `k` below) to start checking from sequences of length 1.

3. **Iterate through possible values of k**: We loop while `k × (k + 1) ≤ 2n`. This bound comes from our constraint that `a ≥ 1`, which gives us `2n ≥ k × (k + 1)`.

4. **Check validity conditions**: For each `k`, we verify two conditions:
   - `doubled_n % k == 0`: This checks if `k` divides `2n` evenly
   - `(doubled_n // k - k + 1) % 2 == 0`: This ensures that `2n/k - k + 1` is even, which guarantees that `a` will be a positive integer

5. **Count valid representations**: If both conditions are satisfied, we increment `count` by 1.

6. **Move to next k**: Increment `k` and continue the loop.

The algorithm efficiently checks all possible lengths of consecutive sequences in `O(√n)` time complexity, since `k` can be at most around `√(2n)` based on the constraint `k × (k + 1) ≤ 2n`.

Here's how the code maps to our mathematical conditions:
- `doubled_n = n << 1` computes `2n`
- The while loop condition `k * (k + 1) <= doubled_n` ensures we stay within valid bounds
- `doubled_n % k == 0` checks divisibility
- `(doubled_n // k - k + 1) % 2 == 0` verifies the evenness condition for a valid starting point

This approach is elegant because it avoids the need to actually find the starting value `a` for each sequence - we just verify that a valid `a` exists through our mathematical conditions.

## Example Walkthrough

Let's walk through the solution with `n = 9` to see how we count the ways to express it as consecutive positive integers.

**Step 1: Double n**
- `n = 9 × 2 = 18` (we work with `2n` for our formula)

**Step 2: Check each possible length k**

For each `k`, we need:
- Condition 1: `k × (k + 1) ≤ 18` (upper bound)
- Condition 2: `18 % k == 0` (k divides 2n)
- Condition 3: `(18/k - k + 1) % 2 == 0` (ensures valid starting point)

**k = 1:**
- `1 × 2 = 2 ≤ 18` ✓
- `18 % 1 = 0` ✓
- `(18/1 - 1 + 1) = 18 % 2 = 0` ✓
- Valid! This represents `9` (single number)

**k = 2:**
- `2 × 3 = 6 ≤ 18` ✓
- `18 % 2 = 0` ✓
- `(18/2 - 2 + 1) = 8 % 2 = 0` ✓
- Valid! This represents `4 + 5 = 9`

**k = 3:**
- `3 × 4 = 12 ≤ 18` ✓
- `18 % 3 = 0` ✓
- `(18/3 - 3 + 1) = 4 % 2 = 0` ✓
- Valid! This represents `2 + 3 + 4 = 9`

**k = 4:**
- `4 × 5 = 20 > 18` ✗
- Stop here as we've exceeded our bound

**Result:** There are **3 ways** to express 9 as consecutive positive integers:
- `9` (one number)
- `4 + 5` (two consecutive numbers)
- `2 + 3 + 4` (three consecutive numbers)

The algorithm efficiently found all valid representations by checking mathematical conditions rather than trying all possible starting points.

## Python Solution

```python
class Solution:
    def consecutiveNumbersSum(self, n: int) -> int:
        # Double n to avoid floating point arithmetic
        # This allows us to work with 2n = k * (2x + k - 1)
        doubled_n = n << 1
      
        # Initialize counter for valid ways and starting sequence length
        count = 0
        sequence_length = 1
      
        # Iterate through possible sequence lengths
        # k * (k + 1) represents the minimum sum for k consecutive numbers starting from 1
        # If k * (k + 1) > 2n, then we can't form a valid sequence
        while sequence_length * (sequence_length + 1) <= doubled_n:
            # Check two conditions:
            # 1. doubled_n is divisible by sequence_length
            # 2. The starting number would be a positive integer
            #    (doubled_n // sequence_length - sequence_length + 1) must be even
            #    to ensure the starting number is an integer
            if (doubled_n % sequence_length == 0 and 
                (doubled_n // sequence_length - sequence_length + 1) % 2 == 0):
                count += 1
          
            sequence_length += 1
      
        return count
```

## Time and Space Complexity

The time complexity is `O(√n)`, where `n` is the given positive integer. The loop continues while `k * (k + 1) <= doubled_n`, that is, `k * (k + 1) <= 2n`. This inequality means `k² + k <= 2n`, so approximately `k² <= 2n`, giving us `k <= √(2n)`. Since the loop increments `k` from 1 until this condition is violated, it runs approximately `√(2n)` times, which is `O(√n)`.

The space complexity is `O(1)`. The algorithm only uses a constant amount of extra space for variables such as `doubled_n`, `count`, and `sequence_length`, regardless of the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow When Computing k × (k + 1)**

In languages with fixed-size integers, `2n` and `k * (k + 1)` must fit in the integer type. With the constraint `n <= 10^9`, `2n` is at most `2 × 10^9`, and the loop stops once `k * (k + 1)` exceeds `2n`, so the largest product computed is about `2 × 10^9 + 9 × 10^4`. Both fit in a signed 32-bit `int` (maximum `2,147,483,647`), so the Java, C++, and TypeScript versions are safe. If the bound on `n` were larger, these values would overflow and should be computed in 64-bit integers. Python integers never overflow.

**Caution when rewriting the bound:** Replacing the exact condition with a loose one such as `k <= int((2 * doubled_n) ** 0.5) + 1` is wrong. It lets the loop test lengths `k` whose starting value `a` would be zero or negative, and some of them pass the divisibility and parity checks. For `n = 9` that loop returns 4 instead of 3. Keep the exact condition `k * (k + 1) <= doubled_n`.

### 2. **Confusion About Why We Double n**

Developers often forget why we double `n` at the beginning and may accidentally use the wrong variable in calculations, leading to incorrect results. The doubling is done to avoid floating-point arithmetic when working with the formula `2n = k × (2a + k - 1)`.

**Solution:** Use clear variable names and maintain consistency:
```python
def consecutiveNumbersSum(self, n: int) -> int:
    # Keep original n for clarity
    original_n = n
    doubled_n = 2 * original_n  # More explicit than n << 1
  
    count = 0
    k = 1  # Use k instead of sequence_length for consistency with formula
  
    while k * (k + 1) <= doubled_n:
        # Now it's clear we're working with doubled_n throughout
        if doubled_n % k == 0 and (doubled_n // k - k + 1) % 2 == 0:
            count += 1
        k += 1
  
    return count
```

### 3. **Off-by-One Errors in the Mathematical Formula**

A subtle pitfall is misunderstanding the formula derivation. The sum of k consecutive integers starting from `a` is `a + (a+1) + ... + (a+k-1)`, NOT `a + (a+1) + ... + (a+k)`. This affects the formula and can lead to incorrect implementations.

**Solution:** Always verify with small examples:
```python
# Test with n = 9
# Valid representations: [9], [4,5], [2,3,4]
# k=1: 9 = 9 ✓
# k=2: 9 = 4+5 ✓  
# k=3: 9 = 2+3+4 ✓
# Your solution should return 3 for n=9
```

### 4. **Forgetting Edge Cases**

It is worth checking edge cases such as `n = 1`. The main loop already handles it: for `doubled_n = 2`, only `k = 1` satisfies `k * (k + 1) <= 2`, and it passes both checks, so the result is 1. A special case like the one below is therefore optional:
```python
def consecutiveNumbersSum(self, n: int) -> int:
    if n == 1:
        return 1  # Only one way: [1]
  
    # Continue with main algorithm...
```

### 5. **Incorrect Modulo Check**

A critical error occurs when developers check `(doubled_n // k) % 2` instead of `(doubled_n // k - k + 1) % 2`. The `-k+1` part is essential for ensuring the starting value `a` is a positive integer.

**Solution:** Remember the complete formula and what each part represents:
```python
# The condition (doubled_n // k - k + 1) % 2 == 0 ensures:
# 2a = doubled_n // k - k + 1 is even (which it must be since 2a is even)
# This guarantees 'a' is a positive integer

# Wrong:
if doubled_n % k == 0 and (doubled_n // k) % 2 == 0:  # Missing -k+1

# Correct:
if doubled_n % k == 0 and (doubled_n // k - k + 1) % 2 == 0:
```

