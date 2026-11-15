# 1735. Count Ways to Make Array With Product

> Source: https://algo.monster/liteproblems/1735

## Problem Statement

You need to find the number of ways to fill an array with positive integers such that their product equals a given value.

Specifically, you're given multiple queries where each query consists of two values:
- `n`: the size of the array to fill
- `k`: the target product

For each query `[n, k]`, you need to count how many different ways you can place positive integers into an array of size `n` such that when you multiply all the integers together, you get exactly `k`.

For example, if `n = 2` and `k = 6`, you could have arrays like:
- `[1, 6]` (product = 1 × 6 = 6)
- `[2, 3]` (product = 2 × 3 = 6)  
- `[3, 2]` (product = 3 × 2 = 6)
- `[6, 1]` (product = 6 × 1 = 6)

So there would be 4 different ways.

Since the number of ways can be very large, each answer should be returned modulo `10^9 + 7`.

The problem requires you to process multiple queries and return an array where each element is the answer to the corresponding query.

## Intuition

To understand how to solve this problem, let's think about what it means for an array of positive integers to have a product equal to `k`.

First, consider the fundamental theorem of arithmetic: every positive integer can be uniquely expressed as a product of prime powers. So if `k = 12 = 2² × 3¹`, then any array whose product is 12 must collectively contain exactly two 2's and one 3 among all its elements when we factor them.

This transforms our problem: instead of thinking about distributing arbitrary positive integers into an array, we can think about distributing prime factors. If `k = 2² × 3¹`, we need to distribute two 2's and one 3 across `n` positions.

For example, with `n = 3` and `k = 12`:
- We could put both 2's and the 3 in one position: `[12, 1, 1]`
- We could separate them: `[4, 3, 1]` (where 4 = 2²)
- Or many other combinations: `[2, 2, 3]`, `[2, 6, 1]`, etc.

The key insight is that distributing prime factors is independent for each prime. The ways to distribute the 2's doesn't affect how we distribute the 3's. This means we can solve for each prime separately and multiply the results.

Now, how many ways can we distribute `x` identical items (copies of a prime) into `n` positions? This is a classic "stars and bars" problem in [combinatorics](/problems/backtracking). We're essentially asking: how many ways can we write `x` as a sum of `n` non-negative integers (since some positions can have 0 copies of the prime, meaning that position's number won't contain this prime factor)?

The formula for this is `C(x + n - 1, n - 1)`. We can visualize this as arranging `x` stars and `n - 1` bars, where the bars separate the stars into `n` groups. The total number of positions is `x + n - 1`, and we choose `n - 1` of them for the bars.

Therefore, our solution approach is:
1. Factorize `k` into prime powers
2. For each prime `p` with exponent `x`, calculate `C(x + n - 1, n - 1)`
3. Multiply all these combinations together to get the final answer

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Combinatorics](/problems/backtracking) patterns.**

## Solution Approach

The implementation consists of two main parts: preprocessing and query processing.

**Preprocessing Phase:**

1. **Factorial and Inverse Factorial Arrays**: We precompute factorials `f[i] = i!` and their modular inverses `g[i] = (i!)^(-1) mod (10^9 + 7)` for all values up to `N = 10020`. The inverse is calculated using Fermat's Little Theorem: `a^(-1) ≡ a^(MOD-2) (mod MOD)` when MOD is prime.

2. **Prime Factorization**: For each number from 1 to N, we compute its prime factorization and store only the exponents in a dictionary `p`. For example, if `12 = 2² × 3¹`, then `p[12] = [2, 1]`. The algorithm uses trial division:
   - For each number `i`, try dividing by all integers `j` from 2 to `√i`
   - Count how many times `j` divides `i` (this gives the exponent)
   - Store only the exponents, not the primes themselves

**Combination Calculation:**

The function `comb(n, k)` calculates `C(n, k) = n! / (k! × (n-k)!)` using the precomputed factorials and their inverses:
```python
comb(n, k) = f[n] * g[k] * g[n - k] % MOD
```

**Query Processing:**

For each query `[n, k]`:
1. Retrieve the list of exponents from `p[k]` (the prime factorization of k)
2. For each exponent `x`, calculate the number of ways to distribute `x` identical items into `n` positions using the formula `C(x + n - 1, n - 1)`
3. Multiply all these combinations together (taking modulo at each step)

The key insight in the implementation is that we don't need to store which primes correspond to which exponents. Since we're multiplying the combinations for all prime factors, we only need the exponents themselves.

For example, if `k = 12 = 2² × 3¹` and `n = 3`:
- For the prime 2 with exponent 2: calculate `C(2 + 3 - 1, 3 - 1) = C(4, 2) = 6`
- For the prime 3 with exponent 1: calculate `C(1 + 3 - 1, 3 - 1) = C(3, 2) = 3`
- Final answer: `6 × 3 = 18` ways

This approach efficiently handles multiple queries by preprocessing all necessary factorizations and using fast combination calculations with precomputed factorials.

