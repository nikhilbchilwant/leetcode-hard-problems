# 479. Largest Palindrome Product

> Source: https://algo.monster/liteproblems/479

## Problem Statement

You need to find the largest palindromic number that can be formed by multiplying two n-digit integers together.

A palindrome is a number that reads the same forwards and backwards (like 9009 or 12321).

Given an integer `n` representing the number of digits, you should:
1. Consider all possible products of two n-digit numbers
2. Find which of these products are palindromes
3. Return the largest such palindrome

Since the result can be extremely large, return the answer modulo 1337.

For example:
- If `n = 2`, the two-digit numbers range from 10 to 99
- The largest palindrome product would be 99 × 91 = 9009
- So you would return 9009 % 1337 = 987

The challenge is to efficiently find this largest palindrome without checking every possible product, as that would be computationally expensive for larger values of `n`.

## Intuition

The key insight is that we can construct palindromes directly rather than checking every possible product. Since we want the largest palindrome, we should start from the largest possible palindrome and work our way down.

For n-digit numbers, the maximum value is `10^n - 1` (like 99 for n=2, 999 for n=3). The largest possible product would be `(10^n - 1) × (10^n - 1)`, which gives us an upper bound for our palindrome.

The clever approach is to build palindromes by taking the first half and mirroring it. For example, if we start with 99, we can create the palindrome 9999. If that doesn't work, try 98 → 9889, then 97 → 9779, and so on.

Here's the process:
1. Start with the largest n-digit number as the first half (`a`)
2. Create a palindrome by appending the reverse of `a` to itself (so 99 becomes 9999)
3. Check if this palindrome can be expressed as a product of two n-digit numbers
4. If yes, we found our answer; if no, decrease `a` and try again

To check if a palindrome `x` is a valid product, we start from the largest n-digit number `t` and check if `x` is divisible by `t`. We only need to check values of `t` where `t × t >= x` (since if `t × t < x`, then `t` multiplied by any n-digit number would be less than `x`).

This approach is efficient because:
- We generate palindromes in descending order, so the first valid one is the largest
- We avoid checking millions of products by directly constructing palindrome candidates
- We stop as soon as we find a valid palindrome product

## Solution Approach

The implementation follows the intuition by systematically generating palindromes and checking if they can be factored into two n-digit numbers.

**Step 1: Calculate the maximum n-digit number**
```python
mx = 10**n - 1
```
This gives us 99 for n=2, 999 for n=3, etc.

**Step 2: Generate palindromes in descending order**
```python
for a in range(mx, mx // 10, -1):
```
We iterate from the largest n-digit number down to the smallest n-digit number (`mx // 10` is the lower bound).

**Step 3: Construct the palindrome**
```python
b = x = a
while b:
    x = x * 10 + b % 10
    b //= 10
```
This creates a palindrome by:
- Starting with `x = a` (the first half)
- Extracting digits from `b` (copy of `a`) using `b % 10`
- Appending each digit to `x` by multiplying `x` by 10 and adding the digit
- For example: if `a = 99`, then `x` becomes 99 → 99 × 10 + 9 = 999 → 999 × 10 + 9 = 9999

**Step 4: Check if the palindrome is a valid product**
```python
t = mx
while t * t >= x:
    if x % t == 0:
        return x % 1337
    t -= 1
```
Starting from the maximum n-digit number `t`:
- Check if `t * t >= x` (ensuring the product can reach our palindrome)
- If `x` is divisible by `t`, then `x = t * (x/t)` where both factors could be n-digit numbers
- If we find such a factor, return the palindrome modulo 1337
- Otherwise, decrease `t` and continue checking

**Edge Case:**
```python
return 9
```
For n=1, the only single-digit palindrome product is 9 (3 × 3), which is handled as a special case.

The algorithm efficiently finds the answer by:
1. Generating palindromes from largest to smallest
2. For each palindrome, checking if it can be factored appropriately
3. Returning immediately upon finding the first (and therefore largest) valid palindrome

## Example Walkthrough

