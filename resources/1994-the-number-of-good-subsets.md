# 1994. The Number of Good Subsets

> Source: https://algo.monster/liteproblems/1994

## Problem Statement

You are given an integer array `nums`. A subset of `nums` is called **good** if the product of all its elements can be expressed as a product of **distinct prime numbers** (each prime appears at most once in the factorization).

In other words, a subset is good if its product has no repeated prime factors - each prime in the prime factorization appears with a power of exactly 1.

For example, with `nums = [1, 2, 3, 4]`:
- `[2, 3]` is good because its product is `6 = 2 × 3` (two distinct primes)
- `[1, 2, 3]` is good because its product is `6 = 2 × 3` (the 1 doesn't affect the product)
- `[1, 3]` is good because its product is `3` (one prime)
- `[1, 4]` is NOT good because its product is `4 = 2²` (prime 2 appears twice)
- `[4]` is NOT good because `4 = 2²` (prime 2 appears twice)

A number whose prime factorization contains only distinct primes (each with power 1) is called a **square-free** number. So essentially, we need to count subsets whose product is square-free.

The task is to count the total number of different good subsets in `nums`. Return the answer modulo `10⁹ + 7`.

Note that:
- A subset can be obtained by selecting any combination of elements from `nums` (including the empty subset, though it won't be good)
- Two subsets are considered different if they are formed by selecting different indices from the array, even if they contain the same values
- The number 1 is special - it doesn't affect the product, so it can be freely included or excluded from any good subset

## Intuition

The key insight is that we need subsets whose product is square-free (no repeated prime factors). Since `nums` contains integers from 1 to 30, we first need to understand which numbers can potentially be part of good subsets.

Numbers that can NEVER be in a good subset are those that already contain repeated prime factors:
- Numbers divisible by `4 = 2²`, `9 = 3²`, `25 = 5²` cannot be used
- These numbers would immediately make any subset "bad"

Numbers that CAN be in good subsets:
- Square-free numbers (numbers with no repeated prime factors)
- The number 1 is special - it doesn't contribute any prime factors, so it can be freely added to any good subset

Since we're dealing with numbers up to 30, there are only 10 possible prime factors: `[2, 3, 5, 7, 11, 13, 17, 19, 23, 29]`. This small number of primes suggests we can use **[bitmask](/problems/bitmask_intro) [dynamic programming](/problems/dynamic_programming_intro)**.

The core idea is to use a [bitmask](/problems/bitmask_intro) to represent which primes are "used" in our current subset's product:
- Each bit position represents one of the 10 primes
- If bit `i` is set to 1, it means prime `primes[i]` is used in our product
- A valid good subset must never use the same prime twice

We can build up our answer using [dynamic programming](/problems/dynamic_programming_intro) where `f[mask]` represents the number of ways to form subsets that use exactly the primes indicated by the [bitmask](/problems/bitmask_intro).

For each valid number `x` (2 to 30, excluding those with repeated prime factors):
1. Calculate its prime mask (which primes divide `x`)
2. For each existing state, if we can add `x` without repeating primes (no overlap in masks), we update the count

The special handling of 1s: Since any number of 1s can be added to any good subset without affecting its "goodness", if there are `cnt[1]` ones in the array, each good subset can be combined with any subset of these 1s, giving us `2^cnt[1]` variations for each good subset.

**Learn more about [Math](/problems/math-basics), [Dynamic Programming](/problems/dynamic_programming_intro) and [Bitmask](/problems/bitmask_intro) patterns.**

## Solution Approach

The implementation uses [bitmask](/problems/bitmask_intro) [dynamic programming](/problems/dynamic_programming_intro) to count all good subsets:

**1. Initialize Constants and Data Structures:**
- Define `primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]` - all primes up to 30
- Use `Counter` to count frequency of each number in `nums`
- Create DP array `f` of size `2^10` (1024) where `f[mask]` represents the number of ways to form subsets using exactly the primes indicated by `mask`

**2. Base Case:**
- `f[0] = pow(2, cnt[1])` - The empty prime set (no primes used) can be formed by taking any subset of 1s from the array. Since 1 doesn't contribute any prime factors, we have `2^cnt[1]` ways.

**3. Process Each Valid Number (2 to 30):**
```python
for x in range(2, 31):
    if cnt[x] == 0 or x % 4 == 0 or x % 9 == 0 or x % 25 == 0:
        continue
```
- Skip if the number doesn't appear in `nums` or contains repeated prime factors

**4. Calculate Prime Mask for Current Number:**
```python
mask = 0
for i, p in enumerate(primes):
    if x % p == 0:
        mask |= 1 << i
```
- Create a [bitmask](/problems/bitmask_intro) representing which primes divide `x`
- If prime `p` at index `i` divides `x`, set bit `i` in the mask

**5. Update DP States:**
```python
for state in range((1 << n) - 1, 0, -1):
    if state & mask == mask:
        f[state] = (f[state] + cnt[x] * f[state ^ mask]) % mod
```
- Iterate through states in reverse order (to avoid using updated values in the same iteration)
- `state & mask == mask` checks if `state` contains all primes in `mask` (i.e., we can remove `mask` from `state`)
- `state ^ mask` removes the primes of `x` from the current state, giving us the previous state
- We add `cnt[x] * f[state ^ mask]` because we can choose any of the `cnt[x]` occurrences of `x` to add to subsets represented by `f[state ^ mask]`

**6. Calculate Final Answer:**
```python
return sum(f[i] for i in range(1, 1 << n)) % mod
```
- Sum all non-empty prime combinations (`f[1]` through `f[1023]`)
- Exclude `f[0]` as it represents the empty subset (or subsets containing only 1s, which have no prime factors)

The algorithm efficiently counts all possible good subsets by tracking which primes have been used, ensuring no prime appears more than once in any subset's product.

## Example Walkthrough

Let's walk through a small example with `nums = [1, 2, 3, 4]`.

**Step 1: Count frequencies**
- `cnt[1] = 1, cnt[2] = 1, cnt[3] = 1, cnt[4] = 1`

**Step 2: Initialize DP array**
- `f[0] = 2^1 = 2` (we have one `1` in the array, so 2 ways: include it or not)
- All other `f[mask] = 0` initially

**Step 3: Process valid numbers (2 to 30)**

**Processing x = 2:**
- `2` is valid (not divisible by 4, 9, or 25)
- Prime mask: `2 = 2^1`, so `mask = 0b0000000001` (bit 0 set, representing prime 2)
- Update states:
  - For `state = 0b0000000001` (has prime 2):
    - Check: `state & mask == mask`? Yes (0b01 & 0b01 = 0b01)
    - Previous state: `state ^ mask = 0b01 ^ 0b01 = 0b00`
    - Update: `f[0b01] = (0 + 1 * f[0b00]) % mod = 1 * 2 = 2`

**Processing x = 3:**
- `3` is valid
- Prime mask: `3 = 3^1`, so `mask = 0b0000000010` (bit 1 set, representing prime 3)
- Update states:
  - For `state = 0b0000000011` (has primes 2 and 3):
    - Check: `state & mask == mask`? Yes (0b11 & 0b10 = 0b10)
    - Previous state: `state ^ mask = 0b11 ^ 0b10 = 0b01`
    - Update: `f[0b11] = (0 + 1 * f[0b01]) % mod = 1 * 2 = 2`
  - For `state = 0b0000000010` (has prime 3):
    - Check: `state & mask == mask`? Yes (0b10 & 0b10 = 0b10)
    - Previous state: `state ^ mask = 0b10 ^ 0b10 = 0b00`
    - Update: `f[0b10] = (0 + 1 * f[0b00]) % mod = 1 * 2 = 2`

**Processing x = 4:**
- `4 = 2²` is NOT valid (divisible by 4)
- Skip this number

**Step 4: Calculate final answer**
- Sum all non-empty states: `f[0b01] + f[0b10] + f[0b11] = 2 + 2 + 2 = 6`

**Verification of the 6 good subsets:**
1. `[2]` → product = 2 (prime factorization: 2¹) ✓
2. `[1, 2]` → product = 2 (prime factorization: 2¹) ✓
3. `[3]` → product = 3 (prime factorization: 3¹) ✓
4. `[1, 3]` → product = 3 (prime factorization: 3¹) ✓
5. `[2, 3]` → product = 6 (prime factorization: 2¹ × 3¹) ✓
6. `[1, 2, 3]` → product = 6 (prime factorization: 2¹ × 3¹) ✓

Note that subsets containing `4` like `[4]`, `[1, 4]`, `[2, 4]`, etc., are NOT good because `4 = 2²` has a repeated prime factor.

The DP approach efficiently counts these by tracking which primes are used (via bitmask) and ensuring no prime appears twice.

## Python Solution

```python
class Solution:
    def numberOfGoodSubsets(self, nums: List[int]) -> int:
        # Define all prime numbers up to 30
        primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
      
        # Count frequency of each number in the input array
        count = Counter(nums)
      
        # Define modulo for the result
        MOD = 10**9 + 7
      
        # Number of prime factors we're tracking
        num_primes = len(primes)
      
        # dp[mask] represents the number of good subsets where mask indicates which primes are used
        # If bit i is set in mask, it means primes[i] is a factor of some number in the subset
        dp = [0] * (1 << num_primes)
      
        # Base case: empty subset can be combined with any number of 1s
        # Since 1 doesn't affect the product being square-free, we can choose any subset of 1s
        dp[0] = pow(2, count[1], MOD)
      
        # Iterate through all possible numbers from 2 to 30
        for num in range(2, 31):
            # Skip if this number doesn't appear in the input
            if count[num] == 0:
                continue
          
            # Skip numbers that have a prime factor appearing more than once
            # (i.e., numbers divisible by 4=2², 9=3², or 25=5²)
            if num % 4 == 0 or num % 9 == 0 or num % 25 == 0:
                continue
          
            # Calculate the prime factorization mask for current number
            prime_mask = 0
            for i, prime in enumerate(primes):
                if num % prime == 0:
                    prime_mask |= 1 << i
          
            # Update dp states in reverse order to avoid using updated values
            for state in range((1 << num_primes) - 1, 0, -1):
                # Check if current state contains all prime factors of num
                # This ensures no prime appears more than once in the product
                if state & prime_mask == prime_mask:
                    # Add subsets that include current number to existing subsets
                    # state ^ prime_mask gives the state without primes from current number
                    dp[state] = (dp[state] + count[num] * dp[state ^ prime_mask]) % MOD
      
        # Sum all non-empty subsets (exclude dp[0] which represents empty subset)
        result = sum(dp[i] for i in range(1, 1 << num_primes)) % MOD
      
        return result
```

## Time and Space Complexity

**Time Complexity:** `O(2^n * (n + 30))` where `n = 10` (number of primes)

- Counting frequency of numbers in `nums` takes `O(len(nums))` time
- Computing `pow(2, cnt[1])` takes `O(log(cnt[1]))` time
- The outer loop iterates through numbers 2 to 30, which is `O(30)` iterations
- For each valid number `x`, we compute its prime mask in `O(n)` time by checking divisibility with each prime
- The inner loop iterates through all states from `(1 << n) - 1` to 1, which is `O(2^n)` iterations
- Each state update operation takes `O(1)` time
- The final summation iterates through `2^n - 1` states
- Since `n = 10` is a constant, the overall time complexity is `O(2^10 * 40) = O(1024 * 40)` which simplifies to `O(1)` for constant bounds, but more precisely `O(2^n * n)` for the algorithmic analysis

**Space Complexity:** `O(2^n)` where `n = 10`

- The `primes` array uses `O(n)` space
- The `cnt` Counter uses `O(30)` space at most (for numbers 1-30)
- The `f` array uses `O(2^n)` space to store DP states for all possible subsets of primes
- Since `n = 10`, the space complexity is `O(2^10) = O(1024)`, which is effectively `O(1)` for constant bounds, but more precisely `O(2^n)` for the algorithmic analysis

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Incorrect Handling of the Number 1**

**Pitfall:** A common mistake is treating 1 like any other number when updating the DP states. Some might try to process 1 in the main loop along with numbers 2-30, which would incorrectly update the DP states since 1 has no prime factors.

**Why it's wrong:** The number 1 is special because:
- It has no prime factors (empty prime factorization)
- It can be freely added to any good subset without affecting whether the subset remains good
- Multiple 1s can be included without violating the square-free property

**Correct approach:**
```python
# Handle 1s separately in the base case
dp[0] = pow(2, count[1], MOD)  # 2^count[1] ways to choose 1s

# Then process numbers 2-30 in the main loop
for num in range(2, 31):
    # ... process numbers with actual prime factors
```

### 2. **Not Checking for Numbers with Repeated Prime Factors**

**Pitfall:** Forgetting to skip numbers that inherently have repeated prime factors (like 4, 8, 9, 12, 16, 18, 20, 24, 25, 27, 28).

**Why it's wrong:** These numbers can never be part of a good subset because their prime factorization already contains repeated primes:
- 4 = 2² (prime 2 appears twice)
- 9 = 3² (prime 3 appears twice)
- 12 = 2² × 3 (prime 2 appears twice)

**Correct approach:**
```python
# Must check for all perfect square factors up to 30
if num % 4 == 0 or num % 9 == 0 or num % 25 == 0:
    continue  # Skip numbers with repeated prime factors
```

### 3. **Updating DP States in Wrong Order**

**Pitfall:** Updating DP states in forward order (from 0 to 2^n - 1) instead of reverse order.

**Why it's wrong:** If we update in forward order, we might use already-updated values from the current iteration when calculating later states, leading to incorrect counting (elements being counted multiple times).

**Correct approach:**
```python
# Update in reverse order to ensure we use values from previous iteration
for state in range((1 << num_primes) - 1, 0, -1):
    if state & prime_mask == prime_mask:
        dp[state] = (dp[state] + count[num] * dp[state ^ prime_mask]) % MOD
```

### 4. **Incorrectly Checking Prime Mask Compatibility**

**Pitfall:** Using `state & prime_mask == 0` to check if a number can be added to a state, or using `state | prime_mask` incorrectly.

**Why it's wrong:** 
- `state & prime_mask == 0` would check if there's NO overlap, but we're updating states that already contain the primes
- We need to check if the current state can have the prime_mask "removed" from it

**Correct approach:**
```python
# Check if state contains all primes in prime_mask
if state & prime_mask == prime_mask:
    # state ^ prime_mask removes those primes from state
    dp[state] = (dp[state] + count[num] * dp[state ^ prime_mask]) % MOD
```

### 5. **Forgetting to Apply Modulo Operations**

**Pitfall:** Not applying modulo at every arithmetic operation, especially when calculating powers or sums.

**Why it's wrong:** The numbers can grow very large, causing integer overflow. Even in Python which handles big integers, not using modulo can slow down calculations significantly.

**Correct approach:**
```python
# Use modulo in pow function
dp[0] = pow(2, count[1], MOD)  # Third parameter applies modulo efficiently

# Apply modulo after each addition
dp[state] = (dp[state] + count[num] * dp[state ^ prime_mask]) % MOD

# Apply modulo to final sum
result = sum(dp[i] for i in range(1, 1 << num_primes)) % MOD
```