## Example Walkthrough

Let's walk through a concrete example with `n = 3` (array size) and `k = 12` (target product).

**Step 1: Prime Factorization**
First, we factorize `k = 12`:
- `12 = 2² × 3¹`
- So we have two 2's and one 3 to distribute across 3 positions

**Step 2: Distribute Prime Factor 2 (appears 2 times)**
We need to distribute 2 copies of prime 2 across 3 positions. Using the stars and bars formula `C(x + n - 1, n - 1)`:
- `C(2 + 3 - 1, 3 - 1) = C(4, 2) = 6`

The 6 ways to distribute two 2's are:
- Position distribution: `[2, 0, 0]` → Array values: `[4, *, *]`
- Position distribution: `[1, 1, 0]` → Array values: `[2, 2, *]`
- Position distribution: `[1, 0, 1]` → Array values: `[2, *, 2]`
- Position distribution: `[0, 2, 0]` → Array values: `[*, 4, *]`
- Position distribution: `[0, 1, 1]` → Array values: `[*, 2, 2]`
- Position distribution: `[0, 0, 2]` → Array values: `[*, *, 4]`

**Step 3: Distribute Prime Factor 3 (appears 1 time)**
We need to distribute 1 copy of prime 3 across 3 positions:
- `C(1 + 3 - 1, 3 - 1) = C(3, 2) = 3`

The 3 ways to distribute one 3 are:
- Position distribution: `[1, 0, 0]` → Array values: `[3, 1, 1]`
- Position distribution: `[0, 1, 0]` → Array values: `[1, 3, 1]`
- Position distribution: `[0, 0, 1]` → Array values: `[1, 1, 3]`

**Step 4: Combine Both Distributions**
Total ways = `6 × 3 = 18`

Some example arrays (combining both prime distributions):
- Two 2's in position 1, one 3 in position 1: `[12, 1, 1]` (12 = 4 × 3)
- Two 2's in position 1, one 3 in position 2: `[4, 3, 1]`
- One 2 in positions 1 and 2, one 3 in position 3: `[2, 2, 3]`
- One 2 in position 1, one 2 in position 3, one 3 in position 2: `[2, 3, 2]`

Each valid array will have exactly two factors of 2 and one factor of 3 distributed among its elements, ensuring the product equals 12.

**Verification**: We can verify a few arrays:
- `[12, 1, 1]`: product = 12 × 1 × 1 = 12 ✓
- `[2, 2, 3]`: product = 2 × 2 × 3 = 12 ✓
- `[4, 3, 1]`: product = 4 × 3 × 1 = 12 ✓

The final answer is **18 ways** to create an array of size 3 with product 12.

## Python Solution

```python
from collections import defaultdict
from typing import List

# Constants
MAX_VALUE = 10020
MOD = 10**9 + 7

# Precompute factorials and their modular inverses
factorials = [1] * MAX_VALUE
factorial_inverses = [1] * MAX_VALUE

# Precompute prime factorizations
prime_factorizations = defaultdict(list)

# Initialize factorials, their inverses, and prime factorizations
for i in range(1, MAX_VALUE):
    # Compute factorial: i! = (i-1)! * i
    factorials[i] = factorials[i - 1] * i % MOD
  
    # Compute modular inverse of factorial using Fermat's Little Theorem
    # Since MOD is prime, inverse of a is a^(MOD-2) mod MOD
    factorial_inverses[i] = pow(factorials[i], MOD - 2, MOD)
  
    # Find prime factorization of i
    current_number = i
    divisor = 2
  
    # Check divisors up to sqrt(current_number)
    while divisor <= current_number // divisor:
        if current_number % divisor == 0:
            # Count the power of this prime factor
            exponent = 0
            while current_number % divisor == 0:
                exponent += 1
                current_number //= divisor
            # Store the exponent of this prime factor
            prime_factorizations[i].append(exponent)
        divisor += 1
  
    # If remaining number > 1, it's a prime factor with exponent 1
    if current_number > 1:
        prime_factorizations[i].append(1)


def combination(n: int, k: int) -> int:
    """
    Calculate nCk (n choose k) modulo MOD using precomputed factorials.
    Formula: nCk = n! / (k! * (n-k)!)
    """
    return factorials[n] * factorial_inverses[k] * factorial_inverses[n - k] % MOD


class Solution:
    def waysToFillArray(self, queries: List[List[int]]) -> List[int]:
        """
        For each query [n, k], find the number of ways to fill an array of size n
        with positive integers such that their product equals k.
      
        Uses the stars and bars method: for each prime factor p^e in k's factorization,
        we need to distribute e occurrences among n positions, which is C(e+n-1, n-1).
        """
        result = []
      
        for array_size, target_product in queries:
            ways = 1
          
            # For each prime factor's exponent in the factorization of target_product
            for exponent in prime_factorizations[target_product]:
                # Apply stars and bars: distribute 'exponent' items into 'array_size' bins
                # This is equivalent to C(exponent + array_size - 1, array_size - 1)
                ways = ways * combination(exponent + array_size - 1, array_size - 1) % MOD
          
            result.append(ways)
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(K × √K + N + m × log K)`