Let's walk through the solution with `n = 2` (two-digit numbers from 10 to 99).

**Step 1: Calculate maximum n-digit number**
- `mx = 10^2 - 1 = 99`

**Step 2: Start generating palindromes from largest to smallest**

First iteration with `a = 99`:
- Create palindrome: Start with `x = 99`, then mirror it
  - `b = 99`
  - Extract last digit: `99 % 10 = 9`, append to x: `x = 99 * 10 + 9 = 999`
  - `b = 99 // 10 = 9`
  - Extract last digit: `9 % 10 = 9`, append to x: `x = 999 * 10 + 9 = 9999`
  - `b = 9 // 10 = 0` (done)
- Palindrome created: `x = 9999`

**Step 3: Check if 9999 can be factored into two 2-digit numbers**
- Start with `t = 99`
- Check: Is `99 * 99 = 9801 >= 9999`? No! So 9999 cannot be a product of two 2-digit numbers
- Move to next palindrome

Second iteration with `a = 98`:
- Create palindrome: `98 → 9889`
- Check factorization starting from `t = 99`
  - Is `99 * 99 = 9801 >= 9889`? No!
- Move to next palindrome

For `a = 97` down to `a = 92`, the palindromes 9779, 9669, 9559, 9449, 9339, and 9229 are all at most 9801, so the divisor loop runs, but none of them has a divisor `t` with `t * t >= x`. Each one is skipped.

At `a = 91`:
- Create palindrome: `91 → 9119`
- Check factorization starting from `t = 99`
  - Is `99 * 99 = 9801 >= 9119`? Yes!
  - Is `9119 % 99 == 0`? No
  - Try `t = 98`: Is `98 * 98 = 9604 >= 9119`? Yes!
  - Is `9119 % 98 == 0`? No
  - Continue checking...
  - The loop stops once `t * t < 9119` (at `t = 95`) without finding a divisor, so 9119 is skipped

When we reach `a = 90`:
- Create palindrome: `90 → 9009`
- Check factorization starting from `t = 99`
  - Is `99 * 99 = 9801 >= 9009`? Yes!
  - Is `9009 % 99 == 0`? Yes! `9009 / 99 = 91`
  - Both 99 and 91 are 2-digit numbers ✓

**Step 4: Return result**
- Found largest palindrome: 9009
- Return `9009 % 1337 = 987`

The algorithm efficiently found that 99 × 91 = 9009 is the largest palindrome product of two 2-digit numbers by:
1. Generating palindromes in descending order (9999, 9889, 9779, ..., 9009)
2. Checking each palindrome for valid factorization
3. Stopping at the first valid palindrome found

## Python Solution

```python
class Solution:
    def largestPalindrome(self, n: int) -> int:
        # Special case: for n=1, the largest palindrome product is 9 (3*3)
        if n == 1:
            return 9
          
        # Find the maximum n-digit number
        max_num = 10**n - 1
      
        # Try to construct palindromes starting from the largest possible
        # We iterate from max_num down to max_num//10 (minimum n-digit number)
        for first_half in range(max_num, max_num // 10, -1):
            # Construct a palindrome by mirroring the first half
            # For example, if first_half = 123, palindrome = 123321
            palindrome = first_half
            temp = first_half
          
            # Mirror the digits to create the second half of the palindrome
            while temp > 0:
                palindrome = palindrome * 10 + temp % 10
                temp //= 10
          
            # Check if this palindrome can be expressed as a product of two n-digit numbers
            # Start from the largest possible divisor
            divisor = max_num
          
            # Only check divisors whose square is >= palindrome
            # (if divisor^2 < palindrome, then palindrome/divisor > divisor,
            # meaning we've already checked this pair)
            while divisor * divisor >= palindrome:
                if palindrome % divisor == 0:
                    # Found a valid factorization, return result modulo 1337
                    return palindrome % 1337
                divisor -= 1
      
        # This should not be reached for valid inputs
        return 9
```

## Time and Space Complexity

**Time Complexity:** `O(10^(2n))` in the worst case

