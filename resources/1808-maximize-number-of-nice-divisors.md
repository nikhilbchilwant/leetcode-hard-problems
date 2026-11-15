# 1808. Maximize Number of Nice Divisors

> Source: https://algo.monster/liteproblems/1808

## Problem Statement

You need to construct a positive integer `n` with specific properties to maximize a certain type of divisor count.

Given a positive integer `primeFactors`, construct an integer `n` such that:

1. **Prime Factor Count Constraint**: The total number of prime factors of `n` (counting multiplicities) must not exceed `primeFactors`. For example, if `n = 12 = 2 × 2 × 3`, it has 3 prime factors: `[2, 2, 3]`.

2. **Nice Divisor Definition**: A divisor of `n` is called "nice" if it is divisible by every distinct prime that appears in `n`'s prime factorization. 
   - For example, if `n = 12 = 2² × 3`, the distinct primes are 2 and 3
   - The divisor 6 (= 2 × 3) is nice because it contains both primes
   - The divisor 12 (= 2² × 3) is also nice
   - But 3 and 4 are not nice divisors (3 doesn't contain prime 2, and 4 doesn't contain prime 3)

3. **Objective**: Maximize the number of nice divisors of `n`.

The problem asks you to return the maximum possible count of nice divisors. Since this number can be very large, return the result modulo `10⁹ + 7`.

**Key Insight**: If `n = p₁^k₁ × p₂^k₂ × ... × pₘ^kₘ` where `p₁, p₂, ..., pₘ` are distinct primes, then:
- The sum `k₁ + k₂ + ... + kₘ ≤ primeFactors`
- The number of nice divisors equals `k₁ × k₂ × ... × kₘ` (since a nice divisor must have the form `p₁^a₁ × p₂^a₂ × ... × pₘ^aₘ` where `1 ≤ aᵢ ≤ kᵢ`)

This transforms the problem into: how to split `primeFactors` into a sum of positive integers such that their product is maximized. The solution uses the mathematical principle that splitting into 3's (with special handling for remainders) gives the optimal product.

## Intuition

The key realization is that we need to maximize the product of integers that sum to at most `primeFactors`. This is a classic mathematical optimization problem.

Let's think about how nice divisors are counted. If we have `n = p₁^k₁ × p₂^k₂ × ... × pₘ^kₘ`, then each nice divisor must include all distinct primes `p₁, p₂, ..., pₘ`. For each prime `pᵢ`, we can choose its exponent from 1 to `kᵢ`, giving us `kᵢ` choices. Therefore, the total number of nice divisors is `k₁ × k₂ × ... × kₘ`.

Now we face the question: given that `k₁ + k₂ + ... + kₘ ≤ primeFactors`, how do we maximize `k₁ × k₂ × ... × kₘ`?

This is equivalent to asking: how should we split a number into parts to maximize their product?

Let's explore with small examples:
- Splitting 6: `6 = 3 + 3` gives product `3 × 3 = 9`, while `6 = 2 + 2 + 2` gives `2 × 2 × 2 = 8`, and `6 = 1 + 5` gives only `1 × 5 = 5`
- Splitting 8: `8 = 3 + 3 + 2` gives `3 × 3 × 2 = 18`, while `8 = 2 + 2 + 2 + 2` gives `2 × 2 × 2 × 2 = 16`