The time complexity consists of three main parts:

1. **Preprocessing factorials and modular inverses:** `O(N)` where `N = 10020`
   - Computing factorials `f[i]` takes `O(N)` time
   - Computing modular inverses `g[i]` using Fermat's Little Theorem with `pow(f[i], MOD - 2, MOD)` takes `O(N × log MOD)`, but since MOD is constant, this is `O(N)`

2. **Prime factorization for all numbers up to N:** `O(N × √N)`
   - For each number `i` from 1 to `N`, we find prime factors
   - The inner while loop runs up to `√i` times (checking `j <= x // j` is equivalent to `j² <= x`)
   - For each `i`, factorization takes `O(√i)` time
   - Total: `O(N × √N)` which simplifies to `O(K × √K)` where `K` represents the maximum value in preprocessing

3. **Processing queries:** `O(m × d)` where `m` is the number of queries and `d` is the average number of distinct prime factors
   - For each query, we iterate through the prime factors of `k`
   - The number of distinct prime factors of any number ≤ N is `O(log N)` in the worst case
   - Therefore, this part is `O(m × log K)`

**Space Complexity:** `O(N)`

The space complexity includes:
- Arrays `f` and `g`: `O(N)` space each
- Dictionary `p` storing prime factorizations: In worst case, stores `O(N × log N)` total values across all keys, but since we only store exponents (not the primes themselves), and the number of distinct prime factors is `O(log N)`, this is `O(N × log N)`. However, the dominant term is still `O(N)` from the arrays
- The answer list: `O(m)` where `m` is the number of queries

The overall space complexity is `O(N)` as the arrays dominate the space usage.

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Integer Overflow in Factorial Computation**
When computing factorials, the intermediate values grow extremely fast. Even with modular arithmetic, if you forget to apply the modulo operation at each multiplication step, you'll encounter integer overflow.

**Pitfall Example:**
```python
# Wrong - can cause overflow
factorials[i] = factorials[i - 1] * i
factorials[i] %= MOD
```

**Solution:**
```python
# Correct - apply modulo at each step
factorials[i] = factorials[i - 1] * i % MOD
```

### 2. **Incorrect Modular Inverse Calculation**
A critical mistake is using regular division instead of modular multiplicative inverse when calculating combinations. This will produce incorrect results.

**Pitfall Example:**
```python
# Wrong - regular division doesn't work with modular arithmetic
def combination(n, k):
    return factorials[n] // (factorials[k] * factorials[n - k]) % MOD
```

**Solution:**
```python
# Correct - use modular multiplicative inverse
def combination(n, k):
    return factorials[n] * factorial_inverses[k] * factorial_inverses[n - k] % MOD
```

### 3. **Missing Edge Cases in Prime Factorization**
The most subtle pitfall is forgetting to handle the case where a prime factor larger than √n remains after trial division. This happens when the number itself is prime or has a large prime factor.

**Pitfall Example:**
```python
# Wrong - misses large prime factors
while divisor * divisor <= current_number:
    if current_number % divisor == 0:
        exponent = 0
        while current_number % divisor == 0:
            exponent += 1
            current_number //= divisor
        prime_factorizations[i].append(exponent)
    divisor += 1
# Forgot to check if current_number > 1!
```

**Solution:**
```python
# Correct - handle remaining prime factor
while divisor * divisor <= current_number:
    # ... factorization logic ...
    divisor += 1

# Critical: Check for remaining prime factor
if current_number > 1:
    prime_factorizations[i].append(1)
```

### 4. **Forgetting Modulo in Product Calculation**
When multiplying the combinations for each prime factor, forgetting to apply modulo after each multiplication can lead to overflow.

**Pitfall Example:**
```python
# Wrong - can overflow
ways = 1
for exponent in prime_factorizations[target_product]:
    ways *= combination(exponent + array_size - 1, array_size - 1)
ways %= MOD  # Too late!
```

**Solution:**
```python
# Correct - apply modulo after each multiplication
ways = 1
for exponent in prime_factorizations[target_product]:
    ways = ways * combination(exponent + array_size - 1, array_size - 1) % MOD
```

### 5. **Off-by-One Errors in Stars and Bars Formula**
The stars and bars formula for distributing n identical items into k distinct boxes is C(n+k-1, k-1). It's easy to mix up the parameters or use the wrong formula variant.

**Pitfall Example:**
```python
# Wrong formulas that are commonly confused
ways *= combination(exponent + array_size, array_size)  # Wrong
ways *= combination(exponent + array_size - 1, exponent)  # Wrong
```

**Solution:**
```python
# Correct - C(items + boxes - 1, boxes - 1)
ways *= combination(exponent + array_size - 1, array_size - 1)
```

