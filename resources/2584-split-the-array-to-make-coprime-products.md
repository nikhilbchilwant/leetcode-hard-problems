# 2584. Split the Array to Make Coprime Products

> Source: https://algo.monster/liteproblems/2584

## Problem Statement

You are given a 0-indexed integer array `nums` of length `n`. Your task is to find the smallest valid split index in the array.

A **split** at index `i` (where `0 <= i <= n - 2`) divides the array into two parts:
- The first part contains elements from index `0` to `i` (inclusive)
- The second part contains elements from index `i + 1` to `n - 1` (inclusive)

A split is considered **valid** if the product of all elements in the first part and the product of all elements in the second part are **coprime** (their greatest common divisor equals 1).

For example, with `nums = [2, 3, 3]`:
- Split at index `i = 0`: First part = `[2]` with product 2, second part = `[3, 3]` with product 9. Since `gcd(2, 9) = 1`, this split is valid.
- Split at index `i = 1`: First part = `[2, 3]` with product 6, second part = `[3]` with product 3. Since `gcd(6, 3) = 3 ≠ 1`, this split is not valid.
- Split at index `i = 2` is not allowed since `i` must be less than `n - 1`.

The goal is to return the **smallest index** `i` where a valid split can be made. If no valid split exists, return `-1`.

**Key Points:**
- The split must divide the array into two non-empty parts
- Two numbers are coprime if their greatest common divisor (GCD) is 1
- You need to find the leftmost (smallest index) valid split position
- The products can be very large, but you don't need to compute them directly - you can work with prime factors instead

## Intuition

The key insight is that two products are coprime if and only if they share no common prime factors. Instead of computing potentially huge products, we can track which prime factors appear on each side of the split.

Think about it this way: if a prime factor `p` appears in any number on the left side and also appears in any number on the right side, then both products will be divisible by `p`, making their GCD at least `p` (not coprime).

This leads to an important observation: for any prime factor `p`, all numbers containing `p` must be on the same side of the split. If we have numbers at indices `i₁, i₂, ..., iₖ` that all contain prime factor `p`, then any valid split must either place all of them on the left side or all on the right side.

We can reformulate this as an interval problem: for each prime factor `p`, find the first index where it appears (`first[p]`) and the last index where it appears (`last[p]`). All occurrences of `p` form an interval `[first[p], last[p]]` that cannot be split.

The problem now becomes: find the smallest index `i` such that no prime factor interval crosses the boundary between index `i` and `i+1`. In other words, for every prime factor, either all its occurrences are at indices `<= i` or all are at indices `> i`.

We can solve this greedily: starting from index 0, we keep track of the maximum right boundary of all prime factor intervals that start at or before our current position. If we ever reach a position `i` where this maximum boundary equals `i`, we've found a valid split point - all prime factors that appeared so far have their last occurrence at or before `i`.

The algorithm essentially merges overlapping intervals of prime factors and finds the first gap between merged intervals, which corresponds to our valid split point.

**Learn more about [Math](/problems/math-basics) patterns.**

## Solution Approach

The implementation follows our intuition of tracking prime factor intervals:

**Step 1: Find Prime Factors and Track Their Intervals**

We initialize two data structures:
- `first`: A dictionary to store the first occurrence index of each prime factor
- `last`: A list where `last[i]` will store the rightmost index that must be grouped with index `i`

For each number in the array, we factorize it to find all its prime factors:
```python
for i, x in enumerate(nums):
    j = 2
    while j <= x // j:  # Check divisors up to sqrt(x)
        if x % j == 0:
            if j in first:
                last[first[j]] = i  # Update the last occurrence for this prime
            else:
                first[j] = i  # Record first occurrence of this prime
            while x % j == 0:
                x //= j  # Remove all occurrences of this prime factor
        j += 1
    if x > 1:  # x is now a prime factor itself
        if x in first:
            last[first[x]] = i
        else:
            first[x] = i
```

This process ensures that for each prime factor `p`, we know its first occurrence at `first[p]` and update `last[first[p]]` to be the rightmost index containing `p`.

**Step 2: Find the Earliest Valid Split**

We traverse the array while maintaining `mx`, the maximum value of `last[j]` for all indices `j` we've seen so far:

```python
mx = last[0]
for i, x in enumerate(last):
    if mx < i:
        return mx  # Found a valid split at index mx
    mx = max(mx, x)
return -1  # No valid split found
```

The key insight: if `mx < i`, it means all prime factors that appeared at or before index `mx` have their last occurrence at or before `mx`. This makes `mx` a valid split point because no prime factor crosses the boundary between `mx` and `mx + 1`.

**Why This Works:**

- `last[i]` represents the furthest index that shares at least one prime factor with `nums[i]`
- By maintaining the maximum of these values as we iterate, `mx` represents the rightmost boundary of all prime factors seen so far
- When `mx < i`, we've found a gap where we can split without any prime factor appearing on both sides
- We return the first such `mx` found, giving us the smallest valid split index

**Time Complexity:** `O(n * sqrt(M))` where `n` is the array length and `M` is the maximum value in the array (for factorization).