The analysis breaks down as follows:
- The outer loop iterates from `mx` to `mx // 10`, which is approximately `10^n - 10^(n-1) ≈ 9 * 10^(n-1)` iterations, simplified to `O(10^n)` iterations
- For each value of `a`, we construct a palindrome by reversing `a` and appending it, which takes `O(n)` time (since `a` has at most `n` digits)
- The inner while loop checks divisors from `mx` down to `√x`, where `x` is the palindrome formed. Since `x ≈ 10^(2n)`, we check approximately `10^n - 10^(n/2) ≈ O(10^n)` potential divisors in the worst case
- Therefore, the total time complexity is `O(10^n) * O(10^n) = O(10^(2n))` for the nested loops. In practice the search stops after a small number of palindromes, because a valid palindrome is found close to the top of the range

**Space Complexity:** `O(1)`

The space analysis:
- The algorithm uses only a constant number of variables: `mx`, `a`, `b`, `x`, and `t`
- No additional data structures like arrays, lists, or recursion stacks are used
- All operations are performed in-place with primitive integer variables
- Therefore, the space complexity is constant `O(1)`

## Common Pitfalls

### 1. **Integer Overflow for Large Values of n**

When `n` gets large (especially n ≥ 7), the palindrome numbers become massive and can exceed the typical integer limits in some programming languages. For instance, with n=8, we're dealing with 16-digit palindromes that can reach values near 10^16.

**Solution:** Python handles arbitrary precision integers natively, but in languages like Java or C++, you'd need to:
- Use `long long` in C++ or `Long` in Java
- Consider using modular arithmetic throughout if only the final modulo result is needed
- Implement custom big integer handling for very large values

### 2. **Inefficient Palindrome Generation Leading to Timeout**

A naive approach might generate all possible products first and then check for palindromes, which has O(10^(2n)) complexity. Even the optimized approach can be slow if not carefully implemented.

**Solution:** 
- Generate palindromes directly rather than checking all products
- Start from the largest possible palindrome and work downward
- Use early termination: once a valid palindrome is found, immediately return it
- Optimize the divisor checking loop by only checking divisors where `divisor * divisor >= palindrome`

### 3. **Incorrect Palindrome Construction**

A common mistake is incorrectly mirroring the digits. For example, attempting to use string concatenation or getting the digit extraction wrong:

```python
# WRONG: This doesn't properly reverse the digits
palindrome = int(str(first_half) + str(first_half))  # Creates 123123, not 123321

# CORRECT: Properly extract and append digits in reverse
while temp > 0:
    palindrome = palindrome * 10 + temp % 10
    temp //= 10
```

**Solution:** Use the mathematical approach shown in the code that extracts digits using modulo 10 and integer division, building the palindrome digit by digit.

### 4. **Missing the n=1 Edge Case**

The algorithm assumes we're creating even-length palindromes by mirroring n-digit numbers. For n=1, this would create 2-digit palindromes (like 11, 22, etc.), but these aren't products of two 1-digit numbers.

**Solution:** Handle n=1 as a special case that returns 9 (which is 3 × 3).

### 5. **Incorrect Bounds for Factor Checking**

When checking if a palindrome can be factored into two n-digit numbers, both factors must be n-digit numbers. A common mistake is not ensuring both factors fall within the valid range.

```python
# WRONG: Doesn't ensure the quotient is also an n-digit number
if palindrome % divisor == 0:
    return palindrome % 1337

# BETTER: Could add verification that palindrome/divisor is also n-digits
if palindrome % divisor == 0:
    quotient = palindrome // divisor
    if quotient <= max_num and quotient > max_num // 10:
        return palindrome % 1337
```

**Solution:** The current implementation does not need the extra check. The loop condition `divisor * divisor >= palindrome` guarantees `quotient = palindrome // divisor <= divisor <= max_num`, and since `palindrome >= 10^(2n-1)` and `divisor <= max_num < 10^n`, the quotient is also greater than `10^(n-1)`. So any divisor found inside the loop already gives two n-digit factors.