From mathematical analysis, we discover that:
1. We should avoid using 1 in our split (since multiplying by 1 doesn't increase the product)
2. We should prefer 3 over 2 when possible (since for the same sum, `3 + 3 = 6` gives product 9, while `2 + 2 + 2 = 6` gives product 8)
3. We shouldn't use numbers larger than 4 (since any number ≥ 5 can be split into 2 and 3 for a larger product)

The optimal strategy emerges:
- Use as many 3's as possible
- When `primeFactors % 3 = 1`, we have one leftover. Instead of using `3 + 1`, we use `2 + 2` (since `2 × 2 = 4 > 3 × 1 = 3`)
- When `primeFactors % 3 = 2`, we add one more 2 to our 3's

This explains why the solution computes `3^(primeFactors // 3)` as the base and adjusts for remainders:
- Remainder 0: use all 3's
- Remainder 1: replace one 3 with two 2's (equivalent to multiplying by 4/3)
- Remainder 2: add one 2 to the product

**Learn more about [Recursion](/problems/recursion_intro) and [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows the mathematical optimization strategy we discovered: splitting `primeFactors` into parts that maximize their product.

**Step 1: Handle Base Cases**
For `primeFactors < 4`, we return `primeFactors` directly. This is because:
- If `primeFactors = 1`, we have `n = p`, giving 1 nice divisor
- If `primeFactors = 2`, we have `n = p²` or `n = p₁ × p₂`, both giving 2 nice divisors
- If `primeFactors = 3`, we have `n = p³` or other combinations, with 3 being optimal

**Step 2: Apply the Splitting Strategy**
For `primeFactors ≥ 4`, we examine `primeFactors % 3`:

**Case 1: `primeFactors % 3 == 0`**
- We can split `primeFactors` perfectly into groups of 3
- The maximum product is `3^(primeFactors // 3)`
- We use `pow(3, primeFactors // 3, mod)` for efficient modular exponentiation

**Case 2: `primeFactors % 3 == 1`**
- We have `primeFactors = 3k + 1` for some integer `k`
- Instead of using `3^k × 1`, we use `3^(k-1) × 4`
- This is because `3 + 1 = 4` can be split as `2 + 2`, giving product `2 × 2 = 4`
- The result is `4 × 3^(primeFactors // 3 - 1)`

**Case 3: `primeFactors % 3 == 2`**
- We have `primeFactors = 3k + 2` for some integer `k`
- We use `3^k × 2`
- The extra 2 is added to the product of 3's
- The result is `2 × 3^(primeFactors // 3)`

**Step 3: Modular Arithmetic**
Since the result can be extremely large, we:
- Use Python's built-in `pow(base, exp, mod)` function for efficient modular exponentiation
- Apply `% mod` to the final result where `mod = 10^9 + 7`

The algorithm runs in `O(log primeFactors)` time due to the fast power operation, making it efficient even for large values of `primeFactors`.

## Example Walkthrough

Let's walk through the solution with `primeFactors = 10`.

**Step 1: Understanding what we're maximizing**
- We need to construct an integer `n` using at most 10 prime factors (counting multiplicities)
- If `n = p₁^k₁ × p₂^k₂ × ... × pₘ^kₘ`, then `k₁ + k₂ + ... + kₘ ≤ 10`
- The number of nice divisors = `k₁ × k₂ × ... × kₘ`
- We need to maximize this product

**Step 2: Finding the optimal split**
We need to split 10 into positive integers that maximize their product.

Let's consider different ways to split 10:
- `10 = 5 + 5`: product = `5 × 5 = 25`
- `10 = 4 + 3 + 3`: product = `4 × 3 × 3 = 36`
- `10 = 3 + 3 + 3 + 1`: product = `3 × 3 × 3 × 1 = 27`
- `10 = 3 + 3 + 2 + 2`: product = `3 × 3 × 2 × 2 = 36`
- `10 = 2 + 2 + 2 + 2 + 2`: product = `2^5 = 32`

**Step 3: Applying the algorithm's strategy**
Since `primeFactors = 10` and `10 % 3 = 1`, we fall into Case 2:
- We would get `10 = 3 + 3 + 3 + 1` if we used all 3's
- But `3 × 1 = 3` is worse than `2 × 2 = 4`
- So we replace one 3 and the 1 with two 2's
- This gives us: `10 = 3 + 3 + 2 + 2`

**Step 4: Calculate the result**
- `primeFactors // 3 = 3`
- Since remainder is 1, we use: `4 × 3^(3-1) = 4 × 3^2 = 4 × 9 = 36`

**Step 5: Construct the actual n**
One possible `n` would be:
- `n = 2³ × 3³ × 5² × 7²` (using exponents 3, 3, 2, 2 which sum to 10)
- Nice divisors must include all four primes: 2, 3, 5, and 7
- For each prime, we can choose exponents: 1-3 for 2, 1-3 for 3, 1-2 for 5, 1-2 for 7
- Total nice divisors = `3 × 3 × 2 × 2 = 36`

This confirms our algorithm correctly identifies that the maximum number of nice divisors is 36 when `primeFactors = 10`.

## Python Solution

```python
class Solution:
    def maxNiceDivisors(self, primeFactors: int) -> int:
        """
        Find the maximum product of integers that sum to primeFactors.
      
        The key insight is to split primeFactors into parts where each part
        is as close to 3 as possible, since 3 gives the optimal product/sum ratio.
      
        Mathematical reasoning:
        - For any integer n > 4, splitting into 3s gives better product than 2s or 4s
        - Special cases: when remainder is 1, use 4 (2*2) instead of 3*1
        - When remainder is 2, use 2 directly
      
        Args:
            primeFactors: The target sum to split into parts
          
        Returns:
            Maximum product of parts modulo 10^9 + 7
        """
        MOD = 10**9 + 7
      
        # Base cases: for small values, return as is
        if primeFactors < 4:
            return primeFactors
      
        # Calculate based on remainder when divided by 3
        remainder = primeFactors % 3
        quotient = primeFactors // 3
      
        if remainder == 0:
            # Perfectly divisible by 3: use all 3s
            return pow(3, quotient, MOD)
        elif remainder == 1:
            # Remainder 1: replace one 3 with two 2s (3 + 1 = 4 = 2 * 2)
            # This gives us (quotient - 1) threes and one 4
            return (4 * pow(3, quotient - 1, MOD)) % MOD
        else:  # remainder == 2
            # Remainder 2: add one 2 to the product of 3s
            return (2 * pow(3, quotient, MOD)) % MOD
```

## Time and Space Complexity

The time complexity of this algorithm is `O(log n)`, where `n` is the value of `primeFactors`. This is because the dominant operation in the code is the `pow(3, primeFactors // 3, mod)` function call, which uses modular exponentiation. The built-in `pow` function with three arguments performs fast exponentiation using the square-and-multiply algorithm, which takes `O(log k)` time where `k` is the exponent. Since the exponent is at most `primeFactors // 3`, and in the worst case could be approximately `primeFactors / 3`, the time complexity is `O(log(primeFactors))` or `O(log n)`.

The space complexity is `O(1)` as the algorithm only uses a constant amount of extra space. The variables used are `mod` and the temporary values during computation. The modular exponentiation operation `pow(3, primeFactors // 3, mod)` is implemented iteratively in Python's built-in function and doesn't require additional space proportional to the input size.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Multiplication**
One of the most common mistakes is forgetting to apply modulo operations during intermediate multiplication steps, especially in the cases where remainder is 1 or 2.

**Incorrect Implementation:**
```python
# This can cause overflow for large quotients
if remainder == 1:
    return 4 * pow(3, quotient - 1, MOD) % MOD  # Overflow risk!
```

**Correct Implementation:**
```python
# Apply modulo to prevent overflow
if remainder == 1:
    return (4 * pow(3, quotient - 1, MOD)) % MOD
```

### 2. **Misunderstanding the Remainder == 1 Case**
Many solutions incorrectly handle when `primeFactors % 3 == 1`. The intuition that "just add 1 to the product" is wrong because `3^k × 1 = 3^k`, which doesn't utilize the extra prime factor.

**Incorrect Approach:**
```python
if remainder == 1:
    return pow(3, quotient, MOD)  # Wrong! This ignores the extra 1
```

**Correct Approach:**
```python
if remainder == 1:
    # Take one 3 away and combine with the 1 to make 4 (as 2×2)
    return (4 * pow(3, quotient - 1, MOD)) % MOD
```

### 3. **Using Regular Power Instead of Modular Exponentiation**
For large values of `primeFactors`, computing `3^quotient` directly will cause overflow or timeout.

**Incorrect:**
```python
result = (3 ** quotient) % MOD  # Too slow and can overflow!
```

**Correct:**
```python
result = pow(3, quotient, MOD)  # Uses fast modular exponentiation
```

### 4. **Off-by-One Error in Base Cases**
Some implementations incorrectly handle small values of `primeFactors`, particularly forgetting that `primeFactors = 4` should be split as `2 + 2` (product = 4), not returned as 4 directly.

**Incorrect Base Case:**
```python
if primeFactors <= 4:
    return primeFactors  # Wrong for primeFactors = 4
```

**Correct Base Case:**
```python
if primeFactors < 4:
    return primeFactors
# primeFactors = 4 should go through the main logic
```

### 5. **Not Considering Edge Case of primeFactors = 0**
While not typically part of the problem constraints, handling `primeFactors = 0` incorrectly can cause issues.

**Better Implementation with Guard:**
```python
def maxNiceDivisors(self, primeFactors: int) -> int:
    if primeFactors == 0:
        return 0  # or 1, depending on problem definition
  
    MOD = 10**9 + 7
    if primeFactors < 4:
        return primeFactors
    # ... rest of the logic
```