**Space Complexity:** `O(n + P)` where `P` is the number of distinct prime factors across all numbers.

## Example Walkthrough

Let's walk through the solution with `nums = [4, 7, 15, 8, 3, 5]`.

**Step 1: Find prime factors for each number and track intervals**

We'll build our `first` dictionary and `last` array:

- Index 0: `nums[0] = 4 = 2²`
  - Prime factor 2: `first[2] = 0`, `last[0] = 0` (initially)

- Index 1: `nums[1] = 7`
  - Prime factor 7: `first[7] = 1`, `last[1] = 1` (initially)

- Index 2: `nums[2] = 15 = 3 × 5`
  - Prime factor 3: `first[3] = 2`, `last[2] = 2` (initially)
  - Prime factor 5: `first[5] = 2`, `last[2] = 2` (stays same)

- Index 3: `nums[3] = 8 = 2³`
  - Prime factor 2: Already in `first` at index 0, so update `last[0] = 3`

- Index 4: `nums[4] = 3`
  - Prime factor 3: Already in `first` at index 2, so update `last[2] = 4`

- Index 5: `nums[5] = 5`
  - Prime factor 5: Already in `first` at index 2, so update `last[2] = 5`

Final state:
- `first = {2: 0, 7: 1, 3: 2, 5: 2}`
- `last = [3, 1, 5, 0, 0, 0]`

The `last` array tells us:
- `last[0] = 3`: Prime factor 2 appears from index 0 to 3
- `last[1] = 1`: Prime factor 7 only appears at index 1
- `last[2] = 5`: Prime factors 3 and 5 appear from index 2 to 5

**Step 2: Find the earliest valid split**

Now we traverse with `mx` tracking the maximum right boundary:

- `i = 0`: `mx = last[0] = 3`
- `i = 1`: Check if `mx (3) < i (1)`? No. Update `mx = max(3, last[1]) = max(3, 1) = 3`
- `i = 2`: Check if `mx (3) < i (2)`? No. Update `mx = max(3, last[2]) = max(3, 5) = 5`
- `i = 3`: Check if `mx (5) < i (3)`? No. Update `mx = max(5, last[3]) = max(5, 0) = 5`
- `i = 4`: Check if `mx (5) < i (4)`? No. Update `mx = max(5, last[4]) = max(5, 0) = 5`
- `i = 5`: Check if `mx (5) < i (5)`? No. Update `mx = max(5, last[5]) = max(5, 0) = 5`

We never find a position where `mx < i`, so return `-1`.

This makes sense because:
- Prime 2 appears at indices 0 and 3 (spans 0-3)
- Primes 3 and 5 appear at indices 2, 4, and 5 (spans 2-5)
- These intervals overlap at indices 2 and 3, meaning we can't split anywhere without having at least one prime factor on both sides

Let's verify with a different example where a valid split exists: `nums = [2, 3, 5]`

- Index 0: `nums[0] = 2`, `first[2] = 0`, `last[0] = 0`
- Index 1: `nums[1] = 3`, `first[3] = 1`, `last[1] = 1`
- Index 2: `nums[2] = 5`, `first[5] = 2`, `last[2] = 2`

Final: `last = [0, 1, 2]`

Traversal:
- `i = 0`: `mx = last[0] = 0`
- `i = 1`: Check if `mx (0) < i (1)`? Yes! Return `mx = 0`

Split at index 0: left = `[2]` (product 2), right = `[3, 5]` (product 15). Since gcd(2, 15) = 1, this is valid.

## Python Solution

```python
class Solution:
    def findValidSplit(self, nums: List[int]) -> int:
        # Dictionary to store the first occurrence index of each prime factor
        first_occurrence = {}
        n = len(nums)
        # List to store the rightmost index that shares a prime factor with index i
        rightmost_shared = list(range(n))
      
        # Process each number to find its prime factors
        for index, number in enumerate(nums):
            current_num = number
            divisor = 2
          
            # Find all prime factors using trial division
            while divisor * divisor <= current_num:
                if current_num % divisor == 0:
                    # Found a prime factor
                    if divisor in first_occurrence:
                        # Update the rightmost index for the first occurrence of this prime
                        rightmost_shared[first_occurrence[divisor]] = index
                    else:
                        # Record the first occurrence of this prime factor
                        first_occurrence[divisor] = index
                  
                    # Remove all occurrences of this prime factor
                    while current_num % divisor == 0:
                        current_num //= divisor
              
                divisor += 1
          
            # Handle the case where current_num is a prime number > sqrt(original)
            if current_num > 1:
                if current_num in first_occurrence:
                    # Update the rightmost index for the first occurrence of this prime
                    rightmost_shared[first_occurrence[current_num]] = index
                else:
                    # Record the first occurrence of this prime factor
                    first_occurrence[current_num] = index
      
        # Find the valid split point by checking if we can split before reaching
        # the maximum rightmost index encountered so far
        max_rightmost = rightmost_shared[0]
      
        for current_index, current_rightmost in enumerate(rightmost_shared):
            # If max_rightmost is less than current index, we found a valid split
            if max_rightmost < current_index:
                return max_rightmost
            # Update the maximum rightmost index seen so far
            max_rightmost = max(max_rightmost, current_rightmost)
      
        # No valid split found
        return -1
```

## Time and Space Complexity

**Time Complexity:** `O(n * √m)` where `n` is the length of the array and `m` is the maximum value in the array.

- The outer loop iterates through all `n` elements in the array
- For each element `x`, we perform prime factorization:
  - The while loop runs from `j = 2` to `j ≤ √x`, which is `O(√x)` iterations
  - Inside this loop, we check if `j` divides `x` and perform dictionary operations (O(1) average case)
  - The inner while loop `while x % j == 0` divides out all factors of `j`, but this doesn't increase the overall complexity as the total number of divisions across all prime factors is `O(log x)`
- After the factorization loop, we handle the case where `x > 1` (when `x` is a prime factor greater than `√x`), which is `O(1)`
- The final loop to find the valid split point iterates through the array once, which is `O(n)`
- Overall: `O(n * √m + n) = O(n * √m)`

**Space Complexity:** `O(p)` where `p` is the number of distinct prime factors across all elements in the array.

- The `first` dictionary stores the first occurrence index of each prime factor, containing at most `O(p)` entries
- The `last` array stores `n` integers, requiring `O(n)` space
- Since `p ≤ n * log m` (each number has at most `O(log m)` prime factors), and in practice `p` is often much smaller than `n`, the space complexity is `O(n + p) = O(n)` in the worst case
- However, more precisely, the space complexity is `O(n)` for the `last` array plus `O(p)` for the dictionary, giving us `O(n + p)`

**Learn more about [how to find time and space complexity quickly](/problems/runtime_summary).**

## Common Pitfalls

### 1. **Inefficient Prime Factorization Leading to TLE**

One of the most common pitfalls is using an inefficient prime factorization method. Many developers initially try checking all numbers up to `n-1` as potential divisors, or use `while divisor <= current_num` instead of `while divisor * divisor <= current_num`.

**Problem Code:**
```python
# Inefficient - checks too many divisors
while divisor <= current_num:  # Wrong!
    if current_num % divisor == 0:
        # process prime factor
```

**Solution:**
Only check divisors up to the square root of the number. Any factor larger than the square root must be paired with a factor smaller than the square root.
```python
while divisor * divisor <= current_num:  # Correct!
    if current_num % divisor == 0:
        # process prime factor
```

### 2. **Computing Actual Products (Integer Overflow)**

Another pitfall is attempting to compute the actual products of array segments and then calculating their GCD. This fails because:
- Products can become astronomically large (exceeding even 64-bit integers)
- Computing GCD of very large numbers is computationally expensive

**Problem Code:**
```python
# Wrong approach - will overflow or be very slow
left_product = 1
right_product = 1
for i in range(split_index + 1):
    left_product *= nums[i]
for i in range(split_index + 1, n):
    right_product *= nums[i]
return gcd(left_product, right_product) == 1
```

**Solution:**
Work with prime factors instead of actual products. Track which indices share prime factors rather than computing products.

### 3. **Incorrect Handling of Prime Factors Greater Than sqrt(n)**

After the factorization loop, if the remaining number is greater than 1, it's a prime factor itself. Forgetting to handle this case will miss important prime factors.

**Problem Code:**
```python
while divisor * divisor <= current_num:
    # ... handle divisors ...
    divisor += 1
# Forgot to check if current_num > 1!
```

**Solution:**
Always check if the remaining number after factorization is greater than 1:
```python
while divisor * divisor <= current_num:
    # ... handle divisors ...
    divisor += 1

if current_num > 1:  # Don't forget this!
    # current_num is itself a prime factor
    if current_num in first_occurrence:
        rightmost_shared[first_occurrence[current_num]] = index
    else:
        first_occurrence[current_num] = index
```

### 4. **Off-by-One Errors in Split Index**

It's easy to confuse whether we're returning the split index itself or the index where we detected the split is valid. The problem asks for the split index (where the first part ends), not the detection point.

**Problem Code:**
```python
for current_index, current_rightmost in enumerate(rightmost_shared):
    if max_rightmost < current_index:
        return current_index  # Wrong! This returns the detection point
```

**Solution:**
Return `max_rightmost` which represents the actual split position:
```python
for current_index, current_rightmost in enumerate(rightmost_shared):
    if max_rightmost < current_index:
        return max_rightmost  # Correct! This is the split index
```

### 5. **Not Removing All Occurrences of a Prime Factor**

When a prime factor is found, all its occurrences must be removed from the number. Using a single division instead of a while loop will leave duplicate prime factors.

**Problem Code:**
```python
if current_num % divisor == 0:
    # Record the prime factor
    current_num //= divisor  # Wrong! Only removes one occurrence
```

**Solution:**
Use a while loop to remove all occurrences:
```python
if current_num % divisor == 0:
    # Record the prime factor
    while current_num % divisor == 0:  # Correct!
        current_num //= divisor
```

